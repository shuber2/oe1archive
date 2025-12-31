# oe1archive

A tool to download Ö1 radio broadcasts. This repo is a clone of this [original repo](https://git.sthu.org/?p=oe1archive.git) and there is also a [website](https://www.sthu.org/code/codesnippets/oe1archive.html).

## Overview

Browse, search, and download OE1 (Austrian National Radio) broadcasts from the past 30 days with automatic HTML archiving.

## Features

- **30-day Interactive Browse** - Select any date and download directly
- **Fast Search** - Search by title and subtitle (≈10 seconds)
- **Deep Search** - Optional search including descriptions (≈5 minutes)
- **Smart Stream Extraction** - Automatic loopstream ID retrieval
- **Batch Processing** - Download multiple shows efficiently
- **Organized Storage** - Timestamped folders with HTML metadata (format: `prefix_yyyyMMdd_hhmm`)

## Requirements

Python 3.6+ with dependencies:
```
pip install requests simplejson python-dateutil
```

## Usage

### Interactive Mode
```bash
./oe1archive -c
```
Browse all 30 days and download shows directly.

### Search Shows by Title and Subtitle
```bash
./oe1archive -s "Show Name"
```
Fast search across title and subtitle fields. **Takes approximately 10 seconds.** Use this for most searches.

### Extended Search (including descriptions)
```bash
./oe1archive -s "Show Name" -e
```
Search all available broadcasts including full description text. **Takes approximately 5 minutes.** Use this if the regular search doesn't find what you're looking for.

### Batch Download
```bash
./oe1archive -d "Show Name" -p "prefix"
```
Download all matching broadcasts with metadata and audio. Use `-e` flag for extended search:
`./oe1archive -d "Show Name" -p "prefix" -e`

### Command Examples

```bash
# Show usage
./oe1archive -h

# Fast search by title/subtitle
./oe1archive -s "Show Name"

# Deep search including descriptions
./oe1archive -s "Some Keyword" -e

# Batch download all matches (fast search)
./oe1archive -d "Show Name" -p "prefix"

# Batch download all matches (deep search)
./oe1archive -d "Some Keyword" -p "prefix" -e
```

## How It Works

The OE1 API provides a rolling weekly window of current broadcasts. Streams remain accessible via loopstream for ~30 days, enabling full month archive access through direct download without search requirements.

## Technical Details

- **API**: http://audioapi.orf.at/oe1/json/2.0/
- **Stream Source**: https://loopstream01.apa.at/?channel=oe1&shoutcast=0&id=...
- **Archive Window**: ~30 days (loopstream availability)

## Output Structure

```
prefix_yyyyMMdd_hhmm/
├── prefix_yyyyMMdd_hhmm.html    (Metadata)
└── prefix_yyyyMMdd_hhmm.mp3     (Audio)
```

## Contributors

- Stefan Huber
- Gerhard Mitterlechner
