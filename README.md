# OpenWrt-Wi-Fi_QR
Wi-Fi QR generator and LuCI integration for OpenWrt.

- CLI tool `wi-fi_qr` which generates SVG QR codes for all enabled Wi-Fi interfaces from UCI.
- LuCI integration `luci-app-wi-fi_qr` which shows small QR icons on status pages and a large QR in an overlay.

> Проект распространяется под лицензией MIT.

---

## Features

- Generates standard Wi-Fi QR codes in the form  
  `WIFI:T:WPA;S:<ssid>;P:<password>;H:false;`
- Supports:
  - open / `nopass` networks,
  - WEP,
  - WPA/WPA2/WPA3-PSK (anything that maps to `psk*` in UCI).
- Output format: **SVG** with a clean border and SSID caption.
- Per-interface files `idN_<safe-ssid>.svg` in `/www/wi-fi_qr/`.
- LuCI integration:
  - **Status → Overview → Wireless**: mini QR icon next to signal strength.
  - **Network → Wireless**: medium QR icon inside interface status cell.
  - Left click — big QR in an overlay, middle click — open QR in a new tab.

All code is shell + JavaScript + CSS, no compiled binaries. Package is marked as `PKGARCH:=all`.

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
