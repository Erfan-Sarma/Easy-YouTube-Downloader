# Easy YouTube Downloader

A simple Google Colab-based YouTube downloader for individual videos, multiple targets, and playlists, including very large playlists.

The notebook provides a straightforward interface for downloading YouTube content directly to Google Drive, with quality selection, automatic retry recovery, subtitle extraction, and persistent playlist progress so completed downloads can be continued across interrupted Google Colab sessions.

## Features

* Download individual YouTube videos, playlists, or multiple links separated by `+`
* Designed to handle very large playlists
* **Download Failure-Retry Mechanism**: Automatically collects failed items during the main loop and retries downloading them in secondary passes
* **Playlist Index Prefixing**: Optional index numbering (`1-video1.mp4`, `2-video2.mp4`) to prevent file managers from scrambling playlist ordering
* **Subtitle Downloader**: Download official or auto-generated subtitles with custom language selection (`en`, `es`, `fr`, etc.)
* **Smart Directory Management**: Select existing Google Drive target folders or create nested directory paths (`Course/Module1`) on the fly
* Three download modes:
  * **Video + Audio**
  * **Audio Only**
  * **Video Only**
* Maximum video quality selection:
  * **Best**
  * **1080p**
  * **720p**
  * **480p**
  * **360p**
* Automatically selects the best available audio stream for **Audio Only**
* Converts audio-only downloads to MP3
* Persistent download archive (`download_archive.txt`) stored in Google Drive
* Detailed human-readable log (`download_history.txt`) with timestamps, video titles, IDs, and links
* Resume completed playlist progress across Google Colab sessions
* **Fast Resume** mode for large sequential playlists
* Interactive `ipywidgets` interface
* Downloads are transferred to Google Drive after each item
* Automatically installs required system dependencies (`FFmpeg`, `Node.js`) and Python packages
* Uses `yt-dlp` for media extraction and downloading

---

## Why This Project?

This project was created around a simple use case: downloading YouTube videos and playlists without having to configure or install a full downloader application.

While working with very large playlists, interrupted download sessions or transient network errors can become inconvenient. Starting again may require checking a large number of already-completed items, or manually identifying individual items that failed during a batch run.

This project focuses specifically on YouTube and keeps the interface and implementation intentionally small.

---

## How It Works

The downloader runs entirely inside **Google Colab**.

The general workflow is:

```text
GitHub
   │
   ▼
Download the .ipynb
   │
   ▼
Open in Google Colab
   │
   ▼
Run the notebook
   │
   ▼
Connect Google Drive
   │
   ▼
Configure download options
   │
   ▼
Download with yt-dlp
   │
   ▼
Transfer completed files to Google Drive
```

No traditional installation is required on your computer.

The notebook automatically checks for and installs the required system packages (`FFmpeg`, `Node.js`) and Python dependencies when necessary.

---

## User Interface

The downloader provides eight main configuration controls.

### 1. YouTube URL(s)

Enter one of the following:

* A single YouTube video URL
* A YouTube playlist URL
* Multiple URLs separated by a `+` symbol (e.g., `URL1 + URL2`)

For example:

```text
[https://www.youtube.com/watch?v=VIDEO_ID](https://www.youtube.com/watch?v=VIDEO_ID)

```

or:

```text
[https://www.youtube.com/playlist?list=PLAYLIST_ID](https://www.youtube.com/playlist?list=PLAYLIST_ID)

```

or multi-target batch input:

```text
[https://www.youtube.com/watch?v=VIDEO_1](https://www.youtube.com/watch?v=VIDEO_1) + [https://www.youtube.com/watch?v=VIDEO_2](https://www.youtube.com/watch?v=VIDEO_2)

```

The notebook automatically extracts flat metadata to determine whether each target is a single video or a playlist.

---

### 2. Directory Selection (Existing Dir / Or New Path)

Choose where your downloaded files will be stored in Google Drive.

* **Existing Dir**: Dropdown menu listing all existing folders inside `YouTubeDownloads`.
* **Or New Path**: Enter a new folder name or nested directory structure using `/` (e.g., `Courses/UE5/Module1`). Text input overrides the dropdown selection if filled.

The destination path in Google Drive will be:

