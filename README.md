# Epomaker TH108 Pro — Local HUB

An unofficial, dependency-free web-based control panel for the **Epomaker TH108 Pro**
mechanical keyboard (and close relatives that use the same Huafenda v1 protocol).

Runs entirely in the browser via the **WebHID API** — no install, no background
service, no vendor cloud. Open the file, click **Connect**, pick your keyboard,
done.

Works over **both wired USB and the 2.4G wireless dongle**.

> ⚠️ **Unofficial project.** Not affiliated with, endorsed by, or supported by
> Epomaker, Huafenda, or any of their partners. Use at your own risk.

---

## Features

- 🎨 **Per-effect backlight control**
  - Effect selection (19 built-in effects: static, breathing, waveflow, ripple, …)
  - RGB color picker with a "Vibrant" mode (rainbow overlay on top of the effect)
  - Brightness and speed sliders
  - Direction toggle for effects that support it
- 🔌 **Wired and 2.4G**
  - Auto-detects the correct HID interface from the device descriptor
    (`0xFF68/0x61` for wired, `0xFF60/0x61` for the dongle)
  - Reads the output report size from the HID descriptor instead of hardcoding it
    (64 bytes wired, 32 bytes over the dongle)
- 📡 **Link state awareness**
  - Listens for vendor push frames (`A6 FF 01` wake-up, `55 FC 05` disconnect,
    `55 FC 06` sleep) and reflects the keyboard link state in the UI
- 🧪 **Built-in diagnostics**
  - Enumerates all Huafenda HID interfaces of the current device
  - Prints report IDs and sizes
  - Probes the keyboard with a harmless `COMM_START` command on both interfaces
  - Useful for verifying a new device variant or a new firmware revision
- 🚫 **No dependencies**
  - Single HTML file, no build step, no CDN calls, no telemetry

## Requirements

- A desktop browser with WebHID support:
  - **Chrome / Edge / Opera ≥ 89**
  - Firefox and Safari do **not** support WebHID (and probably never will)
- The keyboard connected via **wired USB** or via the **2.4G dongle**
- The 2.4G keyboard must be **awake and paired** to the dongle to receive commands

## Usage

1. Download or clone this repository.
2. Open `index.html` in a supported browser.
3. Click **Connect** and pick your keyboard (or its 2.4G receiver) in the browser
   picker.
4. Adjust effect, color, brightness, speed.
5. Click **Apply** — or enable **Auto-apply** for live adjustments.

> **Note:** WebHID permissions are granted per physical device. If you connect a
> second unit of the same model, you will need to click **Connect** once and pick
> it in the browser picker. After that it will be auto-detected on future page
> loads.

## How it works

The Epomaker TH108 Pro talks to the host over a proprietary Huafenda v1 protocol
tunnelled through a vendor-specific HID collection:

- **Output report** carries commands, always as a 32 or 64 byte packet
  starting with `0xAA`.
  The output report size is read from the HID descriptor at connect time, because
  the wired interface exposes 64-byte reports while the 2.4G dongle exposes
  32-byte reports.
- **Input report** carries responses and unsolicited status frames.
  Responses start with `0x55`, push frames may start with `0x55` or `0xA6`.
- **Command** layout for the LED effect (cmd `0x23`):

  | Offset | Meaning                                   |
  |-------:|-------------------------------------------|
  | 0      | `0xAA` — magic byte                       |
  | 1      | `0x23` — command ID (SET_LED_EFFECT)      |
  | 2      | payload length (`0x10`)                   |
  | 3–7    | header bytes                              |
  | 8      | effect ID                                 |
  | 9–11   | R, G, B                                   |
  | 12     | reserved (`0xFF`)                         |
  | 13–15  | secondary R, G, B (unused by the UI)      |
  | 16     | vibrant mode (0/1)                        |
  | 17     | brightness (0–5)                          |
  | 18     | speed (1–5)                               |
  | 19     | direction (0 = forward, 1 = reverse)      |
  | 20–21  | reserved                                  |
  | 22–23  | `0xAA 0x55` — end marker                  |

  The remaining bytes up to the report size are zero-padded.

## What is not (and probably never will be) controllable from the browser

- **Side lighting and the display** — only controllable through keyboard
  hotkeys (Fn layer). The vendor's own hub doesn't expose them either.
- **Key remapping, macros, and profile switching** — not implemented here yet.
  The protocol supports them; it's on the roadmap.
- **Battery level and firmware version** — technically readable via
  `GET_DEVICE_INFO 0x10`, but not shown in the UI.

## Known limitations

- Firefox and Safari don't support WebHID and can't run this.
- macOS users may need to close the vendor app first — only one process can hold
  the HID device.
- If the keyboard is asleep when you hit **Apply**, the command is silently
  dropped (the UI shows a warning). Wake it first by pressing any key.

## Diagnostics

Click **Diagnostics** in the UI to:

- Enumerate all HID interfaces of your keyboard with usage pages, report IDs,
  and report sizes.
- Send a harmless `COMM_START` probe to both the wired and dongle interfaces.
- Print the raw response bytes to the on-page log.

This is the fastest way to figure out why a new keyboard variant isn't responding:
either the interface is missing, the report size is different, or the firmware
answers with a different format.

## Contributing

Issues and pull requests are welcome, especially:

- Dumps of HID descriptors from other Epomaker models using the same protocol
- Reports of which effects work on which firmware revisions
- Support for key remapping / macros via the same protocol

Please **do not** include raw dumps containing MAC addresses, serial numbers, or
other identifiers that could be tied to a specific unit. Redact them first.

## Legal

This project is an independent, unofficial implementation. It contains **no code
copied from the vendor's official hub**. The protocol was reverse engineered
from observing the device's HID traffic for interoperability purposes, which is
expressly permitted in many jurisdictions.

Epomaker, TH108, and any related names are trademarks of their respective
owners. This project is not affiliated with them.

## License

[MIT](LICENSE) © [Your Name or Handle]
