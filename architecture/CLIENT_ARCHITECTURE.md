# WebRTC P2P Client Architecture Specification

This document details the client-side architecture, WebRTC P2P connection sequence flow, and media stream handshake protocols used by **VigorConnect**.

![Workspace Overview](../assets/workspace_overview.png)

---

## 1. System Topology Overview

```mermaid
flowchart LR
    CAM["IP Cameras<br/>(RTSP / ONVIF)"]
    GW["C++ Edge Gateway<br/>(libdatachannel)"]
    SIG["FastAPI Cloud Signaling<br/>(Python WSS / REST)"]
    TURN["Coturn Server<br/>(STUN / TURN Relay)"]
    CLIENT["Browser Client<br/>(CameraPlayer.ts)"]

    CAM -->|"1. Local RTSP Stream"| GW
    GW <--->|"2. Persistent WebSocket Control"| SIG
    CLIENT <--->|"3. WSS Signaling Handshake"| SIG
    
    GW ==>|"4. DIRECT WebRTC P2P STREAM (Video/Audio/PTZ)"| CLIENT
    
    GW -.->|"5. STUN Binding / TURN Relay Fallback"| TURN
    CLIENT -.->|"5. STUN Binding / TURN Relay Fallback"| TURN

    style GW fill:#0f172a,stroke:#38bdf8,stroke-width:2px,color:#fff
    style SIG fill:#1e1b4b,stroke:#a855f7,stroke-width:2px,color:#fff
    style CLIENT fill:#064e3b,stroke:#34d399,stroke-width:2px,color:#fff
    style TURN fill:#451a03,stroke:#fbbf24,stroke-width:2px,color:#fff
```

---

## 2. Architecture Block Diagram

```text
+-----------------------------------------------------------------------------------------+
|                                    EDGE / LAN NETWORK                                   |
|                                                                                         |
|  +------------------------+                        +---------------------------------+  |
|  |       IP Cameras       |                        |        C++ Edge Gateway         |  |
|  |  (RTSP H.264/265 Video | --- RTSP Streams ----> |   - RtspIngest / RtspP2PService |  |
|  |   & ONVIF SOAP/PTZ)    | <--- ONVIF PTZ ------- |   - libdatachannel Engine       |  |
|  +------------------------+                        +---------------------------------+  |
+--------------------------------------------------------------------|--------------------+
                                                                     |
                         = = = = = = = = = = = = = = = = = = = = = = | = = = = = = = = = =
                        ||   PRIMARY PATH: Direct WebRTC P2P Stream  |                   ||
                        ||   (Sub-second Latency ~150ms | Zero Cloud Media Overhead) ||
                         = = = = = = = = = = = = = = = = = = = = = = | = = = = = = = = = =
                                                                     |
                                                                     v
+------------------------------------+              +-------------------------------------+
|       CLOUD CONTROL PLANE          |              |            CLIENT PLANE             |
|                                    |              |                                     |
|  +------------------------------+  |  Signaling   |  +-------------------------------+  |
|  | FastAPI Signaling Server     | <== WSS/REST ==> |  |  Web Browser / React Admin    |  |
|  | (WebSocket Broker & Auth)    |  |              |  |  (CameraPlayer.ts SDK)        |  |
|  +------------------------------+  |              |  +-------------------------------+  |
|                 |                  |              +-------------------------------------+
|                 v                  |                                 ^
|  +------------------------------+  |                                 |
|  | Coturn STUN / TURN Server    |  | <.... STUN / TURN Relay .......+
|  | (NAT Traversal & Relay)      | ...................................+
|  +------------------------------+  |       (Fallback Secondary Path)
+------------------------------------+
```

---

## 3. WebRTC P2P Connection Sequence Flow

```mermaid
sequenceDiagram
    autonumber
    participant Client as Web Browser Client
    participant Cloud as FastAPI Cloud Signaling
    participant Gateway as C++ Edge Gateway
    participant Coturn as Coturn STUN/TURN

    Note over Gateway,Cloud: 1. Gateway establishes persistent WebSocket with Cloud
    Client->>Cloud: 2. Request Stream & Fetch Ephemeral Credentials
    Cloud-->>Client: 3. Return TURN Credentials (HMAC token)
    Client->>Coturn: 4. STUN Request - Get Public Address
    Client->>Cloud: 5. Send SDP Offer
    Cloud->>Gateway: 6. Forward SDP Offer
    Gateway->>Coturn: 7. STUN Request - Get Public Address
    Gateway->>Cloud: 8. Send SDP Answer
    Cloud->>Client: 9. Forward SDP Answer
    
    par Trickle ICE Exchange
        Client->>Cloud: ICE Candidate (Client)
        Cloud->>Gateway: Forward ICE Candidate (Client)
    and
        Gateway->>Cloud: ICE Candidate (Gateway)
        Cloud->>Client: Forward ICE Candidate (Gateway)
    end

    Note over Client,Gateway: 10. ICE Connectivity Check

    alt Direct P2P Success (Primary Path)
        Client->>Gateway: Establish Direct P2P Media Connection (SRTP Video < 150ms)
    else Symmetric NAT Firewall (Fallback Path)
        Client->>Coturn: Relay Data via TURN Server (UDP/TCP Relay)
        Gateway->>Coturn: Relay Data via TURN Server
        Coturn->>Client: Relayed SRTP Media Connection
    end
```

---

## 4. Key Protocol Specifications

| Feature / Protocol | Specification | Details |
| :--- | :--- | :--- |
| **Signaling Protocol** | WebSocket Secure (`wss://`) | JSON-RPC 2.0 SDP Offer / Answer exchange |
| **Video Codec** | H.264 / H.265 Passthrough | Zero-transcoding demuxing for ultra-low CPU overhead |
| **P2P Latency** | 100ms - 250ms | Direct SRTP peer-to-peer media transmission |
| **NAT Traversal** | STUN (UDP 3478) / TURN Relay | Full support for Symmetric NAT fallback |
| **Browser Compatibility** | Chrome, Edge, Firefox, Safari | 100% Native HTML5 WebRTC API (No plugins required) |

---

## Live Resources

- 📦 **Web SDK Reference**: [https://github.com/Vigor-RDLabs/VigorSDK](https://github.com/Vigor-RDLabs/VigorSDK)
- ⚙️ **Edge Gateway Releases**: [https://github.com/Vigor-RDLabs/vigor-gateway-releases](https://github.com/Vigor-RDLabs/vigor-gateway-releases)
- 📚 **Documentation Hub**: [https://github.com/Vigor-RDLabs/VigorDocuments](https://github.com/Vigor-RDLabs/VigorDocuments)
