facetimehd-firmware — patched branch
====================================

Fork of [patjak/facetimehd-firmware](https://github.com/patjak/facetimehd-firmware). The default
branch carries upstream `master` plus two pull requests, both open upstream at the time of writing.
`master` here is a plain mirror of upstream.

| PR | what it does |
|---|---|
| [#14](https://github.com/patjak/facetimehd-firmware/pull/14) | extracts the eleven calibration files from the Boot Camp driver as well, not only from the macOS assistant |
| [#15](https://github.com/patjak/facetimehd-firmware/pull/15) | fixes two real defects found by shellcheck, then quiets the rest |

**On #14.** The `AppleCamera.sys` of the Boot Camp package for the 2017 models, Apple product
041-89042, carries all eleven set files, at different offsets from the macOS assistant. The eleven
hashes extracted from it are identical to the ones already in the tree, which came from an unrelated
binary, so each set of offsets now confirms the other. Seven of the eleven had a single source before.

**On #15.** `checkFirmwareHash()` reported a mismatch with `${firm_hash}` while the hash it computed
sits in `fw_hash`, so the message came out without the one value worth quoting in a bug report. The
`--ignore-hashes` path also called `checkFirmwareHexdump` with a stray extra argument. The rest is
quoting, `-gt` instead of `>`, and checked `cd` — verified not to change behaviour by running both
versions over the same inputs and comparing the output byte for byte.

Usage is unchanged from upstream: `make` downloads and extracts, `-x` takes a driver binary, `-s`
takes a set file source, `make install` puts everything in `/lib/firmware/facetimehd/`.

Which firmware you get
----------------------

No firmware or calibration file is in this repository, and none can be: they are Apple's binaries.
The tool fetches them from Apple's servers with ranged requests and verifies every extracted file
against a known hash.

`make` defaults to **5.60.0** (the firmware identifies itself as `S2ISP-01.57.00`), which is
upstream's default too. `make FW_VER=1.43.0` fetches the older one instead.

On a MacBookPro14,1 both work, keep the same formats and sizes, and load the same `1571_01XX.dat`
calibration. They differ in how the image is processed. Measured on a static scene in low light,
three interleaved rounds of 160 frames each, at the same exposure:

| | 1.43.0 | 5.60.0 |
|---|---|---|
| temporal noise | 5.01 | **2.11** (−58 %) |
| detail, noise-free estimate | **6.24** | 4.19 (−33 %) |
| detail / noise | 1.25 | **1.99** (+60 %) |

So 5.60.0 is not simply better: it removes more than half the noise but loses a third of the real
detail. Cleaner and softer against grainier and sharper. Pick whichever you prefer — it is one
command either way, and both files can sit side by side in `/lib/firmware/facetimehd/`.

---