```text
MyDrive/
└── YouTubeDownloads/
    └── Your Folder/

```

For example:

```text
MyDrive/
└── YouTubeDownloads/
    └── My Music/
        ├── 1-Song 01.mp3
        ├── 2-Song 02.mp3
        ├── download_archive.txt
        ├── download_history.txt
        └── ...

```

---

### 3. Download Type

Choose one of the following modes:

| Mode | Description |
| --- | --- |
| **Video + Audio** | Downloads video and audio and merges them into MP4 when possible |
| **Audio Only** | Downloads the best available audio and converts it to MP3 |
| **Video Only** | Downloads video without an audio track |

When **Audio Only** is selected, the downloader uses the best available audio stream and FFmpeg to convert it to MP3 at 192 kbps.

---

### 4. Maximum Quality

The available quality options are:

```text
Best
1080p
720p
480p
360p

```

The selected value acts as the maximum video resolution limit.

For example:

```text
Maximum Quality: 720p

```

allows the downloader to select the best available video stream up to 720p.

Selecting `Best` does not impose a resolution limit. When **Audio Only** is selected, this setting does not affect the download.

---

### 5. Subtitle Downloader

* **Download Subtitles**: Checkbox to extract official or auto-generated subtitles alongside media files (saved as `.srt`).
* **Sub Lang**: Text input for specifying language codes (default: `en`). Supports multiple codes like `en, es, fr`.

---

### 6. Retry Passes (Failure-Retry Mechanism)

Select how many retry passes (`1`, `2`, `3`, or `5`) to perform after the primary download loop finishes.

If any item fails during the main loop (due to transient network issues or rate limits), it is gathered into a retry queue. Once the main pass completes, the downloader pauses to let rate limits cool down, then automatically attempts to download the failed items again.

---

### 7. Add Video Index

Checkbox (enabled by default) that prefixes each file with its original position index (e.g., `1-Video Title.mp4`, `2-Video Title.mp4`).

This ensures file managers and media players preserve the original playlist order without scrambling file sorting.

---

### 8. Fast Resume

**Fast Resume** is designed primarily for large sequential playlists.

When enabled, the downloader reads `download_archive.txt` and identifies the latest completed item in the playlist sequence. It then skips directly past completed items instantly instead of inspecting every item individually.

#### Normal Resume

Without Fast Resume, each playlist item is passed through `yt-dlp`, which checks the archive to determine whether it has already been completed:

```text
Playlist
   │
   ▼
Process items
   │
   ▼
Check download archive
   │
   ├── Already downloaded → Skip
   │
   └── Not downloaded → Download

```

#### Fast Resume

With Fast Resume enabled:

```text
Playlist
   │
   ▼
Read download archive
   │
   ▼
Find latest completed item
   │
   ▼
Skip previous items
   │
   ▼
Continue downloading

```

---

## Resuming Large Playlists & Logging

The downloader uses `download_archive.txt` to keep track of completed downloads and `download_history.txt` to log human-readable details.

Both files are stored inside your destination folder in Google Drive:

```text
MyDrive/
└── YouTubeDownloads/
    └── My Playlist/
        ├── download_archive.txt    <-- Prevents duplicate downloads
        ├── download_history.txt    <-- Timestamped log (Title, ID, Link)
        ├── 1-Video 001.mp4
        ├── 2-Video 002.mp4
        └── ...

```

Because the archive is saved directly to Google Drive rather than only in Colab's temporary storage, progress is retained across sessions:

```text
Session 1
─────────
Video 1   ✓
Video 2   ✓
Video 3   ✓
...
Video 500 ✓

```

If the Colab session ends:

```text
Session 2
─────────
Read download_archive.txt
        ↓
Recognize completed videos
        ↓
Continue with remaining playlist items

```

---

## Google Colab Runtime Limitations

Google Colab runtimes are temporary. The downloader uses two storage locations:

### Temporary download storage

```text
/content/YT-Temp

```

Files are downloaded and processed here first.

### Persistent storage

```text
/content/drive/MyDrive/YouTubeDownloads/<Folder>/

```

Completed files are transferred to Google Drive immediately after each item finishes downloading.

