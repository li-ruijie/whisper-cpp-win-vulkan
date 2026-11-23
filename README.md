# whisper.cpp for Windows with Vulkan

Builds [whisper.cpp](https://github.com/ggml-org/whisper.cpp) for 64-bit Windows with its Vulkan
backend, which upstream does not publish for Windows. Each release tracks an upstream semantic
release and carries its tag, so `v1.9.5` here is built from upstream `v1.9.5`.

## Release assets

┌────────────────────────────────┬──────────────────────────────────────────────────────┐
│ Asset                          │ Contents                                             │
├────────────────────────────────┼──────────────────────────────────────────────────────┤
│ whisper-bin-win-vulkan-x64.zip │ executables, libraries, and licences at the zip root │
│ SHA256SUMS.txt                 │ the SHA-256 digest of the zip                        │
└────────────────────────────────┴──────────────────────────────────────────────────────┘

The zip holds every tool upstream builds for Windows, including `whisper-cli`, `whisper-server`,
and `whisper-stream`, together with the Vulkan backend and all CPU backend variants. ggml loads
the Vulkan backend when a Vulkan device is present and picks the CPU variant that matches the
processor otherwise, so one archive serves machines with and without a GPU.

To verify a download, compare the digest that PowerShell reports against the one in
`SHA256SUMS.txt`.

```powershell
(Get-FileHash -Algorithm SHA256 whisper-bin-win-vulkan-x64.zip).Hash
```

## Requirements

The binaries link the Visual C++ runtime dynamically, as upstream's own Windows builds do, so
the Visual C++ 2015-2022 Redistributable for x64 must be installed. Vulkan acceleration further
needs a GPU driver that provides Vulkan, which current AMD, Intel, and NVIDIA drivers do. The
Vulkan loader is part of that driver and is not bundled.

Models are not included. Download a ggml model from
[ggerganov/whisper.cpp](https://huggingface.co/ggerganov/whisper.cpp) on Hugging Face, then run
the tool against an audio file.

```powershell
whisper-cli -m ggml-base.en.bin -f audio.wav
```

## Building

A GitHub Actions workflow checks upstream daily and builds any semantic release that has no
release here yet. It compiles with MSVC using upstream's Windows x64 configuration plus
`GGML_VULKAN`, with SDL2 and the LunarG Vulkan SDK as the only external inputs. A manual run
takes a `version` override and a `force` flag, the latter rebuilding a release in place.

Before publishing, the workflow checks that ggml-vulkan.dll imports nothing beyond the Vulkan
loader, ggml, and the C++ runtime, and transcribes a sample with the packaged `whisper-cli`.
The build machine has no GPU, so that test covers the CPU path and confirms that the Vulkan
backend does not prevent startup. Vulkan execution itself is not tested.

## Licence

The workflow and the other files in this repository are licensed under AGPL-3.0. The released
binaries are whisper.cpp, which is MIT licensed, and bundle SDL2 under the zlib licence. Both
licence texts ship inside the zip as `LICENSE` and `LICENSE-SDL2.txt`.
