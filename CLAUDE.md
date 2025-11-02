# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Fcitx5-android is a port of the [Fcitx5](https://github.com/fcitx/fcitx5) input method framework to Android. It's a complex multi-language project combining:
- **Android Kotlin** app layer (UI, services, configuration)
- **C++ native libraries** (Fcitx5 core, input engines)
- **Plugin system** for extensible input methods
- **Gradle build system** with native CMake integration

## Development Commands

### Building
```bash
# Build debug APK
./gradlew assembleDebug

# Build release APK
./gradlew assembleRelease

# Install to connected device
./gradlew installDebug
./gradlew installRelease

# Build specific plugin
./gradlew :plugin:rime:assembleDebug
```

### Testing
```bash
# Run unit tests
./gradlew test

# Run Android instrumentation tests
./gradlew connectedAndroidTest

# Run tests for specific module
./gradlew :lib:fcitx5:test
```

### Native Development
```bash
# Build native libraries only
./gradlew :app:externalNativeBuildDebug

# Clean native build
./gradlew :app:clean
```

### Dependencies and Setup
```bash
# Initialize and update submodules
git submodule update --init --recursive

# Install required system dependencies
# Ubuntu/Debian: sudo apt install extra-cmake-modules gettext
# Arch: sudo pacman -S extra-cmake-modules
# macOS: brew install extra-cmake-modules gettext
```

## Architecture

### Project Structure
```
app/                     # Main Android application
├── src/main/java/       # Kotlin source code
│   └── org/fcitx/fcitx5/android/
│       ├── core/        # Core functionality and IPC
│       ├── input/       # Input method service implementation
│       ├── ui/          # Activities, fragments, settings
│       ├── utils/       # Utilities and helpers
│       └── daemon/      # Background service management
├── src/main/cpp/        # Native C++ code
│   ├── androidfrontend/ # Android-specific frontend
│   ├── androidkeyboard/ # Keyboard integration
│   └── androidnotification/ # Notification system

lib/                     # Shared libraries
├── common/              # Common utilities
├── fcitx5/              # Fcitx5 core library
├── fcitx5-lua/          # Lua scripting support
├── libime/              # Intelligent Input Method library
├── fcitx5-chinese-addons/ # Chinese input methods
└── plugin-base/         # Plugin framework base

plugin/                  # Input method plugins
├── anthy/               # Japanese input
├── chewing/             # Chinese Zhuyin
├── hangul/              # Korean input
├── rime/                # RIME engine
├── thai/                # Thai input
├── unikey/              # Vietnamese input
├── jyutping/            # Cantonese input
├── sayura/              # Sinhala input
└── clipboard-filter/    # Clipboard management

build-logic/             # Gradle build conventions and plugins
codegen/                 # Code generation utilities
```

### Key Components

**Android App Layer (`app/`)**
- `FcitxApplication.kt` - Main application class
- `FcitxRemoteService.kt` - Service interface for Fcitx daemon
- `input/` - InputMethodService implementation
- `ui/` - Settings UI and configuration
- `core/` - IPC and core Android integration

**Native Libraries (`lib/`)**
- `fcitx5/` - Core Fcitx5 engine ported to Android
- `libime/` - Predictive input and language modeling
- `fcitx5-chinese-addons/` - Pinyin, Shuangpin, Wubi etc.

**Plugin System (`plugin/`)**
- Each plugin is a separate Gradle module
- Plugins can be loaded as separate APKs
- Follows Fcitx5's plugin architecture

### Build System

- **Gradle with Kotlin DSL** for Android builds
- **CMake** for native C++ compilation
- **Version management** in `build-logic/convention/src/main/kotlin/Versions.kt`
- **Multi-ABI support** (arm64-v8a, armeabi-v7a, x86, x86_64)
- **Plugin-based architecture** with separate build configurations

## Development Notes

### Native Development
- Native code uses CMake with Android NDK
- Pre-built libraries are in `lib/fcitx5/src/main/cpp/prebuilt/`
- Exclude from Android Studio indexing to improve performance
- Use the specific NDK and CMake versions specified in `Versions.kt`

### Plugin Development
- Each plugin is self-contained with its own build.gradle.kts
- Plugins extend `plugin-base` module
- Use `pluginSchema.xsd` for plugin configuration validation
- Reference existing plugins (e.g., `plugin/rime/`) for structure

### Build Variants
- `debug` version uses debug icons and naming
- `release` version includes ProGuard optimization
- Version codes are calculated based on ABI in `Versions.kt`

### Testing Strategy
- Unit tests for Kotlin components
- Integration tests require Android emulator/device
- Native testing through CMake test framework

## Environment Requirements

- **Android SDK**: Platform & Build-Tools 35
- **Android NDK**: Side by side 25 (or version in Versions.kt)
- **CMake**: 3.22.1+ (or version in Versions.kt)
- **KDE extra-cmake-modules**
- **GNU Gettext** >= 0.20
- **Java**: 11+ (specified in Versions.kt)

## Common Issues

- **Android Studio indexing**: Exclude `lib/fcitx5/src/main/cpp/prebuilt/` directory
- **Gradle variants**: Ensure no conflicting environment variables like `_JAVA_OPTIONS`
- **Submodules**: Always run `git submodule update --init --recursive` after clone
- **Nix environment**: Use provided `shell.nix` for proper Android SDK setup