This distinction is important: completed files transferred to Google Drive remain safe, but a file currently mid-download when a Colab runtime terminates will need to be restarted on the next session.

---

## Installation / Usage

No traditional installation is required.

### Step 1 — Download the Notebook

Download `Easy-YouTube-Downloader.ipynb` from this repository.

### Step 2 — Open It in Google Colab

Upload the notebook to [Google Colab](https://colab.research.google.com/?utm_source=gemini) or open your copy directly.

### Step 3 — Run the Notebook

Run the notebook cells from top to bottom.

The notebook automatically checks for and installs all dependencies:

* `yt-dlp`
* `ipywidgets`
* `FFmpeg`
* `Node.js`

### Step 4 — Configure the Downloader

Fill in the interface fields:

* **YouTube URL(s)**
* **Directory / Folder Name**
* **Download Type**
* **Maximum Quality**
* **Subtitle Options**
* **Retry Passes**
* **Add Video Index**
* **Fast Resume**

### Step 5 — Start the Download

Press **Start Download**. The notebook will mount Google Drive (if not already mounted) and begin processing your target links.

---

## Example

Suppose you want to download a large educational series with subtitles and numerical ordering.

Configure the downloader like this:

```text
YouTube URL(s):
[https://www.youtube.com/playlist?list=](https://www.youtube.com/playlist?list=)...

Or New Path:
Courses/Python_101

Download Type:
Video + Audio

Maximum Quality:
1080p

Download Subtitles:
[X] Enabled (Lang: en)

Retry Passes:
3

Add Video Index:
[X] Enabled

Fast Resume:
[X] Enabled

```

The resulting files in Google Drive:

```text
MyDrive/YouTubeDownloads/Courses/Python_101/
├── 1-Introduction to Python.mp4
├── 1-Introduction to Python.en.srt
├── 2-Variables and Data Types.mp4
├── 2-Variables and Data Types.en.srt
├── download_archive.txt
└── download_history.txt

```

If the session is interrupted, simply re-run the notebook with the same target folder to resume where you left off.

---

## Technology

The project uses:

* **Python** — core application logic
* **Google Colab** — cloud execution environment
* **yt-dlp** — media extraction and downloading engine
* **FFmpeg** — multimedia processing and audio conversion
* **Node.js** — JavaScript engine for `yt-dlp` to resolve YouTube JS signature runtime requirements
* **ipywidgets** — interactive user interface widgets
* **Google Drive** — persistent cloud file storage

---

## Project Structure

```text
Easy-YouTube-Downloader/
├── Easy-YouTube-Downloader.ipynb
├── README.md
├── LICENSE
└── ATTRIBUTIONS.md

```

The notebook contains the complete application code.

---

## Inspiration

This project was inspired by [AzuDL-GC2GD](https://github.com/TheGreatAzizi/AzuDL-GC2GD?utm_source=gemini), a Google Colab-based universal downloader supporting YouTube and several other download workflows.

While using AzuDL primarily for YouTube downloads, I wanted a smaller and more specialized tool focused specifically on YouTube videos and playlists, particularly for long-running playlist downloads with built-in retry mechanisms and playlist index ordering.

Some utility and Google Colab / Google Drive handling code was adapted from the AzuDL-GC2GD implementation under the MIT License. See [ATTRIBUTIONS.md](https://www.google.com/search?q=ATTRIBUTIONS.md&utm_source=gemini) for details.

---

## Responsible Use

This software is intended for downloading content that you have permission or legal rights to download. Users are responsible for complying with applicable laws, copyright restrictions, and the terms of service of the services they access.

---

## License

This project is distributed under the **MIT License**. See [LICENSE](https://www.google.com/search?q=LICENSE&utm_source=gemini) for details.

## Acknowledgements

* [yt-dlp](https://github.com/yt-dlp/yt-dlp?utm_source=gemini) — media extraction and downloading
* [FFmpeg](https://ffmpeg.org/?utm_source=gemini) — multimedia processing
* [Node.js](https://nodejs.org/?utm_source=gemini) — JavaScript engine execution
* [AzuDL-GC2GD](https://github.com/TheGreatAzizi/AzuDL-GC2GD?utm_source=gemini) — inspiration and adapted utility code for Google Colab / Google Drive workflows

```

```
