# Handoff: NYC Subway v2 (written 2026-11-12)

For the next Claude session picking this up. Written on Gio's work Mac.

## What this is

Joe (Light Phone co-founder) sent design feedback on the NYC Subway tool. We rewrote the app
from scratch as v2.0.0 to meet it and go further. The same code now lives in two repos:

| Repo | App id | Where the code is | Ships to |
|---|---|---|---|
| `gi-os/BrightNYCSubwayLite` | `com.gios.brightnycsubwaylite` ("NYC Subway") | `tool/` | Light Tool Library |
| `gi-os/BrightTransit` | `com.thelightphone.lightnycsubway` ("Subway Times") | `examples/LightNYCSubway/` | BrightMarket, via CI on push to `main` |

The Lite repo is the source of truth. BrightTransit's copy is the same files with the Kotlin
package renamed from `com.gios.brightnycsubwaylite` to `com.thelightphone.subway`.

## State right now (updated 2026-11-12, afternoon)

Both repos are on the work Mac at `~/gi-os/`, on local branch `rewrite/v2`.

- **BrightNYCSubwayLite**: one commit, `cc64901`, which IS pushed (`origin/rewrite/v2`).
  This file is untracked on purpose.
- **BrightTransit**: one merge commit, `70393e6`, NOT pushed. It merges Light's upstream SDK
  (`lightphone/light-sdk` at `311aa2d`, SDK 0.2.0) and replaces the app with v2.
- Pushing BrightTransit to `main` cuts a BrightMarket release (workflow `build-apk.yml`).
- **On top of those commits, UNCOMMITTED in both working trees:** service alerts and the L's
  "Brooklyn" direction (see below). Tests pass in both (85 each) and both debug builds are
  green. Nothing has been committed or pushed since the commits above.

## What v2 does

- Home is only saved stations; blank state is centred copy. One bottom row everywhere:
  nearby (pin), search, refresh. "Save / Unsave" wording; text buttons in ALL CAPS.
- Station page: SEARCH, SAVE STATION / UNSAVE, REFRESH. Tap a line to favorite it; favorite
  trains carry a star.
- Trains are grouped by where they are headed, not by the feed's north/south. Headings use
  the MTA's sign for the platform when it says something useful, else the borough the train
  ends in, else the last stop's name. Each train shows its real last stop from the live feed.
- Search: words in any order, abbreviations, one or two typos, nicknames, falls back to the
  closest stations. Opens straight on the keyboard; backing out with nothing searched goes home.
- Nearby: closest stations with distances; the nearest shows its live board. Location comes
  from LightOS service calls.
- Times count down locally, re-fetch every 30 s while the screen shows, and stay up marked
  "Not live" if a fetch fails. One shared feed cache (20 s fresh, 20 min max age).
- Content fades in when it arrives; times dim while a manual refresh is in flight.
- Saved ids written by older versions (platform ids like `L12`) are converted on first launch.

Added 2026-11-12 (uncommitted):

