# ttm57sl-tools

Tools and notes for using a **Rane TTM 57SL** mixer outside Serato Scratch Live on Windows: as a 2×stereo USB
soundcard for DVS (vinyl control) software such as Mixxx, with the turntable timecode sent to the computer while
the channel switches are on the digital (D1/D2) positions.

The key finding: after power-on the mixer is a class-compliant USB Audio device, and **one HID command**
(`TtmCmd.exe 66`) tells it to send the phono inputs to USB Line 1/2. No Serato driver and no 32-bit ASIO driver are needed.
See [docs/PROTOCOL.md](docs/PROTOCOL.md) for the protocol notes and measurements.

> No Serato or Rane files are included. `AsioHost` loads `TTM57SLASIO.dll` from your own Scratch Live installation
> (`C:\Program Files (x86)\Serato\Drivers\ASIO\32\57SL\`).

## Status

| | |
|---|---|
| Audio out to D1/D2 (Mixxx, WDM-KS) | works |
| Timecode on Line 1 with switch on D1 | works (measured, and vinyl control works in Mixxx) |
| Timecode on Line 2 with switch on D2 | expected to work, not verified |
| SL buttons/knobs | not supported (only sent in Serato mode, protocol not decoded) |
| Scratch feel vs Scratch Live | Mixxx feels slower; lower the Mixxx audio buffer (see below) |

## Build

Needs only Windows (the .NET Framework 4.x C# compiler is built in) and, for the scripts, Python 3.

```powershell
powershell -ExecutionPolicy Bypass -File .\build.ps1
```

The tools are built into `bin\`.

## Tools

| Tool | What it does |
|---|---|
| `TtmCmd.exe <hex>` | Sends the `0x18` settings packet over the class-compliant HID interface. `66` = phono to Line 1 and Line 2. Waits up to 10 s for the mixer to appear. |
| `HidSniff.exe <seconds> [log]` | Logs every changed input report from the mixer's HID interface. |
| `AsioHost.exe <TTM57SLASIO.dll> <log> [panel]` | 32-bit research host for Rane's ASIO driver: logs its USB commands, meters the 4 inputs, opens its control panel, and sweeps the record-source byte, saving a WAV per value. **This switches the mixer to Serato mode until it is power-cycled.** |
| `MixerDump.exe` | Dumps the Windows mixer topology (volume, mute, selectors) of all audio devices. |
| `MixerSet.exe "<name part>"` | Unmutes and sets to 100 % every control of matching devices. Windows had the TTM57SL inputs muted at about 54 %. |
| `scripts/pa_list.py` | Lists audio devices exactly as Mixxx sees them (uses Mixxx's `portaudio.dll`). |
| `scripts/pa_capture.py <dir> <tag> [s]` | Records TTM57SL Line 1+2 into a 4-channel WAV (`PA_API=WDM-KS` or `MME`). |
| `scripts/analyze_sweep.py <dir>` | Per WAV: level, peak frequency, share of energy at 1 kHz and L/R phase, which flags timecode. |

Run the Python scripts isolated, for example `py -I scripts\pa_list.py`.

## Mixxx setup (Windows)

1. Power the mixer on. It starts in class-compliant mode.
2. Start Mixxx with `bin\Mixxx-TTM57SL.cmd`. It sends `TtmCmd 66` and then starts Mixxx. The setting is lost on power-off, so always start this way.
3. Sound Hardware: API **Windows WDM-KS**, Deck 1 → TTM57SL Line 1 (channels 1-2), Deck 2 → Line 2,
   Vinyl Control 1/2 ← Line 1/2 inputs. Example: [mixxx/soundconfig.example.xml](mixxx/soundconfig.example.xml).
4. Set the audio buffer to 5.8 ms or 2.9 ms. The 23 ms default adds a lot of lag. Raise the vinyl control gain:
   the phono path arrives at about −45 dBFS over WDM-KS.
5. On the mixer: turntables into the channel phono inputs, channel switches on **D1/D2**.

## License

[MIT](LICENSE). This covers only the code and documentation in this repository. Serato Scratch Live, its drivers and
`TTM57SLASIO.dll` are not part of this project and remain under their owners' terms.

## Magyar összefoglaló

Eszközök és jegyzetek a Rane TTM 57SL keverő Scratch Live nélküli használatához (például Mixxx DVS).
A keverő bekapcsolás után szabványos USB-hangkártya. A `TtmCmd.exe 66` parancs után a phono bemenetek jele
D1/D2 kapcsolóállásban is megy a gépre, így a timecode-vezérlés működik. Részletek: [docs/PROTOCOL.md](docs/PROTOCOL.md).
