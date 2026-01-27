YouTube Downloader (CustomTkinter + yt-dlp)

A modern, clean YouTube downloader built with Python, CustomTkinter, and yt-dlp.
This application lets you download YouTube videos or audio in multiple formats and qualities with a YouTube-styled dark mode interface.

✨ Features

YouTube-style dark UI

Download Audio (MP3, M4A, WAV) or Video (MP4, WEBM, MKV)

Audio quality options: 128–320 kbps or Lossless

Video quality options: 360p to 4K

Real-time download progress, speed, ETA, and file size

Multithreaded downloads (GUI remains responsive)

Cancel downloads anytime

Select custom save directory

Automatic post-processing via FFmpeg

Full error handling and user notifications

🛠️ Technologies Used

Python 3

CustomTkinter

yt-dlp

FFmpeg

📦 Installation
1. Clone the repository
git clone https://github.com/your-username/youtube-downloader.git
cd youtube-downloader

2. Install required packages
pip install customtkinter yt-dlp

3. Install FFmpeg

FFmpeg is required for audio extraction and video merging.

Windows: Download from ffmpeg.org and add to PATH
macOS:

brew install ffmpeg


Linux:

sudo apt install ffmpeg

🚀 Running the Application
python main.py

📁 Project Structure
.
│── main.py
│── downloads/        # Default download location (auto-created)
│── README.md

⚙️ How the App Works

Enter a YouTube URL

Select Type (Audio/Video)

Select Format

Choose Quality

Pick a save directory (optional)

Click Download

Progress bar updates in real-time

Shows speed, ETA, and percentage

Download can be cancelled anytime

🧩 Key Code Features

Threading keeps the UI responsive during downloads

yt-dlp progress hooks update the UI with progress and speed

Dynamic dropdowns adapt to the selected type and format

Graceful cancellation of downloads

Error handling for missing URLs, invalid links, FFmpeg issues, and more

🛡️ Error Handling

The app provides clear alerts for:

Invalid or missing URLs

Failed downloads

Missing FFmpeg

User-cancelled downloads

Conversion or merge errors

🤝 Contributing

Contributions, ideas, and improvements are welcome.

📜 License

This project is licensed under the MIT License.