# FFMPEG API

A web service for converting audio/video/image files using FFMPEG.

Based on:

* https://github.com/samisalkosuo/ffmpeg-api
* https://github.com/surebert/docker-ffmpeg-service
* https://github.com/jrottenberg/ffmpeg 
* https://github.com/fluent-ffmpeg/node-fluent-ffmpeg

FFMPEG API is provided as Docker image for easy consumption. [See the use cases and troubleshooting (catswords-oss.rdbl.io)](https://catswords-oss.rdbl.io/1274143468/3500729784).

## Endpoints
* `GET /` - API Readme.
* `GET /endpoints` - Service endpoints as JSON.
* `POST /convert/audio/to/mp3` - Convert audio file in request body to mp3. Returns mp3-file.
* `POST /convert/audio/to/wav` - Convert audio file in request body to wav. Returns wav-file.
* `POST /convert/video/to/mp4` - Convert video file in request body to mp4. Returns mp4-file.
* `POST /convert/image/to/jpg` - Convert image file to jpg. Returns jpg-file.
* `POST /video/extract/audio` - Extract audio track from POSTed video file. Returns audio track as 1-channel wav-file.
  * Query param: `mono=no` - Returns audio track, all channels.
* `POST /video/extract/images` - Extract images from POSTed video file as PNG. Default FPS is 1. Returns JSON that includes download links to extracted images.
  * Query param: `compress=zip|gzip` - Returns extracted images as _zip_ or _tar.gz_ (gzip).
  * Query param: `fps=2` - Extract images using specified FPS.
  * Query param: `download=yes` - Downloads extracted image file and deletes it from server, immediately.
* `GET /video/extract/download/:filename` - Downloads extracted image file and deletes it from server.
  * Query param: `delete=no` - does not delete file.
* `POST /probe` - Probe media file, return JSON metadata.

## Docker image

### Build your own
* Clone this repository.
* Build Docker image:
  * `docker build -t ffmpeg-api .`
* Run image in foreground:
  * `docker run -it --rm --name ffmpeg-api -p 3000:3000 ffmpeg-api`
* Run image in background:
  * `docker run -d -name ffmpeg-api -p 3000:3000 ffmpeg-api`

### Use existing
* Run image in foreground:
  * `docker run -it --rm --name ffmpeg-api -p 3000:3000 gnh1201/ffmpeg-api`
* Run image in background:
  * `docker run -d --name ffmpeg-api -p 3000:3000 gnh1201/ffmpeg-api`

### Environment variables
* Default log level is _info_. Set log level using environment variable, _LOG_LEVEL_.
  * Set log level to debug:
  * `docker run -it --rm -p 3000:3000 -e LOG_LEVEL=debug gnh1201/ffmpeg-api`
* Default maximum file size of uploaded files is 512MB. Use environment variable _FILE_SIZE_LIMIT_BYTES_ to change it:
  * Set max file size to 1MB:
  * `docker run -it --rm -p 3000:3000 -e FILE_SIZE_LIMIT_BYTES=1048576 gnh1201/ffmpeg-api`
* All uploaded and converted files are deleted when they've been downloaded. Use environment variable _KEEP_ALL_FILES_ to keep all files inside the container /tmp-directory:
  * `docker run -it --rm -p 3000:3000 -e KEEP_ALL_FILES=true gnh1201/ffmpeg-api`
* When running on Docker/Kubernetes, port binding can be different than default 3000. Use _EXTERNAL_PORT_ to set up external port in returned URLs in extracted images JSON:
  * `docker run -it --rm -p 3001:3000 -e EXTERNAL_PORT=3001 gnh1201/ffmpeg-api`

## Usage
Input file to FFMPEG API can be anything that ffmpeg supports. See [ffmpeg docs for supported formats (www.ffmpeg.org)](https://www.ffmpeg.org/general.html#Supported-File-Formats_002c-Codecs-or-Features).

### Convert
Convert audio/video/image files using the API.

* `curl -F "file=@input.wav" 127.0.0.1:3000/convert/audio/to/mp3  > output.mp3`
* `curl -F "file=@input.m4a" 127.0.0.1:3000/convert/audio/to/wav  > output.wav`
* `curl -F "file=@input.mov" 127.0.0.1:3000/convert/video/to/mp4  > output.mp4`
* `curl -F "file=@input.mp4" 127.0.0.1:3000/convert/videp/to/mp4  > output.mp4`
* `curl -F "file=@input.tiff" 127.0.0.1:3000/convert/image/to/jpg  > output.jpg`
* `curl -F "file=@input.png" 127.0.0.1:3000/convert/image/to/jpg  > output.jpg`

### Extract images
Extract images from video using the API.

* `curl -F "file=@input.mov" 127.0.0.1:3000/video/extract/images`
  * Returns JSON that lists image download URLs for each extracted image.
  * Default FPS is 1.
  * Images are in PNG-format.
  * See sample: [extracted_images.json](samples/extracted_images.json).
* `curl 127.0.0.1:3000/video/extract/download/ba0f565c-0001.png`
  * Downloads exracted image and deletes it from server.
* `curl 127.0.0.1:3000/video/extract/download/ba0f565c-0001.png?delete=no`
  * Downloads exracted image but does not deletes it from server.
* `curl -F "file=@input.mov" 127.0.0.1:3000/video/extract/images?compress=zip > images.zip`
  * Returns ZIP package of all extracted images.
* `curl -F "file=@input.mov" 127.0.0.1:3000/video/extract/images?compress=gzip > images.tar.gz`
  * Returns GZIP (tar.gz) package of all extracted images.
* `curl -F "file=@input.mov" 127.0.0.1:3000/video/extract/images?fps=0.5`
  * Sets FPS to extract images. FPS=0.5 is every two seconds, FPS=4 is four images per seconds, etc.

### Extract audio
Extract audio track from video using the API.

* `curl -F "file=@input.mov" 127.0.0.1:3000/video/extract/audio`
 * Returns 1-channel WAV-file of video's audio track.
* `curl -F "file=@input.mov" 127.0.0.1:3000/video/extract/audio?mono=no`
 * Returns WAV-file of video's audio track, with all the channels as in input video.

### Probe
Probe audio/video/image files using the API.

* `curl -F "file=@input.mov" 127.0.0.1:3000/probe`
  * Returns JSON metadata of media file.
  * The same JSON metadata as in ffprobe command: `ffprobe -of json -show_streams -show_format input.mov`.
  * See sample of MOV-video metadata: [probe_metadata.json](samples/probe_metadata.json).

## Background
Originally developed by [Paul Visco @surebert (github.com)](https://github.com/surebert).

Changes include new functionality, updated Node.js version, Docker image based on Alpine, logging and other major refactoring.

## Use case

### Creating a container

```bash
cd /usr/local/src
git clone https://github.com/gnh1201/ffmpeg-api
cd ffmpeg-api
docker build -t gnh1201/ffmpeg-api .
sudo mkdir /var/cache/ffmpeg-api
sudo chmod -R 777 /var/cache/ffmpeg-api
sudo docker run -itd --rm -p 127.0.0.1:3000:3000 -v /var/cache/ffmpeg-api:/tmp -e KEEP_ALL_FILES=false -e LOG_LEVEL=debug gnh1201/ffmpeg-api # OR `-p 0.0.0.0:3000:3000`
```

## Report abuse
* [GitHub Security Advisories (gnh1201/ffmpeg-api)](https://github.com/gnh1201/ffmpeg-api/security/advisories)
* abuse@catswords.net

## Join the community
* ActivityPub [@gnh1201@catswords.social](https://catswords.social/@gnh1201)
* XMPP [catswords@conference.omemo.id](xmpp:catswords@conference.omemo.id?join)
* [Join Catswords OSS on Microsoft Teams (teams.live.com)](https://teams.live.com/l/community/FEACHncAhq8ldnojAI)
* [Join Catswords OSS #ffmpeg-api on Discord (discord.gg)](https://discord.gg/uEwVWAyBRT)
