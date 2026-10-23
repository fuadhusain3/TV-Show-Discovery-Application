# TVMaze – TV Show Discovery (Flutter)

Flutter + flutter_bloc + layered architecture, powered by https://api.tvmaze.com
(no API key needed).

## Features
- **Home** – Trending row, infinite-scroll "Popular Shows" grid (`/shows?page=N`),
  pull-to-refresh, per-page retry, responsive column count.
- **Search** – debounced (500 ms) search with cancel-previous-request, loading,
  empty and error states (`/search/shows?q=`).
- **Show details** – poster, rating, years, genres, summary; tabs for Overview,
  **Episodes grouped by season** (season picker + episode sheet) and **Cast**.
  Details, cast and episodes load independently, so one failing never hides the others.
- **Watchlist** – heart buttons on cards, search rows and the details screen, a
  full Watchlist tab (posters, titles, empty state, undo, clear all). Persisted
  with SharedPreferences, no duplicates, survives restarts and works offline.
- **Profile** – watchlist shortcut and data credit.

### Capturing them
With the app running on an emulator or a USB-connected Android device:

```bash
adb exec-out screencap -p > docs/screenshots/home.png
```

Repeat for each screen above, changing the file name. On an iOS simulator use
`xcrun simctl io booted screenshot docs/screenshots/home.png`. Portrait phone
captures around 1080 px wide look best; keep each file under ~500 KB.

## Requirements
Flutter **3.27 or newer** (stable; Dart 3.6+). Check with `flutter --version`.

## Setup & run
This zip contains the Dart code only (`lib/`, `test/`, `pubspec.yaml`). Generate the
platform folders once, then run:

```bash
cd tvmaze_app

# 1. Create android/ ios/ web/ (does not touch lib/, pubspec.yaml or test/)
flutter create --project-name tvmaze_app --platforms=android,ios,web .

# 2. Install dependencies
flutter pub get

# 3. (optional) static analysis + tests
flutter analyze
flutter test

# 4. Run on a connected device / emulator / simulator
flutter devices
flutter run            # or: flutter run -d <device-id>
```

If `flutter create` added its own `test/widget_test.dart` template (referencing
`MyApp`), delete it – this project ships its own tests.

### Android release builds
Debug builds already have network access. For **release** builds add this line inside
`<manifest>` (above `<application>`) in `android/app/src/main/AndroidManifest.xml`:

```xml
<uses-permission android:name="android.permission.INTERNET"/>
```

### Notes
- The first launch needs internet: TVMaze data and the Inter font (google_fonts)
  are downloaded and cached. Offline, text falls back to the system font.
- iOS needs macOS + Xcode (`flutter doctor` lists anything missing).

## Architecture (data flows right to left)
`screens/widgets -> blocs -> repositories -> services -> core/network (Dio)`

- No widget talks to the network: `ApiClient` (Dio) is only used by `ShowApiService`;
  all URLs live in `core/constants/api_constants.dart`.
- Services parse JSON into models and skip malformed entries instead of failing a page.
- BLoCs: `HomeBloc` (pagination), `SearchBloc` (debounce + restartable),
  `ShowDetailsBloc` (three independent sections), `WatchlistBloc` (app-wide,
  single sequential queue so storage writes never interleave).
- Dependency injection with `get_it` (`core/di/injector.dart`).

## Tests (`flutter test`)
`models_test` (JSON parsing), `data_layer_test` (service + repository against a fake
client), `blocs_test` (pagination, debounce, details), `watchlist_test` (storage + BLoC),
`widget_test` (watchlist screen).
