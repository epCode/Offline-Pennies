# Personal Finance Tracker - Build & Release Guide

This guide covers building and releasing the Personal Finance Tracker across all platforms: Linux, Windows, macOS, Android, and iOS.

## Table of Contents

- [Prerequisites](#prerequisites)
- [Local Development](#local-development)
- [Desktop Builds](#desktop-builds)
- [Mobile Builds](#mobile-builds)
- [Automated GitHub Actions Releases](#automated-github-actions-releases)
- [Troubleshooting](#troubleshooting)

## Prerequisites

### Global Requirements

- **Node.js** 22+ ([download](https://nodejs.org/))
- **pnpm** 10+ (`npm install -g pnpm`)
- **Git** ([download](https://git-scm.com/))

### Platform-Specific Requirements

#### Linux
```bash
# Ubuntu/Debian
sudo apt-get install libgtk-3-dev libwebkit2gtk-4.0-dev libappindicator3-dev librsvg2-dev patchelf

# Fedora
sudo dnf install gtk3-devel webkit2gtk3-devel libappindicator-gtk3-devel librsvg2-devel
```

#### Windows
- **Visual Studio Build Tools** or **Visual Studio Community** with C++ support
- **Rust** ([install via rustup](https://rustup.rs/))

#### macOS
- **Xcode Command Line Tools**: `xcode-select --install`
- **Rust**: `curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh`

#### Android
- **Java 17+**: `actions/setup-java@v4`
- **Android SDK**: `android-actions/setup-android@v3`
- **EAS CLI**: `npm install -g eas-cli`

#### iOS
- **Xcode 14+** (macOS only)
- **CocoaPods**: `sudo gem install cocoapods`
- **EAS CLI**: `npm install -g eas-cli`

## Local Development

### Setup

```bash
# Clone repository
git clone https://github.com/yourusername/personal-finance-tracker.git
cd personal-finance-tracker

# Install dependencies
pnpm install

# Start development server (web)
pnpm dev
```

### Web Development

```bash
# Development server
pnpm dev

# Build for production
pnpm build

# Preview production build
pnpm preview
```

## Desktop Builds

### Linux

```bash
# Build for Linux (AppImage + .deb)
pnpm tauri:build -- --target x86_64-unknown-linux-gnu

# Output locations:
# - AppImage: src-tauri/target/x86_64-unknown-linux-gnu/release/bundle/appimage/
# - Debian: src-tauri/target/x86_64-unknown-linux-gnu/release/bundle/deb/
```

### Windows

```bash
# Build for Windows (MSI + Portable)
pnpm tauri:build -- --target x86_64-pc-windows-msvc

# Output locations:
# - MSI: src-tauri/target/x86_64-pc-windows-msvc/release/bundle/msi/
# - Portable EXE: src-tauri/target/x86_64-pc-windows-msvc/release/
```

### macOS

#### Intel (x86_64)

```bash
# Build for Intel Macs
pnpm tauri:build -- --target x86_64-apple-darwin

# Create DMG
cd src-tauri/target/x86_64-apple-darwin/release/bundle/macos
hdiutil create -volname "Personal Finance Tracker" -srcfolder . -ov -format UDZO personal-finance-tracker-macos-intel.dmg
```

#### Apple Silicon (ARM64)

```bash
# Build for Apple Silicon Macs
pnpm tauri:build -- --target aarch64-apple-darwin

# Create DMG
cd src-tauri/target/aarch64-apple-darwin/release/bundle/macos
hdiutil create -volname "Personal Finance Tracker" -srcfolder . -ov -format UDZO personal-finance-tracker-macos-arm64.dmg
```

#### Universal Binary (Intel + ARM64)

```bash
# Build both targets
pnpm tauri:build -- --target x86_64-apple-darwin
pnpm tauri:build -- --target aarch64-apple-darwin

# Create universal binary
lipo -create \
  src-tauri/target/x86_64-apple-darwin/release/personal-finance-tracker \
  src-tauri/target/aarch64-apple-darwin/release/personal-finance-tracker \
  -output personal-finance-tracker-universal
```

## Mobile Builds

### Android APK

#### Local Build

```bash
# Install EAS CLI
npm install -g eas-cli

# Create React Native Expo project
mkdir -p mobile
cd mobile
npx create-expo-app personal-finance-tracker

# Build APK
cd personal-finance-tracker
npm install
npx eas build --platform android --local

# Output: personal-finance-tracker.apk
```

#### Using EAS Cloud

```bash
# Configure EAS
npx eas init

# Build APK
npx eas build --platform android

# Output: Downloaded to current directory
```

### iOS App

#### Local Build (macOS only)

```bash
# Install EAS CLI
npm install -g eas-cli

# Create React Native Expo project
mkdir -p mobile
cd mobile
npx create-expo-app personal-finance-tracker

# Build iOS App
cd personal-finance-tracker
npm install
npx eas build --platform ios --local

# Output: personal-finance-tracker.ipa
```

#### Using EAS Cloud

```bash
# Configure EAS
npx eas init

# Build iOS App
npx eas build --platform ios

# Output: Downloaded to current directory
```

## Automated GitHub Actions Releases

### Setup

1. **Push to GitHub**
   ```bash
   git remote add origin https://github.com/yourusername/personal-finance-tracker.git
   git push -u origin main
   ```

2. **Create Release Tag**
   ```bash
   git tag -a v1.0.0 -m "Release version 1.0.0"
   git push origin v1.0.0
   ```

### Workflow Triggers

The `build-release.yml` workflow automatically triggers when:
- A tag matching `v*.*.*` is pushed (e.g., `v1.0.0`, `v1.2.3`)
- Manually triggered via GitHub Actions UI

### Release Process

1. **Create Release** - GitHub Release is created automatically
2. **Build Desktop Apps** - Parallel builds for Linux, Windows, macOS (Intel & ARM64)
3. **Build Mobile Apps** - Android APK and iOS IPA builds
4. **Upload Artifacts** - All builds automatically uploaded to GitHub Release

### Workflow Structure

```
create-release
├── build-linux
├── build-windows
├── build-macos-intel
├── build-macos-arm64
├── build-android
└── build-ios
```

All desktop and mobile builds run in parallel for faster release times.

### GitHub Secrets (Optional)

For advanced features, add these secrets to your GitHub repository:

- `APPLE_ID` - Apple ID for code signing
- `APPLE_PASSWORD` - Apple ID app-specific password
- `APPLE_TEAM_ID` - Apple Developer Team ID
- `ANDROID_KEYSTORE_BASE64` - Base64-encoded Android keystore

## Troubleshooting

### Linux Build Issues

**Error: "libgtk-3-dev not found"**
```bash
sudo apt-get update
sudo apt-get install libgtk-3-dev libwebkit2gtk-4.0-dev
```

**Error: "patchelf not found"**
```bash
sudo apt-get install patchelf
```

### Windows Build Issues

**Error: "MSVC not found"**
- Install Visual Studio Build Tools with C++ support
- Or install Visual Studio Community with C++ workload

**Error: "Rust toolchain not found"**
```bash
rustup target add x86_64-pc-windows-msvc
```

### macOS Build Issues

**Error: "Xcode Command Line Tools not found"**
```bash
xcode-select --install
```

**Error: "Code signing failed"**
```bash
# Check signing identity
security find-identity -v -p codesigning

# Or disable signing for development
export CODESIGN_ALLOCATE=$(xcrun -find codesign_allocate)
```

### Android Build Issues

**Error: "Java not found"**
```bash
# Install Java 17
sudo apt-get install openjdk-17-jdk
```

**Error: "Android SDK not found"**
```bash
# Use EAS cloud build instead
npx eas build --platform android
```

### iOS Build Issues

**Error: "CocoaPods not found"**
```bash
sudo gem install cocoapods
pod setup
```

**Error: "iOS SDK not found"**
- Ensure Xcode is installed: `xcode-select --install`
- Update Xcode: `sudo xcode-select --switch /Applications/Xcode.app/Contents/Developer`

## Build Artifacts

After building, artifacts are located at:

### Desktop
- **Linux**: `src-tauri/target/x86_64-unknown-linux-gnu/release/bundle/`
- **Windows**: `src-tauri/target/x86_64-pc-windows-msvc/release/bundle/`
- **macOS**: `src-tauri/target/[target]/release/bundle/macos/`

### Mobile
- **Android**: `mobile/personal-finance-tracker/dist/` or EAS output
- **iOS**: `mobile/personal-finance-tracker/dist/` or EAS output

## Release Checklist

- [ ] Update version in `package.json`
- [ ] Update version in `src-tauri/tauri.conf.json`
- [ ] Update `CHANGELOG.md`
- [ ] Commit changes: `git commit -am "Bump version to v1.0.0"`
- [ ] Create tag: `git tag -a v1.0.0 -m "Release v1.0.0"`
- [ ] Push: `git push origin main && git push origin v1.0.0`
- [ ] GitHub Actions workflow runs automatically
- [ ] Verify all builds completed successfully
- [ ] Download and test each platform's build
- [ ] Publish GitHub Release

## Signing & Notarization

### macOS Code Signing

For distribution outside the App Store:

```bash
# Create signing certificate
security create-keychain -p password build.keychain
security unlock-keychain -p password build.keychain

# Import certificate
security import certificate.p12 -k build.keychain -P password

# Sign app
codesign -s "Developer ID Application" --deep --force \
  src-tauri/target/x86_64-apple-darwin/release/bundle/macos/Personal\ Finance\ Tracker.app
```

### Windows Code Signing

For distribution:

```bash
# Sign EXE
signtool sign /f certificate.pfx /p password /t http://timestamp.server \
  src-tauri/target/x86_64-pc-windows-msvc/release/personal-finance-tracker.exe
```

### Android Signing

```bash
# Create keystore
keytool -genkey -v -keystore release.keystore -keyalg RSA -keysize 2048 -validity 10000 -alias release

# Sign APK
jarsigner -verbose -sigalg SHA1withRSA -digestalg SHA1 \
  -keystore release.keystore personal-finance-tracker.apk release
```

## Support

For issues or questions:
- GitHub Issues: [Create an issue](https://github.com/yourusername/personal-finance-tracker/issues)
- Documentation: See `README.md`
- Tauri Docs: [https://tauri.app/](https://tauri.app/)

## License

MIT License - See LICENSE file for details
