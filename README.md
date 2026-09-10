# Easy YouTube Downloader

A simple and efficient Google Colab-based YouTube downloader designed for individual videos, small playlists, and very large playlists.

The notebook provides a straightforward interface for downloading YouTube content directly to Google Drive, with quality selection and persistent playlist progress so interrupted sessions can be continued later.

## Features

* Download individual YouTube videos
* Download YouTube playlists
* Supports very large playlists
* Download modes:

  * Video + Audio
  * Audio Only
  * Video Only
* Maximum quality selection:

  * Best
  * 1080p
  * 720p
  * 480p
  * 360p
* Automatic best-quality audio selection when using **Audio Only**
* Persistent download archive stored in Google Drive
* Resume playlist progress across Google Colab sessions
* **Fast Resume** mode for large playlists
* Simple Google Colab widget-based interface
* Downloads are stored directly in Google Drive
* Automatically installs required dependencies
* Uses `yt-dlp` for YouTube extraction and downloading
* Uses FFmpeg for audio extraction and media processing

---

## Why This Project?

This project was created around a simple use case: downloading YouTube videos and playlists reliably without having to configure a full downloader application.

While working with very large playlists, restarting a download session can become inconvenient. A downloader may need to determine which items have already been completed before continuing.

This project therefore focuses specifically on YouTube and keeps the interface and implementation intentionally small.

The goal is not to be a universal downloader, but to provide a simple tool that can handle both ordinary downloads and long-running playlist jobs.

---

## How It Works

The project runs entirely through **Google Colab**.

The basic workflow is:

1. Download the `.ipynb` notebook from this repository.
2. Upload/open it in Google Colab.
3. Run the notebook.
4. Grant the notebook access to your Google Drive.
5. Enter the download settings.
6. Press **Start Download**.
7. Files are downloaded and stored in your Google Drive.

The notebook automatically installs `yt-dlp`, `ipywidgets`, and FFmpeg when necessary.

---

## User Interface

The downloader provides a simple interface with the following options:

### 1. YouTube URL

Paste either:

* A single YouTube video URL
* A YouTube playlist URL

Example:

```text
https://www.youtube.com/watch?v=VIDEO_ID
```

or

```text
https://www.youtube.com/playlist?list=PLAYLIST_ID
```

### 2. Folder Name

Specify the folder where the downloaded files should be stored.

The folder is created inside:

```text
Google Drive
└── MyDrive
    └── YouTubeDownloads
        └── Your Folder
```

The notebook automatically creates this directory when necessary.

### 3. Download Type

Choose between:

| Mode              | Description                                                               |
| ----------------- | ------------------------------------------------------------------------- |
| **Video + Audio** | Downloads video and audio together and merges them into MP4 when possible |
| **Audio Only**    | Downloads the best available audio and converts it to MP3                 |
| **Video Only**    | Downloads video without an audio track                                    |

The audio-only mode uses FFmpeg to extract MP3 audio at 192 kbps.

### 4. Maximum Quality

Available options:

```text
Best
1080p
720p
480p
360p
```

The selected value acts as a maximum video resolution. When **Best** is selected, the downloader does not impose a resolution limit.

For **Audio Only**, the quality setting is ignored and the best available audio stream is selected.

### 5. Fast Resume

The **Fast Resume** option is intended primarily for large playlists.

When enabled, the downloader uses its persistent download archive to identify the latest completed item and skips the items before it rather than checking every item individually.

```text
Normal operation:

Playlist
   ↓
Process playlist items
   ↓
Check download archive
   ↓
Skip completed items
   ↓
Continue downloading
```

With Fast Resume:

```text
Playlist
   ↓
Read download archive
   ↓
Find last completed item
   ↓
Skip directly to the remaining items
```

This can significantly reduce the amount of work required when continuing a large playlist.

> **Note:** Fast Resume assumes that the playlist order has not changed significantly since the previous session. If the playlist has been reordered or items have been inserted/deleted, normal resume behavior is safer.

---

## Resuming Large Playlists

Download progress is stored in a `download_archive.txt` file inside the selected Google Drive folder.

For example:

```text
MyDrive/
└── YouTubeDownloads/
    └── MyPlaylist/
        ├── download_archive.txt
        ├── Video 001.mp4
        ├── Video 002.mp4
        ├── Video 003.mp4
        └── ...
```

