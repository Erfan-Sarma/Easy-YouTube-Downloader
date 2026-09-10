# Easy YouTube Downloader

A simple Google Colab-based YouTube downloader for individual videos and playlists, including very large playlists.

The notebook provides a straightforward interface for downloading YouTube content to Google Drive, with quality selection and persistent playlist progress so completed downloads can be continued across interrupted Google Colab sessions.

## Features

* Download individual YouTube videos
* Download YouTube playlists
* Designed to handle very large playlists
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
* Persistent download archive stored in Google Drive
* Resume completed playlist progress across Google Colab sessions
* **Fast Resume** mode for large sequential playlists
* Simple `ipywidgets` interface
* Downloads are transferred to Google Drive after each item
* Automatically installs required dependencies
* Uses `yt-dlp` for media extraction and downloading
* Uses FFmpeg for media processing

---

## Why This Project?

This project was created around a simple use case: downloading YouTube videos and playlists without having to configure or install a full downloader application.

While working with very large playlists, interrupted download sessions can become inconvenient. Starting again may require checking a large number of already-completed items before the downloader can continue.

This project focuses specifically on YouTube and keeps the interface and implementation intentionally small.

The goal is not to be a universal downloader. Instead, it provides a focused workflow for YouTube downloads, with particular attention to long-running playlist jobs and the ability to continue completed playlist progress across multiple Google Colab sessions.

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

The notebook automatically checks for and installs the required Python packages and FFmpeg when necessary.

---

## User Interface

The downloader provides five main configuration options.

### 1. YouTube URL

Enter either:

* A single YouTube video URL
* A YouTube playlist URL

For example:

```text
https://www.youtube.com/watch?v=VIDEO_ID
```

or:

```text
https://www.youtube.com/playlist?list=PLAYLIST_ID
```

The notebook automatically determines whether the provided URL represents a single video or a playlist.

---

### 2. Folder Name

Specify the folder where the downloaded files should be stored.

The folder is created under:

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
        ├── Song 01.mp3
        ├── Song 02.mp3
        └── ...
```

The folder is also used to store the persistent download archive.

---

### 3. Download Type

Choose one of the following modes:

| Mode              | Description                                                      |
| ----------------- | ---------------------------------------------------------------- |
| **Video + Audio** | Downloads video and audio and merges them into MP4 when possible |
| **Audio Only**    | Downloads the best available audio and converts it to MP3        |
| **Video Only**    | Downloads video without an audio track                           |

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

The selected value acts as the maximum video resolution.

For example:

```text
Maximum Quality: 720p
```

allows the downloader to select the best available video stream up to 720p.

Selecting:

```text
Maximum Quality: Best
```

does not impose a resolution limit.

When **Audio Only** is selected, this setting does not affect the audio download. The best available audio stream is selected instead.

---

### 5. Fast Resume

**Fast Resume** is designed primarily for large playlists.

When enabled, the downloader reads the persistent `download_archive.txt` and identifies the latest completed item in the playlist. It then skips directly to the items after it instead of checking every previous item individually.

This can significantly reduce the amount of work required when continuing a large playlist.

#### Normal Resume

Without Fast Resume, each playlist item is passed through `yt-dlp`, which uses the download archive to determine whether the item has already been completed.

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

### Fast Resume Considerations

Fast Resume is optimized for playlists that are downloaded sequentially from beginning to end.

It assumes that completed downloads form a continuous sequence from the beginning of the playlist.

For example:

```text
Video 1 ✓
Video 2 ✓
Video 3 ✓
Video 4 ✓
Video 5 ← Continue here
```

It is therefore recommended to use **normal resume** if:

* The playlist has been significantly reordered
* Videos have been inserted or removed
* An earlier video failed while later videos were completed
* You want every playlist item to be individually checked against the archive

---

## Resuming Large Playlists

The downloader uses `download_archive.txt` to keep track of completed downloads.

The archive is stored inside the selected Google Drive folder:

```text
MyDrive/
└── YouTubeDownloads/
    └── My Playlist/
        ├── download_archive.txt
        ├── Video 001.mp4
        ├── Video 002.mp4
        ├── Video 003.mp4
        └── ...
```

Because the archive is stored in Google Drive rather than only in the temporary Google Colab environment, completed playlist progress remains available when the notebook is started again.

For example:

```text
Session 1
─────────
Video 1   ✓
Video 2   ✓
Video 3   ✓
Video 4   ✓
...
Video 500 ✓
```

If the Google Colab session ends:

```text
Session 2
─────────
Read download_archive.txt
        ↓
Recognize completed videos
        ↓
