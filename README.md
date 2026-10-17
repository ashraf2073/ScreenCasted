# ScreenCast

Mirror an Android phone to any laptop browser. Nothing to install on the laptop, no USB debugging, no root.

**Made par Ashraf**

## Build

1. Open the `ScreenCast` folder in Android Studio (Ladybug or newer). Let Gradle sync.
   The Gradle/AGP/Kotlin versions in `build.gradle.kts` are pinned; accept Studio's upgrade
   suggestions if it offers them.
2. Run on a phone running Android 8.0 (API 26) or newer.

## Use

1. On the phone, tick both consent boxes, tap **Start mirroring**, accept the system capture dialog.
2. The app shows an address (for example `http://192.168.42.129:8080`) and a 6-digit PIN.
3. On the laptop, open that address in Chrome, Edge or Firefox, tick the consent box, enter the PIN.

| Link | What to do first | Typical address |
|------|------------------|-----------------|
| USB cable (preferred) | Plug phone into laptop, switch on **USB tethering** (Settings > Network > Hotspot & tethering) | `192.168.42.129` |
| Wi-Fi hotspot | Switch on the phone's hotspot, join it from the laptop | `192.168.43.1` (varies) |
| Same Wi-Fi | Put both on the same network | the phone's LAN IP |
| Bluetooth | Pair the laptop, switch on **Bluetooth tethering** on the phone | `192.168.44.1` |

The app lists every address it finds, best link first, and refreshes the list every 3 seconds, so you
can enable tethering after starting.

## How it works

`MediaProjection -> VirtualDisplay -> ImageReader -> JPEG -> WebSocket -> <canvas>`

* `ScreenStreamService` - mediaProjection foreground service; capture, encode, optional audio.
* `EmbeddedServer` - NanoHTTPD/NanoWSD. Serves `assets/viewer.html` and the `/ws` stream.
  Each viewer has its own sender thread; a slow viewer drops frames instead of stalling the phone.
* `NetworkInfoProvider` - finds USB / hotspot / Wi-Fi / Bluetooth addresses.
* `MainActivity` - Compose UI, consent checkboxes, permission flow.

JPEG frames over WebSocket are used rather than H.264 because browsers only allow WebCodecs on secure
contexts (https or localhost), and a plain `http://192.168...` page is neither.

## Security

* Every session has a random PIN; five wrong tries from one address lock it out for a minute.
* Both the phone and the browser page ask for explicit consent before anything is captured or shown.
* Traffic is plain HTTP/WebSocket on your own link. Use it on networks and cables you trust.
* Stop from the app or the notification. Android also shows its own capture indicator.

## Limits to know about

* Android does not let apps switch on USB tethering, hotspot or Bluetooth tethering for you; the app opens the right Settings page.
* USB tethering behaves like a fast local network, so latency is low, but it is not zero.
* A browser cannot open a Bluetooth RFCOMM socket, so "Bluetooth" means Bluetooth PAN tethering. Real-world throughput is about 1-3 Mbit/s; the app switches to 480 px / 10 fps when it is the only link.
* Android 14+ lets the user share a single app instead of the whole screen.
* Screens marked secure (banking apps, DRM video) are captured as black. Some apps block audio capture.
* Audio needs Android 10+ and the microphone-type permission prompt (it captures playback, not the microphone).
* View-only: there is no remote control from the laptop.