- **Service alerts.** Station page: the MTA alerts in effect now, worst first, the first two
  above the times and the rest below them. Home and the top Nearby board: one grey line per
  station (first alert's headline, "+N" if more), station notices left out.
  - An alert shows at a station if it names one of its stops, or is line-wide for one of its
    lines (delays), or is a suspension anywhere on one of its lines. Shuttles are told apart
    by stop id (GS / FS / H), expresses fold into their line.
  - One shared download (about 89 KB gzipped, 561 KB raw), after the trains, never awaited:
    fresh 3 min, refresh button at most once a minute, retry after 2 then 4 min, "Alerts as
    of h:mm" after 10 min, dropped after 60. A failed alerts download shows nothing and never
    touches the train times.
- **The L reads "Manhattan / Brooklyn"** at all 18 of its stops outside Manhattan (was
  "Canarsie" or the turn-back stop). Done in `build_stations.py`, so it is data, not logic.
  This closes the old open question about Joe expecting "Brooklyn".
- `LiveBoards.follow()` now carries a pressed refresh over instead of dropping it (Nearby).

## Code map (Lite repo, under `tool/src/main/kotlin/com/gios/brightnycsubwaylite/`)

- `data/Stations.kt` catalog; `data/StationSearch.kt` search; `data/GtfsRealtime.kt` protobuf
  reader (trains: `parse`, alerts: `alerts`); `data/Arrivals.kt` directions, grouping, feeds;
  `data/ArrivalsRepository.kt` train cache and `MtaHttp`; `data/Alerts.kt` which alerts show
  where and their wording; `data/AlertsRepository.kt` alerts download;
  `data/SavedStore.kt` saved stations and favorite lines.
- `Subway.kt` shared objects and `LiveBoards` (auto refresh). `ui/Components.kt` all shared UI.
- `HomeScreen.kt`, `StationScreen.kt`, `SearchScreen.kt` (also the keyboard screen), `NearbyScreen.kt`.
- `tool/scripts/build_stations.py` regenerates `assets/stations.json` from `Stations.csv`.
  `tool/scripts/trim_alerts.py` cuts a live alerts download down to the test fixture.
- Tests in `tool/src/test/` (85, all passing). Fixtures: `gtfs-l.pb` is a real L feed,
  `subway-alerts.pb` is 30 real alerts (14 in effect at its timestamp 1794506045).

## Commands

```sh
export ANDROID_HOME=$HOME/Library/Android/sdk   # not set in the shell on the work Mac
# Lite repo
./gradlew -DlightSdk.toolOnly=true :tool:testDebugUnitTest :tool:assembleDebug
# BrightTransit
./gradlew -DlightSdk.toolOnly=true :examples:LightNYCSubway:testDebugUnitTest :examples:LightNYCSubway:assembleDebug
```

After changing code in the Lite repo, copy it into BrightTransit: replace
`examples/LightNYCSubway/src/main/kotlin/com/thelightphone/subway` and the tests with the Lite
files, then replace `com.gios.brightnycsubwaylite` with `com.thelightphone.subway` in them.
Also copy `stations.json`, `gtfs-l.pb`, `subway-alerts.pb` and `scripts/` (both `.py` files;
in BrightTransit their docstrings say `examples/LightNYCSubway/scripts/` and `src/main/assets`
instead of `tool/...`). As of this update the two copies match.

The SDK's build check rejects reflection (`::class.java`), even in tests.

## Tested how

- Android emulator (AVD `lp3`, AOSP Android 14 image, Light's emulator app as a system app):
  home, search, station, save, nearby all worked with live MTA data.
- Gio's Light Phone: v2 is installed over Subway Times (BrightTransit's own build). Saved
  stations and favorites carried over. Home screen seen working with real data.
- Not seen on a device: the fade-ins, refresh dimming, the search back fix (Gio said it
  worked before the fade-in build), the scroll wheel, the light theme.
- A second, empty copy ("NYC Subway", the Lite id) is also on the phone and can be removed.
- 2026-11-12, emulator only (not the phone): alerts and the L change seen working with live
  MTA data on Home (Morgan Av, 14 St-Union Sq) and both station pages, dark theme. Not seen:
  a station with three or more alerts ("N more alerts below."), the "Alerts as of" note, the
  light theme, Nearby with an alert.
- Emulator build: set `serverPackage = "com.thelightphone.sdk.emulator"` in `tool/lighttool.toml`
  for that one build, add `-DlightSdk.allowAltServerPackage=true`, then put the file back.
  Boot with `emulator -avd lp3 -writable-system`. Use `ANDROID_SERIAL=emulator-5554` for every
  adb command: Gio's real Light Phone is often plugged in at the same time.

## Ride mode (2026-11-16)

Three parts, none in v2.0.0's commit: a train's stops (`TrainScreen.kt`, `data/Train.kt`),
the count-down to a stop (`RideScreen.kt`, `data/Ride.kt`, `LiveRide` and `LiveOtherTrains`
in `Subway.kt`) and direct trains (`DirectScreen.kt`, `data/Planner.kt`, the block in
`StationScreen.kt`). Every sentence a rider reads is in `ui/RideWords.kt`, `ui/PlanWords.kt`
or an `...Words` object beside its screen, and is pinned by a test. None of them names where
the times come from.

- What is promised, and how it is worded to Light: the count never rises while no new word
  arrives; the grey count runs early and can be put right by a stop, rarely two, when word
  comes back (two in 10 of 2,274 cases on the L replay; other lines were not measured); a
  train that was behind its times when last heard of is not counted on at all. A train is
  called held by what its last word said, never by the clock since, so "Standing at ... since"
  is never followed by a count made from the times it has just disowned.
- Tried and dropped: calling a lettered-line train held only once it is also a minute past
  the time first listed for its next stop. On Monday's recording that stopped 8 stays being
  called held, and 7 of the 8 trains did reach the next stop over a minute late. The plain
  four-minute rule called 45 stays held and was wrong about one.
- A stop a followed train once listed is never forgotten: it is ahead, left or dropped. A
  trip first given its train some stops down its list has left the stops before it, with no
  time put to that, and one of them listed again is given back at once.
- Fixtures: `snapshots/weekday-midday` keeps the feeds' positions (the afternoon one has
  none), and `replays/` holds one train each from Monday's recording. `cut_replay.py` lists
  them. The mirrored repo needs all of them.
- The planner leaves out a trip with no train past its start only on the numbered lines, the
  L and the railway. The railway's place in that list rests on a hand-built test alone:
  Monday's recording holds no railway trip past its start without a train.
- By hand on the emulator, because no JVM test can build a screen: back from a ride, then
  relaunch, lands on Home (the screen's `onScreenDestroy` is what ends a ride); search from
  a station's page shows its one grey line under the keyboard's text line; relaunch on a
  saved ride whose train has gone lands on Home (`HomeScreen` passes `resumed = true`, which
  is what has the ride judged); a ride left asleep until ten minutes after its train was
  due closes on wake (`RideViewModel.onAppPause` calls `ride.paused()`, not `hidden()`).
- Not seen on a phone: Akkurat, the light theme, the wheel on the new screens, and whether
  LightOS keeps the tool alive across a sleep.

## Next steps, in order

1. Commit the alerts and L work in both repos (Gio has not asked for a commit yet). Push when
   Gio says; BrightTransit's branch has never been pushed.
2. Screenshots: the v2 ones predate alerts and still show "Canarsie" for the L. The Lite
   README text is updated; BrightTransit's README still describes v1.
3. Reply to Joe. The draft is in Gmail (thread "NYC Subway Tool Submission Feedback"). Its
   line "the other way is named for where the train is going" is now out of date for the L,
   which reads "Brooklyn"; it also does not mention alerts. Then resubmit the Lite tool.
4. Not doing, per the draft reply: the collapsed view of saved stations ("I kept every line
   visible rather than collapsing"). Joe had no strong preference.
5. Possible later: alert descriptions (shuttle bus details) behind a tap. Left out on purpose.

## Open questions

- None open. (The L now says "Brooklyn" outbound, as Joe's email asked.)
