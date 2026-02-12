# dataurl

A CLI tool to convert any file into a [Data URL (RFC 2397)](https://datatracker.ietf.org/doc/html/rfc2397) with accurate MIME type detection.

## Features

- **5-tier MIME detection** — extension mapping, magic number inspection, `file` command, content inspection, and smart fallback
- **60+ file types** supported out of the box — images, fonts, audio/video, documents, office formats, code files, and more
- Outputs to **stdout** or directly to a **file**
- Warns on large files (>5 MB) and empty files
- Works on **macOS** (Bash 3.2 compatible, BSD tools)

## Installation

### Quick install

```bash
# Copy to a directory in your PATH
cp dataurl ~/.local/bin/dataurl
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

# Use in CSS
echo "background: url($(dataurl bg.png));" >> style.css

# Round-trip test (encode then decode back)
dataurl image.png | cut -d, -f2 | base64 -D > decoded.png
diff image.png decoded.png
```

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

## Requirements

- **macOS** (tested) or Linux (should work with minor adjustments to `stat`)
- Bash 3.2+
- `openssl` (for base64 encoding)
- `xxd` (for magic number detection)
- `file` (for fallback MIME detection)

## License

MIT
