<div align="center">
    <h1>dataurl</h1>
    <h4>Convert any file into a Data URL — right from your terminal.</h4>

A fast, single-file CLI tool that encodes any file into a [Data URL (RFC 2397)](https://datatracker.ietf.org/doc/html/rfc2397) with accurate MIME type detection. No dependencies beyond what macOS already ships.

[![Version](https://img.shields.io/badge/version-1.0.0-blue?style=flat-square)](https://github.com/rckmath/dataurl/releases/latest)
![Platform](https://img.shields.io/badge/platform-macOS%20%7C%20Linux-blue?style=flat-square)
![Shell](https://img.shields.io/badge/shell-bash%203.2%2B-green?style=flat-square)
[![License](https://img.shields.io/github/license/rckmath/dataurl?style=flat-square)](LICENSE)

   <a href="https://www.buymeacoffee.com/rckmath" target="_blank">
       <img src="https://cdn.buymeacoffee.com/buttons/v2/default-yellow.png" alt="Buy Me A Coffee" style="height: 60px !important; width: 217px !important;">
   </a>
</div>

---

## Features

- **5-tier MIME detection** — extension mapping, magic number inspection, `file` command, content inspection, and smart fallback
- **60+ file types** supported out of the box — images, fonts, audio/video, documents, office formats, code files, and more
- **Zero external dependencies** — uses only `bash`, `openssl`, `xxd`, and `file` (all pre-installed on macOS)
- **Single file** — one self-contained script, easy to audit, copy, or vendor
- Outputs to **stdout** or directly to a **file**
- Warns on large files (>5 MB) and empty files
- Works on **macOS** (Bash 3.2 compatible, BSD tools) and **Linux**

---

## Installation

### Homebrew (recommended)

```bash
brew tap rckmath/tap
brew install dataurl
```

### Quick install

```bash
curl -fsSL https://raw.githubusercontent.com/rckmath/dataurl/main/dataurl -o ~/.local/bin/dataurl
chmod +x ~/.local/bin/dataurl
```

> Make sure `~/.local/bin` is in your `PATH`. Add this to your shell profile if needed:
> ```bash
> export PATH="$HOME/.local/bin:$PATH"
> ```

### From source

```bash
git clone https://github.com/rckmath/dataurl.git
cd dataurl
cp dataurl ~/.local/bin/dataurl
chmod +x ~/.local/bin/dataurl
```

---

## Usage

```
dataurl <file> [output_file]
```

| Argument | Description |
|----------|-------------|
| `file` | Input file path (required) |
| `output_file` | Output file path (optional; prints to stdout if omitted) |

| Option | Description |
|--------|-------------|
| `-h`, `--help` | Show help message |
| `-v`, `--version` | Show version |

### Examples

```bash
# Print Data URL to stdout
dataurl image.png

# Save Data URL to a file
dataurl photo.jpg output.txt

# Pipe into clipboard (macOS)
dataurl icon.svg | pbcopy

# Embed in CSS
echo "background: url($(dataurl bg.png));" >> style.css

# Embed in HTML
echo "<img src=\"$(dataurl photo.jpg)\">" >> index.html

# Round-trip test (encode then decode back)
dataurl image.png | cut -d, -f2 | base64 -D > decoded.png
diff image.png decoded.png  # should be identical
```

---

## How It Works

`dataurl` uses a **5-tier strategy** to detect the correct MIME type before encoding:

```
Input file
  │
  ├─ Tier 1: Extension map (case statement, 60+ types)
  │           Instant lookup — handles types where `file` is known to be wrong
  │
  ├─ Tier 2: Magic numbers (first 12 bytes via xxd)
  │           Detects images, PDFs, fonts, audio — even without an extension
  │
  ├─ Tier 3: `file --mime-type` (system fallback)
  │           Catches anything the above tiers miss
  │
  ├─ Tier 4: Content inspection (first 512 bytes)
  │           Checks for <svg>, <!doctype html>, <?xml> patterns
  │
  └─ Tier 5: Binary/text heuristic
              binary → application/octet-stream
              text   → text/plain
```

Once the MIME type is resolved, the file is base64-encoded with `openssl base64 -A` and assembled into a Data URL:

```
data:<mime>;base64,<encoded-data>
```

---

## Supported MIME Types

### Tier 1 — Extension map (instant, no I/O)

| Category | Extensions |
|----------|------------|
| Images | `png`, `jpg`/`jpeg`, `gif`, `bmp`, `tif`/`tiff`, `ico`, `svg`, `webp`, `avif`, `heic`, `heif` |
| Code | `js`/`mjs`/`cjs`, `ts`, `tsx`, `jsx`, `css`, `json`, `jsonld`, `wasm` |
| Fonts | `woff`, `woff2`, `ttf`, `otf` |
| Audio/Video | `mp4`/`m4v`, `webm`, `mp3`, `ogg`/`oga`, `opus`, `flac`, `wav` |
| Documents | `md`, `yaml`/`yml`, `html`/`htm`, `xml`, `pdf`, `csv`, `tsv`, `rtf` |
| Office (MS) | `doc`, `dot`, `xls`, `ppt`, `docx`, `xlsx`, `pptx` |
| Office (ODF) | `odt`, `ods`, `odp`, `odg`, `odf` |
| Office (WPS) | `wps`, `et`, `dps` |

### Tier 2 — Magic numbers (binary inspection)

Detects PNG, JPEG, GIF, BMP, TIFF, ICO, PDF, WOFF/WOFF2, GZIP, FLAC, OGG, MP3, WebP, HEIC, HEIF, AVIF, and MP4 from file headers — even without an extension.

### Tier 3–5 — Fallbacks

- `file --mime-type` system command
- Content inspection for SVG, HTML, and XML
- Binary/text heuristic fallback

Any file type not listed above will still work — it just falls through to the system `file` command or the final heuristic.

---

## Requirements

- **macOS** (tested) or **Linux** (may need `stat -c %s` instead of `stat -f%z`)
- Bash 3.2+
- `openssl` — base64 encoding
- `xxd` — magic number detection
- `file` — fallback MIME detection

All of these are pre-installed on macOS.

---

## License

This project is available under the [MIT License](LICENSE).
