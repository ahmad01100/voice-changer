# Voice Changer

**English** | [العربية](README.ar.md)

**A Windows desktop app for local voice conversion, reusable voice profiles, fine-tuning, and live microphone processing.**

**Voice Changer — مغير الأصوات**

> The current interface is Arabic. Processing runs locally after the required dependencies and models are downloaded.

## Overview

Voice Changer is a Windows desktop application with an **Arabic interface**. Record speech or open an audio file, then convert its speaker characteristics using one or more reference clips. Save reusable voice profiles, fine-tune a speaker-specific version, and compare base-model and fine-tuned results on the same speech.

The conversion pipeline uses [Seed-VC v1](https://github.com/Plachtaa/seed-vc), Whisper-small, and BigVGAN. A separate pitch-shifting mode produces a higher or deeper voice without AI voice imitation.

## Planned availability, languages, and pricing

The program is planned to be available to everyone, with support for multiple languages **to be announced later**. The current interface is Arabic; additional languages are planned for a future release. Availability means anyone can obtain and use the program subject to its system requirements and chosen plan.

| Plan | Price in USD | Term |
| --- | --- | --- |
| Annual subscription | **$69** | One year |
| Perpetual license | **$199** | One-time payment, no annual renewal |

These are the planned commercial-release prices. Launch, activation, and plan terms will be announced later; the current project does not include a purchase or subscription-activation system.

## Accuracy

**We aim to achieve 98% accuracy; this figure has not yet been validated through comprehensive testing.** It is a development target, not a measured result for the current version.

The program aims to approximate the reference speaker's vocal characteristics while preserving the original speech. It uses word, character, and acoustic checks to select an output. However, **no accuracy percentage has currently been established through comprehensive evaluation**, and 100% voice matching is not guaranteed.

Results depend on recording clarity, language and dialect, reference voice, training, and settings. Some consonants or words may still be pronounced incorrectly. Internal speaker-similarity scores are not application accuracy percentages. Speaker resemblance, content preservation, and naturalness need evaluation across diverse recordings before a performance percentage can be published.

## Features

- Microphone recording and WAV, MP3, FLAC, and OGG input.
- Reference-based voice conversion without mandatory fine-tuning.
- Named voice profiles with up to **10 reference clips**, each **3–25 seconds** long.
- Multiple excerpts from one recording, such as `10–30` and `60–80` seconds.
- Reference preview, addition, removal, and confirmed deletion of a profile and its associated training files.
- Local fine-tuning with **100, 300, or 600 training steps**.
- Separate playback and export of base-model and fine-tuned comparison results.
- Three file-conversion quality modes, including candidate ranking with content and audio checks.
- Live microphone conversion to a selected output device or virtual audio cable.
- WAV, MP3, OGG/Opus, M4A/AAC, FLAC, and black-screen MP4 export.
- Conversion/training cancellation and retention of the last successful result during the session.

## Requirements

| Component | Details |
| --- | --- |
| Operating system | Windows; the launch and setup scripts target Windows |
| Python | **64-bit Python 3.12**, used in the development environment, with Tkinter and the `py` launcher |
| GPU | NVIDIA/CUDA required for fine-tuning and live conversion; file conversion has a slower CPU path |
| Internet | Needed for initial dependency/model downloads and first-time training components |
| Storage | Several GB for dependencies and weights, plus additional space for saved training runs |
| Audio hardware | Microphone for recording; headphones preferred for live monitoring |
| FFmpeg | Used for compressed audio/video export; discovered through PATH or `imageio-ffmpeg` |

AI setup installs PyTorch 2.5.1 with CUDA 12.4 wheels. Newer Python versions are not automatically compatible with the pinned dependencies. Training and conversion paths have been tested on an RTX 3060 Laptop GPU with 6 GB VRAM; this is a tested configuration, not a guaranteed minimum for every workload.

## Installation

1. Download or clone the repository and open its folder.
2. Install 64-bit Python 3.12 with Tkinter and Python Launcher.
3. Run `setup.bat` to create the UI environment and install audio dependencies. It launches the app when finished.
4. For AI conversion, run `setup_ai.bat` and wait for the engine and model downloads. Close and reopen the app after setup if necessary.
5. Use `start.bat` for subsequent launches.

Equivalent PowerShell commands, run in order from the project folder:

```powershell
.\setup.bat
.\setup_ai.bat
.\start.bat
```

The project uses separate `.venv` and `.venv-ai` environments for the UI and AI pipeline. Basic pitch shifting does not require AI setup.

## Basic workflow

1. Open or record the speech to convert. Source recordings are limited to **5 minutes** and stereo input is converted to mono.
2. Select **«بدء صوت جديد بعينات جديدة»** (start a new voice with new samples).
3. Choose reference files belonging to the same speaker. Set excerpt start/end times in seconds or `minutes:seconds`; Arabic digits are accepted.
4. Add further excerpts with **«إضافة الجزء إلى القائمة»**, then confirm using **«اعتماد جميع الأجزاء»**.
5. Select **«حفظ كنموذج…»** (save as a profile) and enter a name for reuse.
6. Choose a quality mode and select **«تحويل إلى صوت العيّنة»** (convert to the reference voice).
7. Listen, select an export format, and save the result.

**Saving a profile stores references and settings. Fine-tuning creates custom weights. Neither operation alone creates a converted recording.** Result playback becomes available after successful conversion.

Saved excerpts are independent copies, so the original reference file is no longer required. Appending references to an existing saved profile persists them directly. Selecting one reference for preview does not exclude the other references from conversion.

## Quality and fine-tuning

| File-conversion mode | Behavior |
| --- | --- |
| Balanced / متوازن — أسرع | One candidate, 30 generation steps |
| High / جودة عالية | One candidate, 80 generation steps |
| Similarity and clarity / تشابه ووضوح — 3 محاولات | Three candidates using 30, 80, and 80 generation steps |

The three-candidate mode compares recognized words and characters, checks short segments for acoustic support of source speech, and uses speaker similarity and signal checks to resolve remaining choices. With a fine-tuned voice, it also tests reduced fine-tuning influence by blending model weights **in memory**, without overwriting saved checkpoints. Clarity can take priority over speaker similarity. Selection details and unavailable checks are recorded in `logs/last_quality.json`.

To fine-tune, save and select a voice, choose 100/300/600 steps, and press **«تدريب الصوت المحفوظ»**. The first run downloads OpenVoiceV2 training components. After completion, enable **«استخدام النسخة المدرّبة»**, or generate both versions for comparison.

- Training steps are separate from generation steps in the quality table.
- Each new run starts from the base model and saves a separate version linked to the voice profile.
- When enough chunks are available, one is held out and three training stages are evaluated; the lowest validation-loss stage is retained. Otherwise, the final stage is saved.
- New training runs freeze content-conditioning layers to limit pronunciation drift.
- Editing references makes previous training out of date; retrain to use the new reference set.
- Cancellation or failure preserves the previous saved version. Interrupted runs cannot currently be resumed.

## Live microphone conversion

In **«الصوت المباشر للمكالمات»**, select the microphone, saved voice, output device, and quality. Start processing and wait for readiness.

To route converted audio to a calling app through [VB-CABLE](https://vb-audio.com/Cable/):

1. Install the virtual cable and reopen Voice Changer.
2. Select **CABLE Input** as this app's live output.
3. Select **CABLE Output** as the calling app's microphone, and headphones for listening to others.
4. Start live processing and use the calling app. If it has no input selector, check its audio settings or Windows device settings.

| Live quality | Generation steps | Maximum reference prompt |
| --- | --- | --- |
| Fast | 6 | 3 seconds |
| Higher quality | 10 | 6 seconds |
| Maximum quality | 14 | 8 seconds |

Block duration is a separate choice: 0.6 or 1 second. Try 1-second blocks with maximum quality and monitor slow-processing/dropout warnings. Live conversion is experimental and adds hardware- and application-dependent latency. It does not provide a direct WhatsApp/Facebook integration or send messages automatically.

## Export formats

| Format | Output |
| --- | --- |
| WAV | 16-bit PCM audio |
| MP3 | 192 kbps compressed audio |
| OGG / Opus | Mono, 48 kHz, 64 kbps |
| M4A / AAC | 128 kbps compressed audio |
| FLAC | Lossless compression of 16-bit PCM audio |
| MP4 | Black 1280×720 H.264 video with AAC audio |

An OGG/Opus file is not automatically a native WhatsApp voice message. Its presentation depends on the application and sending method. Exports are saved locally.

## Limitations and privacy

- **No guarantee of 100% voice matching or perfect pronunciation.** Words and consonants may change, especially with dialects or difficult recordings. Automated quality checks and validation loss are not substitutes for listening.
- More training or generation steps do not necessarily improve results. Compare both versions by ear.
- Export results before closing the app; unsaved session audio is not restored on restart.
- Confirmed profile deletion removes the associated samples and training files identified by the app. Back up anything you need.
- Processing is local; the app does not upload recordings to a conversion service. Initial setup connects to dependency/model download providers.
- Use your own voice or voices you have permission to use, and avoid misleading impersonation.

## Troubleshooting

| Issue | Suggested action |
| --- | --- |
| `py` is not recognized | Install Python Launcher and reopen the terminal |
| AI model is not ready | Complete `setup_ai.bat` and inspect any setup error |
| Result playback is disabled | Complete a conversion; saving or training a profile does not generate speech |
| Playback stutters | Export WAV and play it outside the app to distinguish generation from playback problems |
| Live processing is slow / GPU memory is exhausted | Lower quality, use longer blocks, and close GPU-heavy applications |
| MP3/MP4 export fails | Check FFmpeg or rerun UI dependency setup; try WAV |
| Some consonants change | Try the three-candidate mode and compare base/fine-tuned output; retain source and result for diagnosis |

Main logs: `logs/conversion.log`, `logs/live.log`, and `logs/last_quality.json`. Training logs/reports live inside each run under `profiles/library/training/`.

## Preparing a GitHub repository

Publish source files, tests, requirements, setup scripts, and this README. The current `.gitignore` excludes downloaded environments, model data, and local artifacts:

```text
.venv/       .venv-ai/    vendor/      models/
profiles/   logs/        test-output/ __pycache__/
*.log       *.wav
```

Review staged files before publishing: the current rules do not exclude every media format, such as MP3, OGG, or MP4. Keep private recordings, trained weights, and test outputs out of the source repository. Users obtain the engine and models through setup rather than a copy of the entire local project folder.

## Components and licensing

The interface uses Python/Tkinter, with NumPy, librosa, sounddevice, and SoundFile for audio. PyTorch/Seed-VC handle conversion, CAMPPlus supports speaker comparison, OpenVoiceV2 supports training, and FFmpeg handles media export.

There is currently no standalone application-level `LICENSE` file in this repository. Seed-VC carries a [GPL-3.0 license](https://github.com/Plachtaa/seed-vc/blob/main/LICENSE); other dependencies and model weights retain their own licenses. Document the application's license and retain third-party notices when preparing a public release.


## Project layout

```text
main.py                    Arabic desktop interface
voice_profiles.py          Saved voice library
profile_deletion.py        Profile deletion planning and cleanup
reference_selection.py     Multiple excerpt selection
reference_engine.py        Isolated conversion process management
quality_conversion.py      Candidate generation and selection
candidate_quality.py       Text, acoustic-support, and signal checks
training_strength.py       In-memory fine-tuning influence
training_service.py        Training lifecycle and publication
training_worker.py         Local training process
training_selection.py      Held-out validation and stage selection
training_content.py        Content-layer freezing
live_service.py            Live-session lifecycle
live_worker.py              Audio device routing and processing
live_engine.py             Streaming conversion
live_quality.py            Live quality presets
media_export.py            Audio and black-screen video export
setup.bat                  UI dependency setup
setup_ai.bat                AI setup entry point
setup_ai.py                 Engine and model setup
start.bat                   Application launcher
test_*.py                  Automated tests
```

## Tests

Run UI/audio tests in the UI environment; Tkinter tests need a desktop session and may create hidden windows. Run targeted PyTorch checks in the AI environment. Passing tests does not establish perceptual voice fidelity.

```powershell
.\.venv\Scripts\python.exe -m unittest discover -p "test_*.py"
.\.venv-ai\Scripts\python.exe -m unittest test_candidate_quality test_training_strength test_training_content test_training_selection test_continuity
```
