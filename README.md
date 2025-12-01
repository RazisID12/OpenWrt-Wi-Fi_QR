# OpenWrt Wi-Fi QR

Small helper for generating Wi-Fi QR codes on OpenWrt.

- CLI tool `wi-fi_qr` which generates SVG QR codes for all enabled Wi-Fi interfaces from UCI.
- LuCI integration `luci-app-wi-fi_qr` which shows small QR icons on status pages and a large QR in an overlay.

> This project is licensed under the MIT License.

---

## Features

**CLI (`wi-fi_qr`):**

- Reads `wireless` config from UCI.
- Skips disabled radios and interfaces.
- Detects encryption type and password (including `nopass`).
- Generates standard Wi-Fi QR strings, for example:  
  `WIFI:T:WPA;S:<ssid>;P:<password>;H:false;`
- Writes SVG files to `/www/wi-fi_qr`:
  - one file per Wi-Fi interface: `idN_<safe-ssid>.svg`;
  - SVG already contains a clean border and SSID caption.

**LuCI (`luci-app-wi-fi_qr`):**

- **Status → Overview → Wireless**
  - Mini QR icon next to signal strength badge for each active interface.
- **Network → Wireless**
  - Medium QR icon inside interface status cell.
- Mouse actions:
  - **Left click** – open large QR in a full-screen overlay.
  - **Middle click (wheel)** – open the same SVG in a new tab.
- Automatically injects `wi-fi_qr.js` and `wi-fi_qr.css` into the Bootstrap theme footer.
- On uninstall, restores the original footer.

All code is shell + JavaScript + CSS, no compiled binaries.  
Packages are marked as `PKGARCH:=all`.

---

## Packages

Repository contains two OpenWrt packages:

- `wi-fi_qr` – core shell module and CLI helper.
- `luci-app-wi-fi_qr` – LuCI CGI bridge and frontend (JS + CSS).

### Dependencies

- `wi-fi_qr`
  - `qrencode`
  - `uci`
- `luci-app-wi-fi_qr`
  - `luci-base`
  - `wi-fi_qr`

---

## Installation

### From prebuilt `.ipk`

1. Copy the `.ipk` files to your router (for example via `scp`).
2. Install them with `opkg`:

   ```sh
   opkg install wi-fi_qr_*.ipk luci-app-wi-fi_qr_*.ipk
