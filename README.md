# Yamm

An independent browser with personality. Yamm combines native macOS browsing with customizable themes, profiles, page tools, translation, music, and an integrated AI console.

**Current Mac release: 0.0.7 · macOS 15.5+ · Apple silicon and Intel.** The Windows implementation is an experimental Electron port; it is not feature-complete or a supported release.

## Download

Get the Mac installer from [GitHub Releases](https://github.com/sobalaan/Yamm/releases). Open the DMG and drag **Yamm.app** into **Applications**.

The current development installer is ad-hoc signed, not Developer ID signed or Apple-notarized. macOS may prevent it from opening. A public production release should use Developer ID signing and notarization.

## Features

- Native tabs, address/search suggestions, bookmarks, history, downloads, and session restoration.
- Profile-specific settings and browsing data, plus partial and full private modes.
- Themeable SwiftUI chrome, including Royal Purple and Bling Bling, and customizable Home shortcuts.
- WebKit content blocking, editable blocking lists, and optional user JavaScript hacks.
- Page zoom, volume, printing, media discovery, and a console with JavaScript, DOM/source, and network views.
- AI chat with selectable providers and per-profile credentials stored in macOS Keychain.
- Newspaper collects updates from selected pages and feeds; optional AI helps identify new entries.
- Music playback and optional local media tools; translation and Phoenician vocabulary learning.
- Optional Tor routing and custom proxy configuration.

## Architecture

The Mac app is written in **Swift**, using **SwiftUI/AppKit** for its interface and **WKWebView/WebKit** for web content. It uses the system WebKit engine rather than shipping Chromium. Foundation provides networking and persistence; Security/Keychain stores provider credentials. JavaScript bridges support page inspection, media extraction, translation, and user scripts.

| Source | Responsibility |
| --- | --- |
| `macos/MinimalBrowser/MinimalBrowserApp.swift` | Application entry point and window setup |
| `BrowserModel.swift`, `BrowserWebView.swift` | WebKit navigation, delegates, browser state, native view bridge |
| `BrowserView.swift`, `ControlPages.swift`, `ConsoleDock.swift` | Browser chrome, settings, and developer/AI console |
| `TabStore.swift` | Tabs, sessions, library records, and site-cache cleanup |
| `ProfileFeatures.swift`, `BrowserServices.swift` | Profiles, preferences, lists, blocking, scripts, and page bridge |
| `AIService.swift`, `AIKeychain.swift` | Provider requests and credential storage |
| `NewsCore.swift`, `SocialNewsReader.swift` | Feed/page reading and Newspaper |
| `MusicCore.swift`, `MusicLibrary.swift` | Music metadata, playback/library support, subprocess tools |
| `PageTools.swift`, `PageTranslationCore.swift`, `WebPageTranslation.swift` | Page utilities and translation |
| `macos/Tests/` | Native regression checks and local fixtures |
| `windows/` | Experimental Electron implementation and Windows checks |
| `scripts/` | Local packaging scripts |
| `docs/` | Version history and release notes |

Paths in the table without a directory prefix are under `macos/MinimalBrowser/`.

## Build on macOS

Install Xcode with its macOS SDK and command-line tools. The deployment target is macOS 15.5; the project uses Swift 5 language mode.

```sh
open macos/MinimalBrowser.xcodeproj
```

Select the **MinimalBrowser** scheme. The product is **Yamm.app**. To build without distribution signing:

```sh
xcodebuild -project macos/MinimalBrowser.xcodeproj \
  -scheme MinimalBrowser -configuration Release \
  -derivedDataPath /tmp/Yamm-Build CODE_SIGNING_ALLOWED=NO build
```

### Optional bundled runtimes

The local development source includes `macos/MinimalBrowser/MusicRuntime/` and `TorRuntime/`. These binary dependencies are intentionally excluded from Git history, as are the Windows runtimes. A source clone therefore does **not** include every resource needed to reproduce the full installer. Restore those folders from your verified local runtime distribution before building the complete app; the Xcode project has resource references to them.

Music tools include yt-dlp, Deno, and FFmpeg, invoked as separate processes. Preserve their version manifests, hashes, upstream licenses, and notices. Redistribution must satisfy the licenses for the exact bundled binaries, including corresponding-source obligations where applicable. This repository does not supply or grant a license to those binaries.

With the required runtimes restored, package a universal app, DMG, and ZIP:

```sh
bash scripts/package-macos.sh
```

Artifacts are written to `releases/macos/`. The packaging script ad-hoc signs the app and does not publish it. Developer ID signing/notarization requires a separate distribution workflow.

## Checks

The native check scripts compile Swift test executables and generally use isolated temporary data directories. For example:

```sh
bash macos/Tests/run-checks.sh
bash macos/Tests/run-session-checks.sh
bash macos/Tests/run-ai-checks.sh
bash macos/Tests/run-page-translation-checks.sh
```

Other scripts cover downloads, blocking, caching/TLS, formatting, music, and page tools. Some checks require local fixture servers, WebKit, network access, or optional runtimes. Fixture success does not verify a live AI provider account or every website.

For the experimental Windows port:

```sh
cd windows
npm ci
npm test
npm start
```

See `windows/README.md` for its current limitations. Windows packaging also requires its separate runtime resources.

## Data and privacy

Local JSON data lives under `~/Library/Application Support/Yamm/`, with additional profiles under `Profiles/`. WebKit manages website storage. Provider keys are stored separately in Keychain; no API key is required in the source tree.

Partial private mode pauses history while retaining the existing website session; cache cleanup occurs when eligible tabs close or during normal quit. Full private mode uses a nonpersistent WebKit data store. These modes have different storage behavior and are not anonymity guarantees.

Explicit AI questions can send a bounded excerpt of the current page, selected text, and page metadata to the selected provider. Translation and optional AI Newspaper features also send their selected content. Provider accounts, quotas, and policies apply. Review the in-app controls before using them with private content.

## Project status and attribution

Version 0.0.7 refines the page settings menu; see [Mac release notes](docs/MAC-0.0.7.md). Historical Mac documentation retains earlier internal 2.x version names and recovery notes; those headings are not the current release number. This source lineage includes recovered code; see the retained Mac documentation for details.

Third-party blocklists, vocabulary data, artwork, and runtime dependencies retain their own notices. No overall open-source license has been selected for Yamm in this repository; public source availability alone does not grant broad reuse rights.