Continue with remaining playlist items
```

This makes the downloader particularly useful for playlists containing hundreds or thousands of items.

---

## Google Colab Runtime Limitations

Google Colab runtimes are temporary.

The downloader therefore uses two different storage locations:

### Temporary download storage

```text
/content/YT-Temp
```

Files are downloaded here first.

### Persistent storage

```text
/content/drive/MyDrive/YouTubeDownloads/
```

Completed files are then transferred to Google Drive.

This distinction is important.

The project can preserve **completed playlist progress** across Google Colab sessions through the download archive stored in Google Drive.

However, it does **not** guarantee recovery of a file that was still being downloaded when the Colab runtime was terminated.

For long-running downloads, it is therefore recommended to allow completed files to be transferred to Google Drive before the session ends.

---

## Installation / Usage

No traditional installation is required.

### Step 1 — Download the Notebook

Download:

```text
Easy-YouTube-Downloader.ipynb
```

from this repository.

### Step 2 — Open It in Google Colab

Upload the notebook to Google Colab or open the notebook from your downloaded copy.

### Step 3 — Run the Notebook

Run the notebook from top to bottom.

The notebook automatically checks for the required dependencies and installs them if necessary:

* `yt-dlp`
* `ipywidgets`
* FFmpeg

### Step 4 — Configure the Downloader

Fill in the interface:

```text
YouTube URL
Folder Name
Download Type
Maximum Quality
Fast Resume
```

### Step 5 — Start the Download

Press:

```text
Start Download
```

The notebook will mount Google Drive and begin processing the provided URL.

---

## Example

Suppose you want to download a large music playlist as MP3 files.

Configure the downloader like this:

```text
YouTube URL:
https://www.youtube.com/playlist?list=...

Folder Name:
My Music

Download Type:
Audio Only

Maximum Quality:
Best

Fast Resume:
Enabled
```

The files will be stored in:

```text
MyDrive/YouTubeDownloads/My Music/
```

If the Google Colab session is interrupted, start the notebook again and use the same folder.

The existing:

```text
download_archive.txt
```

allows the downloader to recognize previously completed items and continue the playlist.

---

## Large Playlist Testing

The downloader was tested with a YouTube playlist containing approximately **2,500 items** in Audio Only mode across multiple Google Colab sessions.

The test was used to verify that:

* Large playlists can be processed
* Completed items are recorded in the download archive
* Files are transferred to Google Drive
* Progress remains available after starting a new Colab session
* Fast Resume can skip previously completed playlist items

This project is intended to be practical for long-running playlist downloads rather than only small one-off downloads.

---

## Technology

The project uses:

* **Python** — application logic
* **Google Colab** — execution environment
* **yt-dlp** — media extraction and downloading
* **FFmpeg** — media processing and audio conversion
* **ipywidgets** — interactive user interface
* **Google Drive** — persistent file storage

The project uses `yt-dlp` for the actual YouTube extraction and downloading rather than implementing its own YouTube extraction system.

---

## Project Structure

The repository intentionally contains very little:

```text
Easy-YouTube-Downloader/
├── Easy-YouTube-Downloader.ipynb
├── README.md
├── LICENSE
└── ATTRIBUTIONS.md
```

The notebook contains the complete application.

---

## Inspiration

This project was inspired by [AzuDL-GC2GD](https://github.com/TheGreatAzizi/AzuDL-GC2GD), a Google Colab-based universal downloader supporting YouTube and several other download workflows.

While using AzuDL primarily for YouTube downloads, I wanted a smaller and more specialized tool focused specifically on YouTube videos and playlists, particularly for long-running playlist downloads.

Some utility and Google Colab / Google Drive handling code was adapted from the AzuDL-GC2GD implementation.

The project was subsequently simplified and reorganized around a YouTube-specific workflow, including:

* The downloader interface
* Format selection
* Quality selection
* Playlist processing
* Download archive handling
* Google Drive file management
* Fast Resume behavior

AzuDL-GC2GD is distributed under the MIT License.

See [ATTRIBUTIONS.md](ATTRIBUTIONS.md) for the attribution and applicable license text.

---

## Limitations

This project intentionally focuses on a narrow use case.

It currently does not aim to provide:

* Torrent downloading
* Generic HTTP/FTP downloading
* GitHub repository downloading
* Batch downloading of unrelated URLs
* Archive management
* A general download history system
* A standalone desktop application
* A graphical desktop interface

If you need a general-purpose downloader with these capabilities, a more comprehensive downloader may be more appropriate.

---

## Responsible Use

This software is intended for downloading content that you have permission or legal rights to download.

Users are responsible for complying with applicable laws, copyright restrictions, and the terms of service of the services they access.

This project does not provide or distribute copyrighted media.

---

## License

This project is distributed under the **MIT License**.

See [LICENSE](LICENSE) for the complete license text.

## Acknowledgements

* [yt-dlp](https://github.com/yt-dlp/yt-dlp) — media extraction and downloading
* [FFmpeg](https://ffmpeg.org/) — multimedia processing
* [AzuDL-GC2GD](https://github.com/TheGreatAzizi/AzuDL-GC2GD) — inspiration and adapted utility code for parts of the Google Colab / Google Drive workflow
* Google Colab — notebook execution environment
* Google Drive — persistent storage
