# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

A Flask web application that converts uploaded images into Portuguese-style blue tile (azulejo) murals. The image processing pipeline applies a pencil sketch effect and blue-tan colorization (#4c9ccc / #e9ddb3), then overlays a tile grid.

Based on: https://github.com/WillBolduc05/imageToPencilSketch

## Build and Run

**Local development:**
```bash
pip3 install opencv-python pillow argparse flask
python3 sketchConverter.py                    # Web server on http://localhost:8000
```

**CLI mode:**
```bash
python3 sketchConverter.py --input=image.jpg --output=tile.png --thickness=5
```

**Docker:**
```bash
docker build -t image2tile .
docker run -p 80:8000 image2tile
```

There are no tests, linting, or formatting tools configured.

## Architecture

The entire application is in **sketchConverter.py** (single file, ~109 lines).

**Image processing pipeline** (in `convertImage()`):
1. Convert to sketch via Gaussian blur inversion technique (`convertToSketch`)
2. Apply blue-tan colorization (`blueScaleFromGrey` / `blueScaleFromRGB`)
3. Blend sketch with colorized original at 75/25 ratio (`imgOverlay`)
4. Draw tile grid lines (`drawTiles` — 4x4 grid, 2px grey lines)
5. Encode as base64 JPEG (500x500 preview + full-size download)

**Flask routes:**
- `GET /` — Upload form (`templates/index.html`)
- `POST /` — Process image and return result (`templates/image.html`)

## CI/CD

Two GitHub Actions workflows build and push Docker images on every push:
- **docker-image-dockerhub.yaml** → `britorafa/image2tile` (Docker Hub)
- **docker-image-quay.yaml** → `quay.io/brito_rafa0/sketcher` (Quay.io)

Both use SHA-based tags and require repository secrets: `DOCKERHUB_USERNAME`, `DOCKERHUB_TOKEN`, `QUAY_USERNAME`, `QUAY_ROBOT_TOKEN`.

## Key Dependencies

- **opencv-python** — Core image processing (sketch conversion, grid drawing)
- **Pillow** — Image manipulation and colorization
- **Flask** — Web server (port 8000)
- **Dockerfile** installs system deps `ffmpeg`, `libsm6`, `libxext6` for OpenCV

## Known Issues

The blue colorization palette is still being tuned — colors may look off for some images (noted in README).