Because the archive is stored in Google Drive rather than only in the temporary Colab environment, the completed playlist progress remains available when the notebook is started again.

This is particularly useful for playlists containing hundreds or thousands of items.

### Important

Google Colab runtimes are temporary and can disconnect or reset. This project does **not** bypass Colab's runtime limitations.

The persistent archive allows the notebook to continue previously completed playlist progress after starting a new session, but files that are still being processed in Colab's temporary storage may need to be downloaded again after a runtime reset.

---

## Installation / Usage

No traditional installation is required.

### Step 1 — Download the Notebook

Download:

```text
Easy-YouTube-Downloader.ipynb
```

from this repository.

### Step 2 — Open Google Colab

Open the notebook in Google Colab.

### Step 3 — Run the Notebook

Run the cells from top to bottom.

The notebook will automatically check for and install the required dependencies:

* Python
* `yt-dlp`
* `ipywidgets`
* FFmpeg

### Step 4 — Configure the Downloader

Fill in:

```text
YouTube URL
Folder Name
Download Type
Maximum Quality
Fast Resume
```

### Step 5 — Start Downloading

Press:

```text
Start Download
```

The notebook will mount Google Drive and begin processing the requested video or playlist.

---

## Example

Suppose you want to download a large music playlist as MP3 files.

Configure the interface like this:

```text
YouTube URL:
https://www.youtube.com/playlist?list=...

Folder Name:
My Music Playlist

Download Type:
Audio Only

Max Quality:
Best

Fast Resume:
Enabled
```

The resulting files will be stored in:

```text
MyDrive/YouTubeDownloads/My Music Playlist/
```

If the Colab session later ends, run the notebook again and use the same folder. The existing `download_archive.txt` allows the downloader to recognize previously completed items and continue the playlist.

---

## Technology

This project is built around a small set of Python and Google Colab technologies:

* **Python**
* **Google Colab**
* **yt-dlp** — YouTube extraction and downloading
* **FFmpeg** — audio extraction and media processing
* **ipywidgets** — interactive Colab interface
* **Google Drive** — persistent download storage

The notebook uses `yt-dlp` directly rather than implementing its own YouTube extraction layer.

---

## Project Structure

The repository intentionally contains very little:

```text
Easy-YouTube-Downloader/
├── Easy-YouTube-Downloader.ipynb
├── LICENSE
├── ATTRIBUTIONS.md
└── README.md
```

The notebook contains the complete application.

---

## Inspiration

This project was inspired by [AzuDL-GC2GD](https://github.com/TheGreatAzizi/AzuDL-GC2GD), a Google Colab-based universal downloader that supports YouTube downloads along with many other download workflows.

While using AzuDL primarily for YouTube downloads, I wanted a smaller and more specialized tool focused specifically on YouTube videos and playlists, particularly for long-running playlist downloads.

Some utility and Google Colab/Google Drive handling code was adapted from AzuDL-GC2GD. The project was then substantially simplified and reorganized around a YouTube-specific workflow, including the downloader interface, format selection, playlist processing, download archive handling, and Fast Resume behavior.

AzuDL-GC2GD is distributed under the MIT License. See [`ATTRIBUTIONS.md`](ATTRIBUTIONS.md) for attribution details.

---

## Limitations

This project intentionally focuses on a narrow use case.

It currently does **not** aim to provide:

* Torrent downloading
* Direct HTTP/FTP downloading
* GitHub repository downloading
* Batch downloading of unrelated URLs
* Archive management
* Download history management
* A standalone desktop application

If you need a general-purpose downloader with these capabilities, a more comprehensive downloader may be a better choice.

---

## Legal / Responsible Use

This software is intended for downloading content that you have permission or legal rights to download.

Users are responsible for complying with applicable laws, copyright restrictions, and the terms of service of the services they access.

The author does not provide or distribute copyrighted media through this project.

---

## License

This project is distributed under the **MIT License**.

See [`LICENSE`](LICENSE) for the complete license text.

## Acknowledgements

* [yt-dlp](https://github.com/yt-dlp/yt-dlp) — media extraction and downloading
* [FFmpeg](https://ffmpeg.org/) — multimedia processing
* [AzuDL-GC2GD](https://github.com/TheGreatAzizi/AzuDL-GC2GD) — inspiration and adapted utility code for parts of the Colab/Google Drive workflow
* Google Colab — notebook execution environment
* Google Drive — persistent storage
