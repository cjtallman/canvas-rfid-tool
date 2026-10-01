# Canvas RFID Tool

A browser tool for writing NFC tags for the **ELEGOO Canvas** multi-material system (Centauri Carbon 2). Enter your spool's details, or look a filament up, and it generates the raw page-write commands for a blank NTAG213 sticker. The Canvas then recognizes third-party spools as if they were ELEGOO spools.

**Live site:** https://cjtallman.github.io/canvas-rfid-tool/

Everything runs in your browser. There's no sign-up, no tracking, and nothing is sent to a server. The only network request is the optional filament database download.

## Features

- **Manual entry form:** material, subtype, color, finish, nozzle temps, diameter, weight and production date.
- **Filament search:** 8,000+ filaments from [SpoolmanDB](https://github.com/Donkie/SpoolmanDB). The closest ELEGOO material and subtype are picked automatically, and the app warns you when the match is imperfect.
- **NFC Tools commands:** `A2:PP:XXXXXXXX` page writes, ready to paste.
- **Decode existing tags:** paste a memory dump or commands, or load a `.bin`, to fill the form from a tag.
- **Presets:** saved in your browser, with JSON export and import.
- **Page map:** shows exactly which bytes get written to which page.
- **Raw output:** `.bin` export and raw hex.

## Writing a tag

You need a phone with NFC, the free **NFC Tools** app by wakdev ([iOS](https://apps.apple.com/app/nfc-tools/id1252962749) / [Android](https://play.google.com/store/apps/details?id=com.wakdev.wdnfc)), and blank **NTAG213** stickers. NTAG215 and NTAG216 also work.

1. Fill in the form, or search for your filament and click a result.
2. Click **Copy commands**.
3. In NFC Tools, go to **Other → Advanced NFC commands**. Don't use the regular "Write" tab.
4. Paste into the **Data** field, run it, and hold the sticker to your phone until you see a checkmark.
5. Optional check: use **Other → Read memory**, then paste the dump into **Decode an existing tag**.
6. Stick the tag on the spool and load it into the Canvas.

> ⚠️ Never use NFC Tools' **Lock tag** or **Set password** options. Locking is permanent.

## Tag format

The layout follows the community reverse-engineered format from [DnG-Crafts/ELG-RFID](https://github.com/DnG-Crafts/ELG-RFID), which matches genuine ELEGOO spools. ELEGOO's [official guide](https://github.com/ELEGOO-3D/ELEGOO-RFID-Tag-Guide) doesn't match what the hardware actually reads. All values are big-endian.

| Page | Bytes | Contents |
|---|---|---|
| 4–15 | — | Optional NDEF URL (phones only; the printer ignores it) |
| 16 | `36 EE EE EE` | Header `0x36` + manufacturer code |
| 17 | `EE cc cc 00` | Manufacturer code (cont.) + filament code |
| 18 | `mm mm mm mm` | Material (e.g. PLA = `00807665`) |
| 19 | `ss ss 00 00` | Subtype (high byte = family, low byte = variant) |
| 20 | `RR GG BB ff` | Color RGB + finish (`M` matte, `S` silk, `T` translucent, `G` galaxy) |
| 21 | `tt tt TT TT` | Nozzle temp min / max (°C) |
| 22 | `00 00 00 00` | Unknown (possibly bed temps) |
| 23 | `dd dd ww ww` | Diameter (1/100 mm) / weight (g) |
| 24 | `YY MM 00 00` | Production date (BCD) |

Material and subtype codes come from [Savion/elegoo-rfid-editor](https://github.com/Savion/elegoo-rfid-editor).

## Known limitations

- **The brand always shows as ELEGOO.** The manufacturer code is fixed (`EEEEEEEE`), and the printer displays every tag as ELEGOO.
- **Only ELEGOO's material list is supported.** Materials without an ELEGOO equivalent (PEI, PVB, etc.) fall back to the closest base type.
- **Some subtypes don't show on the CC2 screen.** They're marked with `*` in the form, and the printer may show only the base material.
- **The Canvas may ignore the tag's temperatures.** In one test, the Canvas showed its own default (190 °C) even though the tag stored 210 °C. This isn't confirmed yet. The printing temperature comes from your slicer profile either way.
- **The filament search needs internet the first time** (a ~4.6 MB download). Everything else works offline.

## Running locally

It's a single `index.html` with no build step. Open it in a browser. To use it from a phone on your network, serve the folder:

```powershell
node -e "require('http').createServer((q,s)=>{s.setHeader('content-type','text/html');s.end(require('fs').readFileSync('index.html'))}).listen(8080)"
```

Then open `http://<your-pc-ip>:8080` on the phone.

## Credits

- [DnG-Crafts/ELG-RFID](https://github.com/DnG-Crafts/ELG-RFID): tag layout and real spool dumps
- [Savion/elegoo-rfid-editor](https://github.com/Savion/elegoo-rfid-editor): material and subtype codes
- [Donkie/SpoolmanDB](https://github.com/Donkie/SpoolmanDB): filament database (MIT)
- [AnySpool](https://anyspool.de): the original inspiration

## License

[MIT](LICENSE). Not affiliated with ELEGOO.
