---
name: whisper_cpp
description: Local speech-to-text using Whisper.cpp. Runs fully offline with downloaded models. High quality transcription with multiple model sizes.
metadata: {"clawdbot":{"emoji":"🎙️","requires":{"bins":["ffmpeg"]}}}
---

# Local Whisper (cpp)

Local speech-to-text using Whisper.cpp. Runs fully offline with downloaded models. High quality transcription with multiple model sizes.

# Usage

Transcribe audio files to text. Outputs transcription content directly to stdout.

```bash
whisper_cpp <audio_file> [--model=MODEL_NAME] [--language=LANG_CODE]
```

## Parameters

- `<audio_file>`: Full path to audio file (mp3, wav, m4a, flac, aac, ogg, wma)
- `--model`: Optional. Model name (e.g., base, small, medium, large, turbo). Default: base
- `--language`: Optional. Language code (e.g., en, fr, de, es). Default: auto-detect

## Examples

```bash
# Use default base model with auto language detection
whisper_cpp /path/to/audio.mp3

# Use specific model
whisper_cpp /path/to/audio.mp3 --model=medium

# Specify language
whisper_cpp /path/to/audio.mp3 --language=en

# Both model and language
whisper_cpp /path/to/audio.mp3 --model=large --language=fr
```

# Implementation

```bash
#!/bin/bash

set -e

# Configuration
MODELS_DIR="/home/ivo/Projects/others/whisper.cpp/models"
WHISPER_BIN="/home/ivo/Projects/others/whisper.cpp/build/bin/whisper-cli"
DEFAULT_MODEL="base"

# Parse arguments
audio_file=""
model="$DEFAULT_MODEL"
language="auto"

while [[ $# -gt 0 ]]; do
    case $1 in
        --model=*)
            model="${1#*=}"
            shift
            ;;
        --language=*)
            language="${1#*=}"
            shift
            ;;
        *)
            if [[ -z "$audio_file" ]]; then
                audio_file="$1"
            else
                echo "Error: Unknown parameter: $1" >&2
                exit 1
            fi
            shift
            ;;
    esac
done

# Validate audio file
if [[ -z "$audio_file" ]]; then
    echo "Error: No audio file specified" >&2
    echo "Usage: whisper_cpp <audio_file> [--model=MODEL] [--language=LANG]" >&2
    exit 1
fi

if [[ ! -f "$audio_file" ]]; then
    echo "Error: File not found: $audio_file" >&2
    exit 1
fi

if [[ ! "$audio_file" =~ \.(mp3|wav|m4a|flac|aac|ogg|wma)$ ]]; then
    echo "Error: Not a valid audio file: $audio_file" >&2
    exit 1
fi

# Validate model file
model_file="$MODELS_DIR/ggml-${model}.bin"
if [[ ! -f "$model_file" ]]; then
    echo "Error: Model file not found: $model_file" >&2
    echo "Available models:" >&2
    find "$MODELS_DIR" -name "ggml-*.bin" ! -name "*for-tests*" -printf "%f\n" | sed 's/^ggml-//' | sed 's/\.bin$//' >&2
    exit 1
fi

# Create temporary output directory
temp_dir=$(mktemp -d)
trap 'rm -rf "$temp_dir"' EXIT

output_base="${temp_dir}/transcription"

# Build whisper command flags
whisper_flags="-np -otxt"

# Add tdrz flag and srt output if model contains 'tdrz'
if [[ "$model" == *"tdrz"* ]]; then
    whisper_flags="$whisper_flags -osrt -tdrz"
fi

# Add language flag
if [[ "$model" == *".en"* ]]; then
    # English-only model
    whisper_flags="$whisper_flags -l en"
elif [[ "$language" != "auto" ]]; then
    # Specific language requested
    whisper_flags="$whisper_flags -l $language"
fi
# If language is "auto", omit the -l flag to let whisper auto-detect

# Run whisper.cpp
"$WHISPER_BIN" -m "$model_file" -f "$audio_file" -of "$output_base" $whisper_flags 2>/dev/null

# Output the transcription content
txt_file="${output_base}.txt"
if [[ -f "$txt_file" ]]; then
    # For tdrz models, combine txt and srt
    if [[ "$model" == *"tdrz"* ]] && [[ -f "${output_base}.srt" ]]; then
        cat "$txt_file"
        echo ""
        echo "========================================"
        echo "TIMESTAMPED TRANSCRIPTION WITH SPEAKERS:"
        echo "========================================"
        echo ""
        cat "${output_base}.srt"
    else
        cat "$txt_file"
    fi
else
    echo "Error: Transcription failed. Output file not created." >&2
    exit 1
fi
```
