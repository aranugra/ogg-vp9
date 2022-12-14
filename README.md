
# VP9 inside Ogg

Mapping of WebM codec inside Ogg container has been specified since
the [introduction of the WebM][1] <sup>[archived][9]</sup> that refer to a document describing
 _"Ogg container format mapping for WebM"_, and implementations
of VP8 inside Ogg is already implemented by [ffmpeg][3],
development version of [liboogz][4] and some other softwares.

Some time later google release VP9, but no software that support VP9 inside Ogg yet.
Firefox [documentation claim that][5] <sup>[archived][8]</sup> they can play VP8 and VP9 inside Ogg.
Contrary to their documentation though, Firefox in fact can *not* play Ogg with VP8 codec
(tested on Firefox version 104.0.1). On related note, Google Chrome and MS Edge do
support VP8 inside Ogg. I can not test whether they can play VP9 inside 
Ogg because I can not even able to find any software that can put VP9 inside Ogg.

Because lack of OggVP9 implementation, for learning purpose, I am interested to implement it.


## 1. OggVP9 Mapping

Here is what I found while searching for the elusive document describing  _"Ogg container format mapping for WebM"_

- The mapping document on the [aftermentioned article][1] <sup>[archived][9]</sup> is now inaccessible
- The [Xiph Wiki][6] also have no mention of it, which is understandable since they are not Xiph codecs
- A bit searching I found this [PDF document][2] which I believe is the same document linked on the article

From there I am implementing OggVP9 mostly using the same mapping as OggVP8 with VP9 specific changes

- The `ID` field is (obviously) set to `VP90`
- VP9 pts always increasing regardless visible or invisible frame, this also true for VP9 inside IVF format

More detail is written in [this document](OggVP9.md).

> However it should be noted that it is not offical OggVP9 mapping or anything.
> What I am doing is just taking existing OggVP8 specification, experimenting
> what is work for VP9 and documenting it.


## 2. Building code

My implementation is just a small patch for FFmpeg. All patches licensed under LGPL v2.1 or
later, or the same license as FFmpeg this patch applied to.
However the code is _experimental_ and might contains serious bugs. Use it at your own risk.

### 2.1. Dependencies

To build from source you should install at least gcc, yasm, and development version of these libraries

- libvpx
- libopus
- libvorbis
- libtheora
- sdl2


### 2.2. Patch FFmpeg

This patch should work on FFmpeg 5.1, and may also work with more recent version of FFmpeg.

    $ git clone https://github.com/aranugra/ogg-vp9.git
    $ wget http://ffmpeg.org/releases/ffmpeg-5.1.tar.xz
    $ tar -xf ffmpeg-5.1.tar.xz
    $ patch -p1 -d ffmpeg-5.1 < ogg-vp9/ffmpeg/vp9-inside-ogg.patch

### 2.3. Configure, build and install

Configure and build FFmpeg

    $ OGGVP9_PREFIX=/opt/ffmpeg-5.1-ogg-vp9
    $ cd ffmpeg-5.1
    $ ./configure --prefix=$OGGVP9_PREFIX --enable-libvpx --enable-libopus --enable-libtheora --enable-libvorbis --enable-shared
    $ make -j3
    $ make install

Then add `$OGGVP9_PREFIX/bin` to path if necessary.

### 2.4. Binary Packages

Compiled packages for Win64 are available on the [release page](https://github.com/aranugra/ogg-vp9/releases).

## 3. Encoding & Playing OggVP9

### 3.1. Encoding

Encoding with constant rate factor

    $ ffmpeg -i input.mp4 -f ogg -c:v libvpx-vp9 -crf 30 -b:v 0 -c:a libopus -b:a 160k output.ogv

Generally you can refer to [VP9 encoding guide][7] and just replace
`-f webm` to `-f ogg` and replace output extension to `.ogv`.

### 3.2. Sample files

Here is some OggVP9 video [samples](https://drive.google.com/drive/folders/1z0MrMkNkHYjzfg0p0hoP6x2ndLs8iUiX?usp=sharing).

### 3.3. Playing

To play OggVP9 video, you can use `ffplay`

    $ ffplay output.ogv

For a real video player, you can build [mpv][10] linked against this version of FFmpeg.
In turn you can use [SMPlayer][11] as mpv front-end.


## 4 Limitation

Known limitation on current implementation

- Does not support any features of VP9 that only supported by matroska container (alpha, WebTTV, etc, etc)
- No VP9 codec private information


[1]: https://blogs.gnome.org/uraeus/2010/05/19/webm-and-gstreamer/
[2]: https://people.freedesktop.org/~slomo/ogg-vp8/ogg-vp8.pdf
[3]: https://ffmpeg.org
[4]: https://gitlab.xiph.org/xiph/liboggz
[5]: https://developer.mozilla.org/en-US/docs/Web/Media/Formats/Containers
[6]: https://wiki.xiph.org/Ogg
[7]: https://trac.ffmpeg.org/wiki/Encode/VP9
[8]: http://web.archive.org/web/20220830180037/https://developer.mozilla.org/en-US/docs/Web/Media/Formats/Containers
[9]: http://web.archive.org/web/20211026055813/https://blogs.gnome.org/uraeus/2010/05/19/webm-and-gstreamer/
[10]: https://github.com/mpv-player/mpv
[11]: https://github.com/smplayer-dev/smplayer
