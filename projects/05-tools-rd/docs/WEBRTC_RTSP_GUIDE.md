# WebRTC RTSP 串流整合指南 (go2rtc)

本文件記錄如何透過 `go2rtc` 作為中介伺服器，將 RTSP 攝影機訊號轉為 WebRTC 低延遲串流並整合至前端網頁。

---

## 1. 架構原理

WebRTC 並非由瀏覽器直接讀取 RTSP，而是透過中介服務進行協定轉換與信令交換。

**流程：**
`RTSP Camera` --(RTSP)--> `go2rtc` --(WebRTC)--> `Browser`

---

## 2. 部署方式

### 2.1 推薦做法：Docker Compose 整合部署
將 `go2rtc` 作為獨立 Service 與後端並列，最易於維護與擴充。

```yaml
services:
  app-backend:
    image: your-backend-api:latest
    networks: [cam-net]

  go2rtc:
    image: alexxit/go2rtc:latest
    ports:
      - "1984:1984"      # API
      - "8555:8555/tcp"  # WebRTC Signaling
      - "8555:8555/udp"  # WebRTC Media
    volumes:
      - ./go2rtc.yaml:/config/go2rtc.yaml
    networks: [cam-net]
```

### 2.2 進階做法：單一容器混合部署 (Single Container)
若環境限制只能執行一個容器，可透過 Multi-stage Build 將 `go2rtc` 包進後端鏡像。

**Dockerfile 參考：**
```dockerfile
# 取得 go2rtc 執行檔
FROM alexxit/go2rtc:latest AS go2rtc-img
FROM your-backend-base-image

# 安裝必要組件
RUN apt-get update && apt-get install -y ffmpeg

# 複製執行檔
COPY --from=go2rtc-img /usr/local/bin/go2rtc /usr/local/bin/go2rtc

# 使用啟動腳本同時執行兩個進程
# entrypoint.sh 內容: /usr/local/bin/go2rtc & ./your-backend-app
COPY entrypoint.sh /entrypoint.sh
ENTRYPOINT ["/bin/sh", "/entrypoint.sh"]
```

---

## 3. go2rtc.yaml 配置範例
支援直接接入、轉碼接入以及 ONVIF 自動探索。

```yaml
streams:
  # 1. 直接接入 (H.264 攝影機首選)
  cam1: rtsp://admin:password@192.168.1.10:554/stream1

  # 2. FFmpeg 轉碼接入 (解決音訊不相容，如 G.711 轉 Opus)
  cam_opus:
    - rtsp://admin:password@192.168.1.11/ch0
    - ffmpeg:cam_opus#audio=opus 

  # 3. ONVIF 自動探索
  cam_tapo: onvif://admin:password@192.168.1.12:2020

webrtc:
  listen: ":8555"
  candidates:
    - stun:8555
  ice_servers:
    - urls: [stun:stun.l.google.com:19302]
```

---

## 4. 前端實作方式 (以 Angular 為例)

### 4.1 WebRTC Service (WebSocket 信令)
```typescript
async startStream(streamName: string, serverUrl: string, videoElement: HTMLVideoElement) {
  this.pc = new RTCPeerConnection({ iceServers: [{ urls: 'stun:stun.l.google.com:19302' }] });
  this.ws = new WebSocket(`ws://${serverUrl}/api/ws?src=${streamName}`);

  this.ws.onopen = async () => {
    this.pc!.onicecandidate = (e) => {
      if (e.candidate) this.ws?.send(JSON.stringify({ type: 'webrtc/candidate', value: e.candidate.candidate }));
    };
    const offer = await this.pc!.createOffer();
    await this.pc!.setLocalDescription(offer);
    this.ws?.send(JSON.stringify({ type: 'webrtc/offer', value: offer.sdp }));
  };

  this.ws.onmessage = async (e) => {
    const msg = JSON.parse(e.data);
    if (msg.type === 'webrtc/answer') await this.pc?.setRemoteDescription({ type: 'answer', sdp: msg.value });
    if (msg.type === 'webrtc/candidate') await this.pc?.addIceCandidate({ candidate: msg.value, sdpMid: '0' });
  };
}
```

---

## 5. 常見品牌 RTSP 路徑參考

| 品牌 | 常見 RTSP 路徑範例 |
| :--- | :--- |
| **Dahua** | `rtsp://user:pass@IP/cam/realmonitor?channel=1&subtype=0` |
| **Hikvision** | `rtsp://user:pass@IP:554/Streaming/Channels/101` |
| **TP-Link** | `rtsp://user:pass@IP:554/stream1` |

---

## 6. 維運與除錯
- **埠口**：必須確保 **UDP 8555** 與 **TCP 1984** 已開通。
- **Codec**：若無畫面請檢查是否為 H.265，若是請轉碼為 H.264。
- **診斷**: `chrome://webrtc-internals` 是最強大的除錯工具。
