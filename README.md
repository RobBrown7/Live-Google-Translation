#READ ME

# Live-Google-Translation
This is a premade script by Google in Colab to automatically translate live audio.

# Gemini Live Audio Translation Quickstart

Stream audio from a public URL to the Gemini Live API, display source and translated transcripts, and save the translated speech as a WAV file. This README documents the Google Gemini Cookbook notebook linked below.

[![Open in Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/google-gemini/cookbook/blob/main/quickstarts/Get_started_LiveTranslate.ipynb)

## What the notebook does

- Downloads and decodes audio with FFmpeg.
- Streams 16 kHz, mono, 16-bit PCM audio at approximately real-time speed, using nominal 100 ms chunks.
- Receives translated audio and prints source and translation transcripts as they arrive.
- Writes translated audio to `audio_translation.wav` as 24 kHz, mono, 16-bit audio.
- Plays the completed translation in the notebook.

The notebook identifies the Live API as a preview feature. Model availability and SDK interfaces may change; consult the [Gemini Live API documentation](https://ai.google.dev/gemini-api/docs/live) when adapting the example.

## Requirements

- A Gemini API key from [Google AI Studio](https://aistudio.google.com/apikey).
- Google Colab, or a Jupyter environment with Python 3.11 or later. The example uses `asyncio.TaskGroup`.
- The `google-genai` Python SDK.
- FFmpeg available on the system path. It is preinstalled in the Colab environment described by the notebook.
- A publicly accessible audio URL.

## Run in Google Colab

1. Open the notebook using the **Open in Colab** button above.
2. In Colab's **Secrets** panel, add a secret named `GEMINI_API_KEY` and enable notebook access.
3. Run the setup cells to install the SDK, load the secret, and initialize the client.
4. Select a translation-capable model in the model cell. The viewed notebook defaults to `gemini-3.8-flash`; choose a model available to your API key that supports the translation configuration.
5. Run the imports and helper cells in order.
6. Set `audio_url` and `target_lang` in the final cell, then run it.

The example uses this public sample and Spanish as the target language:

```python
audio_url = "https://storage.googleapis.com/generativeai-downloads/gemini-cookbook/audio/gemini-live-translate-sample.wav"
target_lang = "es"

await run_audio_translation(
    url=audio_url,
    target_lang=target_lang,
)
```

After the session completes, the notebook displays an audio player for `audio_translation.wav`. Transcripts are printed in the cell output with `[Source]` and `[Translation]` labels, including language codes when provided by the API.

## Adapt for a local notebook

Install FFmpeg using your operating system's package manager, then install the Python dependencies in a dedicated environment:

```bash
python -m pip install --upgrade google-genai ipython jupyterlab
```

Set the `GEMINI_API_KEY` environment variable before starting Jupyter. Replace the Colab-specific secret-loading cell with:

```python
import os
from google import genai
from google.genai import types

client = genai.Client(api_key=os.environ["GEMINI_API_KEY"])
```

Run the remaining notebook cells in order. Keep API keys out of committed notebooks, source files, and repository history.

The example uses top-level `await`, supported in notebook cells. A standalone Python script needs an asynchronous entry point, such as `asyncio.run(...)`, and a replacement for notebook audio playback.

## How translation works

The notebook runs three asynchronous tasks together:

1. `stream_audio_url()` decodes the URL with FFmpeg, places PCM chunks in a bounded queue, and paces them to simulate live input.
2. `send_realtime()` sends queued chunks to the Live session with the MIME type `audio/pcm;rate=16000`.
3. A receive loop writes returned audio to the WAV file and prints both transcript streams.

The Live connection requests audio responses and enables translation and transcription:

```python
config = types.LiveConnectConfig(
    response_modalities=[types.Modality.AUDIO],
    translation_config=types.TranslationConfig(
        echo_target_language=True,
        target_language_code=target_lang,
    ),
    input_audio_transcription=types.AudioTranscriptionConfig(),
    output_audio_transcription=types.AudioTranscriptionConfig(),
)
```

When input finishes, the runner waits for queued audio to be sent, allows four seconds for final responses, and cancels the remaining tasks.

## Configuration

| Setting | Example value | Purpose |
| --- | --- | --- |
| `MODEL` | `gemini-3.8-flash` in the viewed notebook | Selects the Live model; verify translation support and availability. |
| `audio_url` | Cookbook sample URL | Supplies the source audio. |
| `target_lang` | `es` | Sets the target language code. |
| `file_name` | `audio_translation.wav` | Names the translated audio file; change it inside the runner. |
| Final response wait | `4.0` seconds | Allows remaining translation responses to arrive after upload ends. |

## Limitations and troubleshooting

- **Local microphone and camera:** Standard Colab runs on a remote VM. This example streams a URL; consult the [Cookbook Live API examples](https://github.com/google-gemini/cookbook/tree/main/examples) for local hardware streaming.
- **Missing API key:** Check the Colab secret name and notebook access, or confirm that the environment variable exists in your local Jupyter process.
- **Dependency conflicts:** The viewed notebook shows conflicts after upgrading `google-genai` in Colab. If imports or execution fail, use a clean environment with compatible dependencies and restart the runtime after changing packages.
- **Model or configuration errors:** Confirm that your selected model supports the Live translation settings and is available to your API key.
- **No audio input:** Verify that the URL is accessible to the runtime and FFmpeg can decode it. The example suppresses FFmpeg error output; expose its stderr when diagnosing decoding failures.
- **Incomplete final translation:** The four-second response wait is a fixed delay, so it may cut off delayed responses. Adjust it when experimenting or implement explicit completion handling for a production adaptation.
- **Output overwritten:** Each run writes to the same WAV filename unless you change it.

## Source and license

Based on Google's [Multimodal Live API – Translation Quickstart](https://github.com/google-gemini/cookbook/blob/main/quickstarts/Get_started_LiveTranslate.ipynb).

The source notebook states **Copyright 2026 Google LLC** and is licensed under the [Apache License, Version 2.0](https://www.apache.org/licenses/LICENSE-2.0). If you copy or adapt its code into this repository, retain the applicable copyright and license notices and include the required license materials.
