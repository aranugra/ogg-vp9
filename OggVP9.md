
# VP9 inside Ogg mapping

This specification assumes a working knowledge of the 
[Ogg bitstream][3] document.

## 1. Header

### 1.1 Identification Header

Identification header is the only packet on the first logical Ogg page and marked
"beginning of stream" in the page flags.


| Name   | Size (Bytes)  | Description
|--------|---------------|--------------
| HDRID  | 1             | Header identifier `0x4F`
| ID     | 4             | VP9 Identifier `VP90` or `{0x56, 0x50, 0x39, 0x30}`
| HDRTYPE| 1             | Stream info header type `0x01`
| VMAJ   | 1             | Mapping major version, break compatibility. This specification define major version `1`
| VMIN   | 1             | Mapping minor version, backward compatible. This specification define minor version `0`
| FW     | 2             | Frame width
| FH     | 2             | Frame height
| PARN   | 3             | Pixel aspect ratio numerator
| PARD   | 3             | Pixel aspect ratio denominator
| FPSN   | 4             | Frame rate numerator
| FPSD   | 4             | Frame rate denominator

Unless otherwise noted, integers are unsigned and encoded in big endian byte order.
Newer minor versions of this specification might append new fields to this structure.

### 1.2. Comment Header

A single comment header, if available, beginning on the second page
of the logical stream and may span one or more pages.

| Name   | Size (Bytes)  | Description
|--------|---------------|--------------
| HDRID  | 1             | Header identifier `0x4F`
| ID     | 4             | VP9 Identifier `VP90` or `{0x56, 0x50, 0x39, 0x30}`
| HDRTYPE| 1             | Comment header type `0x02`
| SPACE  | 1             | `0x20`
|VORBISCOMMENT | \*      | Vorbis comments

The structure of VORBISCOMMENT metadata is defined in the Ogg Vorbis I [format specification][1].

### 1.3. Additional Headers

Newer minor versions of this specification may define new headers that will
start with the `HDRID` and `ID`, implemetations should ignore `HDRTYPE` they
not recognize. Any possible `HDRTYPE` values are reserved and might be defined
in later versions of this specification.

### 1.4 Granule Position

The granule position of these first pages containing only headers is zero.

## 2. Video Frames

The first video frame packet must begin on a new page and must be a keyframe. 
The last page is marked "end of stream" in the page flags.

### 2.1. Pages & Packets

An Ogg page may contain one or more Ogg packets.

    --------------------------------------  -----------------------------
    |                page                |  |            page           |
    --------------------------------------  -----------------------------
    |  packet | packet | packet | packet |  |  packet | packet | packet |
    --------------------------------------  -----------------------------

A packet may span multiple pages if necessary.

    ----------------------------------  ----------------------------------
    |              page              |  |              page              |
    ----------------------------------  ----------------------------------
    |           very la..            |  |           ..rge packet         |
    ----------------------------------  ----------------------------------

One Ogg packet must contain exactly one VP9 frame with no additional data.

    ----------------------------------  --------------------------
    |              page              |  |          page          |
    ----------------------------------  --------------------------
    |  frame | frame | frame | frame |  |  frame | frame | frame |
    ----------------------------------  --------------------------

VP9 keyframe packets should start a new Ogg page and should be the only packet in a page.

    ----------------  --------------------------------  ----------------
    |     page     |  |              page            |  |     page     |
    ----------------  --------------------------------  ----------------
    |   keyframe   |  |  p-frame | p-frame | p-frame |  |   keyframe   |
    ----------------  --------------------------------  ----------------

### 2.2. Granule position

The granule position of a page represents the end
position of the last packet completed on that page.

The granule position for VP9 stream contains following fields:

    MSB                                         LSB
    -----------------------------------------------
    | pts            | inv_count | dist    | resv |
    -----------------------------------------------
    | 63          32 | 31     30 | 29    3 | 2  0 |

- pts: End of presentation timestamp as frame count of the last complete packet in the Ogg page.
- inv_count: Number of invisible VP9 frame since the last visible frame. For visible frame all bits are set.
- dist: Distance, in packets to the last key frame.
- resv: Reserved for future use.

> This specification define VP9 `show_existing_frame` packet as a non keyframe and is visible.


## 3. References

- Ogg [bitstream format][2] specification
- Vorbis [comment][1] specification

[1]: https://xiph.org/vorbis/doc/v-comment.html
[2]: https://xiph.org/ogg/doc/rfc3533.txt
[3]: https://xiph.org/ogg/doc/oggstream.html
