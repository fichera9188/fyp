# Offline LoRa SOS communication and image transfer

Final-year project using Heltec WiFi LoRa 32 V3 boards. Phones connect to a node's open Wi-Fi network to send SOS reports, location information, compressed photographs and two-way text messages. Civilian and rescuer nodes exchange data over LoRa and acknowledge saved records.

## Project contents

| Item | Location |
| --- | --- |
| Arduino sketch, shared firmware and browser interface | [testing/](testing/) |
| Setup and upload instructions | [SETUP.md](SETUP.md) |
| Application behaviour and delivery semantics | [Firmware README](testing/README.md) |
| Paper draft: methodology, results, discussion and references | [Methodology-IEEE-HD-draft.md](testing/research/Methodology-IEEE-HD-draft.md) |
| Paper with portable figures, references, results and drafting notes | [Paper ZIP](testing/research/LoRa-Paper-Methodology-and-Figures.zip) |
| IEEE / IEEE-CVF bibliography | [Methodology-IEEE-references.bib](testing/research/Methodology-IEEE-references.bib) |
| Recorded timings and calculated statistics | [Results JSON](testing/research/Reported-image-results-2026-10-18.json) |
| Architecture and image-compression figures | [Figures](testing/research/architecture/figures/) |
| Supporting architecture explanation | [Architecture guide](testing/research/architecture/Architecture-and-Compression-Guide.md) · [PDF](testing/output/pdf/LoRa-Software-Architecture-and-Compression.pdf) |
| Existing software checks and development tools | [Tests](testing/tests/) · [Tools](testing/tools/) |

The paper is a working draft of the methodology and results sections. The architecture guide and PDF are supporting material; the manuscript contains the current experimental interpretation. Final layout and page-limit work remain, as described in the [drafting notes](testing/research/Methodology-drafting-notes.md).

## Four transfer modes

1. Baseline stop-and-wait.
2. Tuned stop-and-wait.
3. Fixed groups with bitmap acknowledgements.
4. Adaptive groups using confirmed delivery rate and acknowledgement recovery.

Select the role in `testing/Config.h` and the mode in `testing/TransferConfig.h`. All participating nodes must use the same mode, radio settings and private mesh key. The checked-in defaults are civilian and mode 1.

## Recorded image-reception results

We recorded five runs per mode using the same 2,284-byte JPEG with the boards side by side. Timing began when the rescuer indicated image reception and ended when the photo showed **complete and verified**. This interval excludes the initial SOS exchange and does not end at the civilian's final acknowledgement.

| Mode | Individual readings (s) | Mean (s) | Sample standard deviation (s) |
| --- | --- | ---: | ---: |
| 1 — baseline | 94, 90, 95, 98, 113 | 98.0 | 8.86 |
| 2 — tuned | 85, 89, 84, 83, 88 | 85.8 | 2.59 |
| 3 — fixed | 75, 80, 77, 82, 73 | 77.4 | 3.65 |
| 4 — adaptive | 58, 71, 75, 63, 61 | 65.6 | 7.13 |

Mode 4 reduced the mean measured interval by 33.1% against mode 1 and 15.2% against mode 3. These are descriptive results from five runs per mode. All four modes were also observed at approximately 50–100 m with a clear path, but numerical distance-specific timings were not recorded. Obstruction results are pending.

## Get started

Follow [SETUP.md](SETUP.md), then open `testing/testing.ino` in Arduino IDE. Keep the sketch in the `testing` folder so its folder name matches the `.ino` file.

Device credentials and private keys are generated locally and excluded from Git. The certificate and credential generators are included. Existing nodes require their existing private `NetworkSecrets.h` to join the same mesh.

## Project history

The application feature reference is [hiew798/src at cb88553](https://github.com/hiew798/src/tree/cb88553d10082b3374b14d7a4f48013c77505c6d). Integration details are recorded in the [firmware README](testing/README.md). Third-party Arduino libraries are installed separately and retain their own licences.
