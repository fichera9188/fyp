# Setup and upload

## 1. Open the sketch

Clone or download this repository. Open `testing/testing.ino` in Arduino IDE, keeping all firmware headers beside it.

The `testing/data/` files are the browser source. `WebAssets.h` already embeds them, so a separate filesystem asset upload is unnecessary.

## 2. Install the recorded dependencies

- Arduino ESP32 board package by Espressif Systems: **2.0.17**.
- Board: **Heltec WiFi LoRa 32(V3) / Wireless shell(V3) / Wireless stick lite (V3)**.
- FQBN: `esp32:esp32:heltec_wifi_lora_32_V3`.
- RadioLib: **7.8.1**.
- ArduinoJson: **6.21.5**.
- ESP32 HTTPS Server (`ESP32_HTTPS_Server`): **1.0.0**.
- Python 3 for helper scripts and PowerShell 7 for certificate generation and the SHA compatibility patch.

The WiFi Kit board selection is different from this LoRa board.

## 3. Prepare local credentials

Run these commands from the repository's `testing` directory for a new installation:

```powershell
python tools/generate_network_secrets.py
pwsh -File tools/generate_certificates.ps1
```

The first command creates `NetworkSecrets.h` with a random mesh authentication key and a separate rescuer dashboard password. Generate it once and privately use the same file for every node in the mesh. The generator preserves an existing file. When using already-configured boards, copy their existing `NetworkSecrets.h` into this local sketch instead of generating a different key.

The second command creates a matching `cert_der.h`, `key_der.h` and public `heltec-sos.cer` for `192.168.4.1`. It preserves existing files unless explicitly forced. An existing matching certificate/key pair can also be copied privately. Generated files are excluded from Git.

Wi-Fi itself has no password. The rescuer dashboard username is `rescuer`; its password is in the local `NetworkSecrets.h`.

## 4. Apply the HTTPS library fixes

Find the Arduino sketchbook location in Arduino IDE preferences. Its `libraries/ESP32_HTTPS_Server` directory needs both patches before compilation. Set the following path to the actual installed library directory:

```powershell
$httpsLibrary = 'C:\path\to\Arduino\libraries\ESP32_HTTPS_Server'
pwsh -File tools/apply_https_compatibility.ps1 -LibraryDirectory $httpsLibrary
python tools/apply_https_disconnect_fix.py --library $httpsLibrary
```

These scripts validate the supported library source and retain the original files. Explicitly passing the library location is required when this repository is outside the original Arduino sketchbook layout.

## 5. Select the role and transfer mode

In `Config.h`, use `ROLE_CIVILIAN` for the civilian board and `ROLE_RESCUER` for the rescuer board. In `TransferConfig.h`, select one of:

```cpp
#define TRANSFER_MODE TRANSFER_BASELINE
// Or TRANSFER_TUNED, TRANSFER_FIXED, TRANSFER_ADAPTIVE.
```

Use the same mode, mesh key and radio settings on both boards. Modes 1/2 use wire protocol 3; modes 3/4 use wire protocol 4. Set the role for each board before uploading and choose its actual COM port.

The checked-in radio configuration is 915 MHz, 125 kHz bandwidth, spreading factor 9, coding rate 4/7 and 22 dBm. Configure it for the actual hardware and deployment location. Keep the LoRa antenna attached while transmitting.

## 6. Upload and connect

Upload each role with Arduino IDE. Serial Monitor uses **115200 baud**. Check the role, mode and protocol in the startup log.

Connect a phone to `Heltec-LoRa-xxxxxx` for the civilian or `Heltec-Rescuer-xxxxxx` for the rescuer. Keep the connection when the phone reports no Internet. Open **https://192.168.4.1/**. The current firmware uses HTTPS; HTTP redirects there. Captive-page opening and local-certificate acceptance depend on the phone/browser.

The firmware README explains storage, delivery status, coordinates and chat. **Received by device** means the destination saved the item; it is separate from a human read receipt or rescue confirmation.

## Development and paper files

- After editing `data/`, run `python tools/embed_web_assets.py` to update `WebAssets.h`.
- Existing software tests and build tools are included, together with historical test notes. No new hardware test is part of the GitHub import.
- Optional native test scripts use Windows MSVC Build Tools; their configured installation path may need adjustment. Pass `--library` to the HTTPS regression runner when the library is elsewhere.
- `tools/build_transfer_matrix.ps1` accepts `-Cli` and `-CliConfig` for a local Arduino CLI installation. Compiled firmware embeds local credentials and should stay private.
- Read the manuscript in `research/Methodology-IEEE-HD-draft.md`. Its PNG figures and editable SVG originals are included.
- To rebuild the portable paper ZIP, run `python research/architecture/package_paper.py` from this sketch directory. It uses only the Python standard library.
- Figure generation requires ReportLab, Windows Arial fonts and Poppler's `pdftoppm` on PATH (or its path in the `PDFTOPPM` environment variable). Existing figures are included, so rebuilding is optional.
- The architecture guide builder rewrites its supporting guide from its own source text. Review those outputs before replacing manually revised documentation.
