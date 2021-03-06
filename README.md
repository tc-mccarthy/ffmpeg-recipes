# ffmpeg-recipes

Scripts for different video transcode scenarios. These scripts are written for ZSH on OSX but should work/can be adapted for any OS and shell.

## Getting started

All of the recipes in this repository can be tweaked at will. To ensure you have all of the dependencies leveraged in the recipes, please run `./setup` before you attempt to run one of these.

## Plex Optimize

I have used Plex for a long time and spent a lot of time tweaking my various videos to play nicely in PMS, which I have running on a 2015 MacBook Pro. I wanted to reduce the realtime transcode overhead but also minimize the amount of storage each file requires. Since my Plex content had a lot of different source types (I've been producing video since 2004, so I had SD, HD, AVI, h264, DV, MOV, etc) I wanted to be able to standardize them, eliminate the originals (I made copies onto dedicated USB 3.0 hard drives to feed Plex) but also preserve some variances in things like audio.

My `plex-optimize` script is tuned for my setup, but it should be easy to adapt it for other purposes. Simply update the PATHS variable (it expects a BASH array) with all of the different directories that contain your movie files. It generates a list of files to be converted and then loops through. It runs ffprobe on each so that I can see if any bitrate adjustments need to be made. I also have logic for upscaling to 1080p for 16:9 and 4:3 aspect ratios.

If my source material has more than two channels of audio, I map it twice. The primary mapping gets downmixed to stereo (2 channels) and the original audio line is limited to a maximum bitrate but otherwise preserved as a secondary audio stream. This is because I find that when Plex serves surround sound to my TV with onboard stereo speakers, I am constantly increasing the volume to hear speech and then pulling it down when there's music. I preserve the surround sound in a second stream though because then it's available to me when I'm watching a movie with my surround sound on.

#### Helpful hints

-   The list building processes uses `find` and searches for a number of common video extensions. You can extend this by adding your own.
-   `*.mp4` is excluded from the list of video files I search for because, in my application, that is the extension of the file I'm converting to. This same logic provides the "lock" logic that keeps the same file from being processed twice after a start/stop of the script or the script being run on a shared directory by multiple hosts. **If I have an mp4 source file that needs to be standardized, I change the extension to `m4v` which preserves everything in the source file, including playback, but allows the transcode script to pick it up and work with it.**

## Zoom Up-convert

In these times of COVID-19, a lot of things are happening via ZOOM, including video production. Zoom local and cloud records do a nice job, but are seldom available in 1080p. The `zoom-upconvert` script uses ffmpeg to up-convert the results to 1080p, and it does a pretty decent job! The script is configured to be placed in the root of wherever your zoom record files are stored by default (thinking about both my workflow as well as the default Zoom Recordings directory structure). You can modify this if you like. It will identify all of the video files recursively within the specified path(s). Each video file identified will have a converted version created in a sibling directory named `processed` (which means that if you have your records organized into subdirectories, each subdirectory will get it's own `processed` directory).

### Extras for more fun

These scripts have some inherent logic for resumable processing and load balancing jobs. Each script generates the list of files to be processed and stores them in a `txt` file that with a naming convention that includes the host's `hostname` so that each system will have its own list, even if the work directory is on a NAS or other shared directory.

The scripts also look to see if the destination file exists as part of the file loop. If the destination file exists, the job is skipped. This allows you to have multiple machines processing the same shared directory concurrently, splitting the work up among each of them.

This same logic plus a SIGTERM trap allows you to cancel a running script midway through `(Ctrl + C)` and, the next time you start it up, it starts with the first unprocessed file.

### h264_videotoolbox

The scripts use the h264_videotoolbox codec for transcodes by default as this leverages the Mac GPU (or Intel QuickSync) for improved encoding speeds. You will also see larger file sizes than when you use libx264 so it's up to you if you want to change this.

### Invitation to contribute

I enjoy having my work reviewed and critiqued so my code, and skills, can improve. I wrote (and will continue to write) these scripts efforting to remove all assumptions from my code, but no one is perfect. Reporting issues (or, more helpfully, opening pull requests) will definitely help.
