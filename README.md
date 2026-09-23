# DuckGPS

DuckGPS lets an Android phone with a working GPS stream its live location to
another Android phone whose GPS is broken. The receiver acts as a **mock
location provider**, so any app on it (maps, find-a-friend, etc.) sees the
sender's real position.

- Works over the local network (both phones on the same Wi-Fi).
- No account, no cloud, no third-party servers. Plain TCP socket, direct phone-to-phone.
- Runs in the background via a foreground service with a persistent notification.
- ~830 KB debug APK. Pure Android framework + Kotlin standard library — no other dependencies.

## How it works

| Role | Phone | What it does |
|------|-------|--------------|
| **Receiver** | Broken GPS | Hosts a TCP server on port `5555`, feeds every incoming fix into Android's mock-location (GPS + network) providers, and keeps the screen-off resume. |
| **Sender** | Working GPS | Connects to the receiver's IP, requests the location providers (GPS / network / fused), and streams `lat,lon,alt,accuracy,speed,bearing,time` lines. |

## Install

1. Install `DuckGPS.apk` on both phones (allow "install from unknown source" if prompted).
2. On the **receiver** phone, enable **Developer options** (Settings > About phone > tap "Build number" 7 times), then go to:

   > Developer options → **Select mock location app** → **DuckGPS**

   This one-time step is required by Android — an app cannot grant itself mock-location power.

## Use

1. **Both phones on the same Wi-Fi network.**
2. On the **receiver** (broken GPS) phone: open DuckGPS and tap **"Start receiving"**.
   The app shows this phone's IP and port (e.g. `192.168.1.25:5555`).
3. On the **sender** (working GPS) phone: enter the receiver's IP address and tap **"Send GPS"**.
4. The receiver's status changes to "Sender connected. Mock location ACTIVE."
   The mock location keeps updating in the background while both apps run.

> Tip: with mock locations on Android 12+, enable GPS / battery-saving location
> on the receiver so apps like Google Maps see the mocked fix.

## Build from source

Prerequisites: JDK 17 and the Android SDK (platform 35, build-tools 34).

```sh
./gradlew assembleDebug
# output: app/build/outputs/apk/debug/app-debug.apk
```

On Windows: `gradlew.bat assembleDebug`.

## Permissions

- `INTERNET` — phone-to-phone socket connection.
- `ACCESS_FINE_LOCATION` / `ACCESS_COARSE_LOCATION` — reading the sender's GPS / setting the receiver's mock location.
- `FOREGROUND_SERVICE` / `FOREGROUND_SERVICE_LOCATION` — keep streaming in the background.
- `POST_NOTIFICATIONS` — show the persistent foreground-service notification (Android 13+).

## Layout

```
app/src/main/java/com/duckgps/
  MainActivity.kt   - minimal single-screen UI (sender / receiver controls)
  GpsService.kt     - foreground service: sender streaming, receiver listener + mock provider
app/src/main/res/   - layout, theme, launcher icon
```

## Disclaimer

Mock locations are visible to apps that check for mocked providers. Use this
tool responsibly — fake-location abuse (e.g. location cheating) may violate
terms of service of the apps you target.