# RF Explorer Pro Audio Edition plugin for SoundBase

Drives an **RF Explorer Pro Audio Edition** handheld spectrum analyzer
(15 MHz – 2.7 GHz), the older RF Venue unit that identifies as **model 11**
(`MODEL_AUDIOPRO`), over USB as a live-scan source in SoundBase.

This is the plugin that answers when the WSUB1G+ plugin logs:

> `/dev/tty.SLAB_USBtoUART is an RF Explorer model 11, which this plugin does not drive`

```sh
npm install
npm run doctor      # is everything wired up?
npm test            # the contract, against a fake unit — nothing attached
npm run smoke       # boots main.js as the host does; sweeps a real unit if one is plugged in
```

## Build and install for SoundBase

1. **Clone and install dependencies** (Node.js 20+):

   ```sh
   git clone https://github.com/gainmedialab/rf_explorer_pro_audio_plugin.git
   cd rf_explorer_pro_audio_plugin
   npm install
   ```

2. **Verify the plugin** without hardware:

   ```sh
   npm run doctor
   npm test
   RFE_FAKE=1 npm run smoke   # optional: handshake + fake sweep path
   ```

3. **Run in SoundBase Desktop** without packing a release — quit the app, then
   start it with `SB_PLUGIN_DIRS` pointing at the *parent* of this folder:

   ```sh
   # macOS — this checkout is ~/CODE/rf_explorer_pro_audio_plugin
   SB_PLUGIN_DIRS="$HOME/CODE" open -a "SoundBase Desktop"
   ```

   [docs/running-in-soundbase.md](docs/running-in-soundbase.md) has the rest:
   the plugin manager, where the device shows up, and the logs.

4. **Release install** (Lab / GitHub Releases): use the repo’s release workflow
   or `npm run pack:release` so SoundBase can install from a public release
   asset. See [docs/publishing.md](docs/publishing.md).

## What it supports

| | |
|---|---|
| Model | **Pro Audio Edition only (UART `<Main_Model>` 11).** Other RF Explorers are recognised, logged, and left alone. |
| Firmware | Same UART protocol as the SoundBase WSUB1G+ plugin; not refused by version. Confirm behaviour on your firmware with hardware (see below). |
| Platform | **macOS (Apple silicon and Intel) and Windows x64** as declared in the manifest. |

Frequency limits are read from the unit’s own configuration echo. Published
Pro Audio Edition range is **15–2700 MHz**. Input stage (direct / 30 dB
attenuator / 25 dB LNA) and DSP filter/fast follow the shared RF Explorer UART
API; DSP-fast-only-at-112-points and sweep-time estimates were measured on a
WSUB1G+ and are applied here until confirmed on a Pro Audio unit.

## Device notes (model 11 / Pro Audio Edition)

- Sold by **RF Venue** as the older handheld Pro Audio Edition (not the newer
  touchscreen “RF Explorer Pro”).
- USB bridge is a Silicon Labs CP210x. On macOS the port often appears as
  `/dev/tty.SLAB_USBtoUART` (or a `usbserial-*` path). Install
  [Silicon Labs' VCP driver](https://www.silabs.com/developers/usb-to-uart-bridge-vcp-drivers)
  if the OS does not already provide it. The plugin ships no drivers.
- Baud rate must be **500 kbps** (unit Config menu default).
- Plug in, power on, and pick **RF Explorer Pro Audio Edition** in SoundBase’s
  device picker. For a manual add, set **Serial port** to the device path
  (e.g. `/dev/tty.SLAB_USBtoUART` or `COM5`).
- While SoundBase holds the unit, the unit’s own calculator (max-hold,
  average) is switched off so every sweep is raw. On close, the unit is
  restored to 112 points and the calculator/input stage it was found with.
- SoundBase’s **RBW** field only offers *Auto*: the unit chooses RBW from
  point spacing; the resolved value is reported.

## What to expect from the instrument

| | |
|---|---|
| Range | 15 MHz – 2.7 GHz (from unit / published specs) |
| Points | 112 – 65 528 |
| RBW | Chosen by the unit from the point spacing; not settable |
| Amplitude | 0.5 dB steps (protocol resolution) |
| Sweep time | Estimated from WSUB1G+ measurements (~2–5 ms per point by RBW) |

**Point count controls both speed and resolution.** A span too wide for the
point count is given more points rather than gaps.

### Controls

- **Input stage** — Direct, Attenuator 30 dB, LNA 25 dB. Levels are referred
  to the antenna connector: the plugin compensates stage gain the same way the
  vendor libraries do.
- **DSP mode** — Filter (default) or Fast. Fast is only applied at 112 points;
  at any other count the control reads back as Filter.

## Layout

```
soundbase-plugin.json   identity and the Pro Audio product
main.js                 shell bootstrap — never edited
adapter.js              adapter contract: open, applyConfig, startSweep, …
discovery.js            which serial ports are RF Explorers (model 11)
driver/
  protocol.js           command encoding and the receive framer
  rfe-client.js         one unit on one port
  transport.js          the serial port
  fake-rfe.js           a fake Pro Audio unit (model 11)
  backend.js            real ports, or the fake (RFE_FAKE=1)
  protocol.md           the protocol as used
```

`RFE_FAKE=1 npm start` runs the plugin against the fake unit, for poking at
with `curl` on a machine with no RF Explorer.

## Not yet done

- **Hardware confirmation** of DSP-fast behaviour, input-stage offsets, and
  sweep timing on a real Pro Audio Edition (currently assumed from the shared
  UART protocol and WSUB1G+ measurements).
- **A minimum firmware check.**
- **Overload warnings** when the LNA is saturated.

The guide set for the plugin model itself is in [docs/](docs/README.md).
