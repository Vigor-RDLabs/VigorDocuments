# Dashboard UI & Workflow Specification

This specification outlines the user interface components and operational workflows in the VigorLabs Camera Operations Dashboard.

---

## 1. Workspace Onboarding & Overview

![Workspace Overview](../assets/workspace_overview.png)

The **Workspace Overview** dashboard provides immediate visibility into:
- **Onboarding Progress Bar**: 5-step checklist tracking workspace creation, site setup, gateway pairing, camera registration, and first stream test.
- **KPI Metrics**: Active camera limits, Gateways Online, Stream Sessions count, and TURN Relay Bandwidth consumption.
- **Live IP Cameras Grid**: Interactive multi-camera video player grid displaying real-time stream status and fps/bitrate metrics.

---

## 2. Camera Management List

![Cameras Management](../assets/cameras_management.png)

The **Cameras Management** view enables administrators to:
- Filter cameras by Site location, paired Edge Gateway, Online/Offline status, and License state.
- Trigger real-time stream testing via WebRTC P2P player modals.
- Assign cameras to logical projects or facility locations.

---

## 3. Site Operations & Facility Mapping

![Site Operations](../assets/sites_management.png)

The **Sites Operations** view provides:
- Interactive map view of facility locations across geographical regions.
- High-level gateway online metrics per site location.
- Site provision form for onboarding new regional facilities.

---

## 4. Platform Sign-In & Security

![Login Page](../assets/login_page.png)

- Enterprise OAuth single sign-on integration via Google and GitHub.
- Direct link to VigorLabs Terms of Service and Privacy Policy.

---

## Related Repositories

- 📦 **Web SDK**: [https://github.com/Vigor-RDLabs/VigorSDK](https://github.com/Vigor-RDLabs/VigorSDK)
- ⚙️ **Edge Gateway Releases**: [https://github.com/Vigor-RDLabs/vigor-gateway-releases](https://github.com/Vigor-RDLabs/vigor-gateway-releases)
