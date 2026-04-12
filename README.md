[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

# b64audio

audio editor but base64
its just an html file

## what it does

you drop in any audio file (mp3, wav, ogg, flac whatever your browser can decode) and it:
1. decodes it using the web audio api
2. encodes it as a 16-bit pcm wav
3. converts that to a base64 string
4. stores that string as the single source of truth

every edit is the same loop: `atob()` the string into bytes > do stuff to the bytes > `btoa()` it back

## operations

- **trim** - pick a start/end percentage, slice the pcm data, keep the wav header intact
- **volume** - multiply every 16-bit sample by a factor (0.5 = half, 2.0 = double), clamped to int16 range
- **fade in** - ramp samples from 0 to 1 over the first n% of the audio
- **fade out** - same but 1 to 0 at the end

stereo works fine, each channel gets processed per frame

## other stuff

- **undo** - pushes the previous b64 string onto a stack before each change. yeah the stack is just a bunch of huge strings in memory
- **waveform** - canvas, reads the actual int16 samples, redraws after edits
- **b64 ticker** - shows a rolling chunk of the b64 string in the header during playback, it looks cool
- **log** - logs
- **export** - download as wav or as raw base64 text

## how to run

open audio-editor.html in your browser 🤯

## why wav

if you try to multiply raw mp3 bytes by a volume factor you just destroy the frame headers and get noise. so on load everything gets decoded to pcm and re-encoded as wav, which is just a header + raw samples so you can actually edit the bytes.

downside is a 3mb mp3 turns into like a 30mb wav and the b64 string is even bigger than that but whateverrr

## limit

- undo stack uses a lot of memory each entry is a full copy of the b64
- big files are slow atob/btoa arent great with multi-megabyte strings
- trim is percentage based not timestamp based
- no copy paste no multitrack
- the b64 ticker position is estimated

# license

under [MIT license](LICENSE)



can someone genuinely help on this 🥹
