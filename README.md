HLS-Stream-Creator
==================

Introduction
-------------

HLS-Stream-Creator is a simple BASH Script designed to take a media file, segment it and create an M3U8 playlist for serving using HLS.
There are numerous tools out there which are far better suited to the task, and offer many more options. This project only exists because I was asked to look
into HTTP Live Streaming in depth, so after reading the [IETF Draft](http://tools.ietf.org/html/draft-pantos-http-live-streaming-11 "HLS on IETF") I figured I'd start with the basics by creating a script to encode arbitrary video into a VOD style HLS feed.



Usage
------

Usage is incredibly simple 

```
./HLS-Stream-Creator.sh -[lf] [-c segmentcount] -i [inputfile] -s [segmentlength(seconds)] -o [outputdir] -b [bitrates]


Deprecated Legacy usage:
	HLS-Stream-Creator.sh inputfile segmentlength(seconds) [outputdir='./output']

```

So to split a video file called *example.avi* into segments of 10 seconds, we'd run

```
./HLS-Stream-Creator.sh -i example.avi -s 10
```

**Arguments**

```
    Mandatory Arguments:

	-i [file]	Input file
	-s [s]		Segment length (seconds)

    Optional Arguments:

	-o [directory]	Output directory (default: ./output)
	-c [count]	Number of segments to include in playlist (live streams only) - 0 is no limit
	-b [bitrates]	Output video Bitrates in kb/s (comma seperated list for adaptive streams)
	-p [name]	Playlist filename prefix
	-t [name]	Segment filename prefix
	-l		Input is a live stream
	-f		Foreground encoding only (adaptive non-live streams only)
	-S		Name of a subdirectory to put segments into
	-2		Use two-pass encoding
	-q [quality]	Change encoding to CFR with [quality]
	-C		Use constant bitrate as opposed to variable bitrate
```


Adaptive Streams
------------------

As of [HLS-6](http://projects.bentasker.co.uk/jira_projects/browse/HLS-6.html) the script can now generate adaptive streams with a top-level variant playlist for both VoD and Linear input streams.

In order to create seperate bitrate streams, pass a comma seperated list in with the *-b* option

```
./HLS-Stream-Creator.sh -i example.avi -s 10 -b 28,64,128,256
```

By default, transcoding for each bitrate will be forked into the background - if you wish to process the bitrates sequentially, pass the *-f* option

```
./HLS-Stream-Creator.sh -i example.avi -s 10 -b 28,64,128,256 -f
```

In either case, in accordance with the HLS spec, the audio bitrate will remain unchanged


Output
-------

As of version 1, the HLS resources will be output to the directory *output*. These will consist of video segments encoded in H.264 with AAC audio and an m3u8 file in the format

>\#EXTM3U  
>\#EXT-X-MEDIA-SEQUENCE:0  
>\#EXT-X-VERSION:3  
>\#EXT-X-TARGETDURATION:10  
>\#EXTINF:10, no desc  
>example_00001.ts  
>\#EXTINF:10, no desc  
>example_00002.ts  
>\#EXTINF:10, no desc  
>example_00003.ts  
>\#EXTINF:5, no desc  
>example_00004.ts  
>\#EXT-X-ENDLIST



Using a Specific FFMPEG binary
-------------------------------

There may be occasions where you don't want to use the *ffmpeg* that appears in PATH. At the top of the script, the ffmpeg command is defined, so change this to suit your needs

```
FFMPEG='/path/to/different/ffmpeg'
```


H265 details
------------

Check has been added for libx265 to enforce bitrate limits for H265 since it uses additional parameters.


Additional Environment Variables
-------------------------------

There are few environment variables which can control the ffmpeg behaviour.

* `VIDEO_CODEC` - The encoder which will be used by ffmpeg for video streams. Examples: _libx264_, _nvenc_
* `AUDIO_CODEC` - Encoder for the audio streams. Examples: _aac_, _libfdk_acc_, _mp3_, _libfaac_
* `NUMTHREADS` - A number which will be passed to the `-threads` argument of ffmpeg. Newer ffmpegs with modern libx264 encoders will use the optimal number of threads by default.
* `FFMPEG_FLAGS` - Additional flags for ffmpeg. They will be passed without any modification.

Example usage:

```
export VIDEO_CODEC="nvenc"
export FFMPEG_FLAGS="-pix_fmt yuv420p -profile:v"
./HLS-Stream-Creator.sh example.avi 10
```

License
--------

HLS-Stream-Creator is licensed under the [BSD 3 Clause License](http://opensource.org/licenses/BSD-3-Clause) and is Copyright (C) 2013 [Ben Tasker](http://www.bentasker.co.uk)


Issue Tracking
----------------

Although the Github issue tracker can be used, the bulk of project management (such as it is) happens in JIRA. See [projects.bentasker.co.uk](http://projects.bentasker.co.uk/jira_projects/browse/HLS.html) for a HTML mirror of the tracking.
