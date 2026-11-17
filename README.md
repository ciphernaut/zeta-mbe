# zeta-mbe

The speech decoder for P25 phase 1 (IMBE) and DMR (AMBE+2) voice frames, as a small program
that reads frames on stdin and writes speech on stdout.

It was written as the vocoder behind the `p25_decode` and `dmr_decode` rows of
[ZetaSDR](https://github.com/ciphernaut/ZetaSDR), which decodes those protocols itself (calls,
talkgroups, radio identities, encryption flags) and pipes the voice frames to `zeta-mbe` when it
is installed. It works as well from a shell.

This repository holds no codec. `zeta-mbe.c` is the glue between the frames and
[mbelib-neo](https://github.com/arancormonk/mbelib-neo), which `install.sh` downloads from its
authors and links in, on your machine.

## Before you build it

- AMBE and IMBE are patented. This repository ships no implementation of them, and no built
  program: there are no releases and CI never compiles the codec.
- `install.sh` downloads mbelib-neo from its authors, under their licence (GPL-2.0-or-later),
  and builds it on your machine at your request. The program it produces is built for you and is
  not to be redistributed.
- Whether you may build or use it where you live is your responsibility.
- `zeta-mbe` decrypts nothing: it turns voice frames into speech, and that is all.

The script prints the same points and goes on only when you type `yes`.

## Build and install

```bash
git clone https://github.com/ciphernaut/zeta-mbe
cd zeta-mbe
./install.sh
```

It needs `git`, `cmake` and a C compiler (on Debian or Ubuntu: `sudo apt install git cmake
build-essential`), and names the package of any that is missing. Then it:

1. prints the notice above and waits for `yes`;
2. clones mbelib-neo at the one commit the script pins (v2.6.11), and refuses any other;
3. builds the library as a static library with `cmake` (its build also fetches the FFT code it
   compiles in, PFFFT, from the same authors, checked against a pinned checksum), then compiles
   `zeta-mbe.c` against it;
4. tests it: 50 frames of silence in for each codec, 160 samples out for each frame (a program
   that fails is not installed);
5. installs `zeta-mbe` under the prefix, `~/.local/bin/zeta-mbe` by default;
6. prints what it built, from which commit, and where it put it.

The build happens in a temporary directory, which the script removes when it ends.

| Option | Meaning |
|---|---|
| `--yes` | Accept the notice without the prompt. |
| `--src DIR` | Build from an mbelib-neo source tree you already have, and download nothing. The tree needs its bundled PFFFT archive (`third_party/pffft-*.tar.gz`), as mbelib-neo's release source archives have it. |
| `--prefix DIR` | Install `DIR/bin/zeta-mbe`. The default is `$HOME/.local`. |
| `--help` | Print the options and exit. |

## Use

`zeta-mbe <codec>` reads frames on its standard input and writes speech on its standard output:

| Codec | Protocol | Bytes in per frame | Out per frame |
|---|---|---|---|
| `imbe7200x4400` | P25 phase 1 | 184 (8 rows of 23) | 160 samples, s16le, 8 kHz |
| `ambe3600x2450` | DMR | 96 (4 rows of 24) | 160 samples, s16le, 8 kHz |

A frame is the codec's channel-coded frame after the protocol's own deinterleaving, row by row.
Each input byte is one bit of it: 0 is a certain zero, 255 a certain one, and the values between
are the demodulator's confidence, which the library's soft-decision decoding uses. Each frame's
speech is written as soon as it is decoded. `zeta-mbe --version` prints its own version and the
library's; the end of the input ends it cleanly (status 0), and a bad argument prints the usage
(status 2).

```bash
zeta-mbe ambe3600x2450 < frames.bin | aplay -f S16_LE -r 8000 -c 1
```

## With ZetaSDR

The node looks for `zeta-mbe` when each call starts, so one installed while a job runs is used
from the next call. It takes the program from `ZETA_HANDLER_ZETA_MBE` when that names it, and
otherwise from the node's `PATH`. `~/.local/bin` is on many shells' `PATH` but not on a
service's: a node running as a systemd service needs `Environment=ZETA_HANDLER_ZETA_MBE=...` in
its unit, or the program somewhere on the unit's `PATH`. Check with
`zeta modes show p25_decode`.

## Licence

GPL-2.0-or-later, the licence of mbelib-neo, which every built `zeta-mbe` links: see
[LICENSE](LICENSE).
