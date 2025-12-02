# OpenWrt Wi-Fi QR

Small helper for generating Wi-Fi QR on OpenWrt.

The initial development and testing was done on **OpenWrt 24.10**
using the official SDK. Other recent
OpenWrt releases should work as well, but are not explicitly tested.

- CLI tool `wi-fi_qr` which generates SVG QR for all enabled Wi-Fi interfaces from UCI.
- LuCI integration `luci-app-wi-fi_qr` which shows small QR on status pages and a large QR in an overlay.

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
   ```

Packages are architecture-independent (`Architecture: all`), so one build
works on any target as long as dependencies are available for that device.

### Build from source (OpenWrt 24.10 SDK)

1. Download and unpack the OpenWrt 24.10 SDK for your target.
2. Clone this repository inside the SDK, for example:
3. Build packages:

   ```sh
   make package/wi-fi_qr/compile V=sc
   make package/luci-app-wi-fi_qr/compile V=sc
   ```

4. Resulting `.ipk` files will appear under `bin/packages/*/` inside the SDK.
   Copy them to the router and install via `opkg`.

---

## Command-line usage

After installing `wi-fi_qr`, the CLI tool is available as `wi-fi_qr` in `/usr/bin`.

### Generate QR codes

```sh
wi-fi_qr --gen
```

What this does:

- Loads `/etc/config/wireless` via UCI.
- Enumerates all `wifi-iface` sections (`@wifi-iface[0]`, `@wifi-iface[1]`, …).
- Skips:
  - disabled radios (`wifi-device`),
  - disabled interfaces (`wifi-iface` with `option disabled '1'`),
  - interfaces without SSID.
- For each active interface:
  - detects encryption (`nopass`, WEP, WPA/WPA2/WPA3-PSK, etc.);
  - fetches Wi-Fi key if needed;
  - builds a Wi-Fi QR string in the standard format;
  - runs `qrencode` to produce SVG;
  - writes files into `/www/wi-fi_qr/`:

    ```text
    /www/wi-fi_qr/id0_MyWiFi.svg
    /www/wi-fi_qr/id1_Guest_WiFi.svg
    ...
    ```

- Temporary files `*.svg.tmp` are cleaned up automatically.

You can then open a QR in any browser or QR app, for example:

```text
http://<router-ip>/wi-fi_qr/id0_MyWiFi.svg
```

(If LuCI is installed, it will generate the same URLs internally.)

### Remove generated QR codes

```sh
wi-fi_qr --del
```

This command:

- Removes only files created by the tool:
  - `/www/wi-fi_qr/id*_*.svg`
  - `/www/wi-fi_qr/id*_*.svg.tmp`
- Leaves any other files in `/www/wi-fi_qr` untouched.
- If the directory becomes empty, removal scripts try to remove it on package uninstall.

If you just want to regenerate QR codes, you can call `--gen` again:
old SVG files will be cleaned and replaced with fresh ones.

---

## LuCI usage

After installing `luci-app-wi-fi_qr` and reloading LuCI:

### Status → Overview → Wireless

- For each active Wi-Fi interface:
  - original signal strength icon is moved into a small vertical stack,
  - a mini QR icon is added next to it.
- Left click on the QR icon:
  - opens a large QR in a centered overlay on a darkened background.
- Middle click on the QR icon:
  - opens the underlying SVG (`/cgi-bin/wi-fi_qr?...`) in a new tab.

### Network → Wireless

- Inside the interface status column, a medium-size QR icon is added.
- Behaviour of mouse buttons is the same:
  - left click → overlay,
  - middle click → new tab.

The LuCI package automatically patches the Bootstrap theme `footer.ut` to
insert:

```html
<link rel="stylesheet" href="/luci-static/resources/wi-fi_qr.css">
<script src="/luci-static/resources/wi-fi_qr.js"></script>
```

On uninstall:

- this block is removed,
- the footer file is restored to its previous form.

---

## Uninstall

To remove both CLI helper and LuCI integration:

```sh
opkg remove luci-app-wi-fi_qr wi-fi_qr
```

Removal scripts:

- clean generated SVG files in `/www/wi-fi_qr`,
- try to remove empty directories `/www/wi-fi_qr` and `/usr/lib/wi-fi_qr`,
- undo the LuCI footer modification.

---

## License

This project is licensed under the **MIT License**.  
See the [`LICENSE`](LICENSE) file for details.
