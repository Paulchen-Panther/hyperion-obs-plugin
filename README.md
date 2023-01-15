# hyperion-obs

[![Latest-Release](https://img.shields.io/github/v/release/hyperion-project/hyperion-obs-plugin)](https://github.com/hyperion-project/hyperion-obs-plugin/releases)
[![GitHub Actions](https://github.com/hyperion-project/hyperion-obs-plugin/workflows/hyperion-obs/badge.svg?branch=main)](https://github.com/hyperion-project/hyperion-obs-plugin/actions)

An [OBS Studio][obs] plugin that provides output capabilities to a [Hyperion.ng][hyperion] Server. \
The idea for this plugin originated from a [Hyperion.ng][hyperion] fork of [Murat Seker][m-seker].

## Usage with hyperion-obs

- Open OBS and select the menu entry `Tools > Hyperion Streaming`.
- Enter the flatbuffer destination IP and select the appropriate port.
- Optionally you can change the `Priority` or the image `Output Decimation` factor.
- Click the `Start` button.

![hyperion-obs](screenshot/hyperion-obs.png)

## Contributing

Contributions are welcome! Feel free to join us! We are looking always for people who wants to participate.<br>
[![Contributors](https://img.shields.io/github/contributors/hyperion-project/hyperion-obs-plugin.svg?label=Contributors)](https://github.com/hyperion-project/hyperion-obs-plugin/graphs/contributors)

For an example, you can participate in the translation.<br>
[![Join Translation](https://img.shields.io/badge/POEditor-translate-green.svg)](https://poeditor.com/join/project?hash=0diZuCpLVX)

## Download

See [Release Page](https://github.com/hyperion-project/hyperion-obs-plugin/releases)

## Build
### Windows & macOS (64-bit)
First follow the build instructions for OBS-Studio on [Windows][obs_build_windows]/[macOS][obs_build_macos].
- Add the following entries before the first configuration:

| Entry name         | Type     | Value (e.g.)            |
|--------------------|----------|-------------------------|
| OBS_SOURCE         | PATH     | /obs-studio             |
| OBS_BUILD          | PATH     | /obs-studio/build       |

- This should cause the plugin DLL file to be created in the desired development environment.

### Linux (64-bit)
- Install OBS Studio, Git, CMake and Qt5 (or Qt6)

```
sudo apt install obs-studio git cmake qtbase5-dev
```

- Build plugin

```
git clone --recursive https://github.com/hyperion-project/hyperion-obs-plugin.git && cd hyperion-obs-plugin
cmake -S . -B build && cmake --build build
sudo cmake --install build
```

## License
The source is released under MIT-License (see https://opensource.org/licenses/MIT).<br>
[![GitHub license](https://img.shields.io/badge/License-MIT-yellow.svg)](https://raw.githubusercontent.com/hyperion-project/hyperion-obs-plugin/main/LICENSE)

[obs]: https://obsproject.com/
[obs_build_windows]: https://github.com/obsproject/obs-studio/wiki/build-instructions-for-windows
[obs_build_macos]: https://github.com/obsproject/obs-studio/wiki/build-instructions-for-mac
[hyperion]: https://github.com/hyperion-project/hyperion.ng
[m-seker]: https://github.com/m-seker
