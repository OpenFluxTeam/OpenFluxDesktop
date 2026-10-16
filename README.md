# OpenFluxDesktop

Desktop client for [OpenFlux](https://github.com/OpenFluxTeam/OpenFlux)
(Windows/macOS/Linux): full-tunnel or SOCKS5/HTTP proxy, multi-transport
sessions with automatic failover, AES-256-GCM encryption, a node-deployment
wizard (deploy an exit over SSH from the app), and a built-in browser for
passing a transport's check (SmartCaptcha, a login wall).

Current release: **3.8.0**, with the OpenFlux core 0.14.0 (node-v1.7.0). Releases
are on the [releases page](https://github.com/OpenFluxTeam/OpenFluxDesktop/releases);
the app checks for them itself (Settings → About and updates) and installs one
only when you agree.

What the current core gives the app:

- **Direct first.** A `direct` transport (a plain TCP connection to the node)
  is used ahead of the document carriers (Yandex, Mail.ru, cups.online, MAX…)
  while it is alive, and the document carriers stay as the reserve. A new
  `direct` transport in a profile starts at priority 110.
- **Speed.** Large packets on direct, pooled buffers, compression only where it
  pays, Reno on direct streams. The measurements and the method are in the
  core's `TRANSPORT_PERFORMANCE.md`.
- **Stripe groups.** Several carriers of one or mixed types can work as one
  stripe; the app has no screen for it yet, set `Lanes = N` in a `.conf`.
- **A restarted node is taken over at once**, not after 30 s of silence.

This repository's app code and UI (the `desktopApp/` and `shared/` modules)
come from [meepo161/OpenFluxClient](https://github.com/meepo161/OpenFluxClient),
used here with the author's agreement. **Huge thanks to
[@meepo161](https://github.com/meepo161)** — see [Credits](#credits) below.
The previous, independent desktop client that used to live in this
repository is preserved at the [`legacy-native-app`](../../tree/legacy-native-app)
tag.

## Getting the code

```bash
git clone --recurse-submodules https://github.com/OpenFluxTeam/OpenFluxDesktop.git
```

Already cloned without `--recurse-submodules`?

```bash
git submodule update --init --recursive
```

This checks out two submodules:

- `shared/` → [OpenFluxClientShared](https://github.com/OpenFluxTeam/OpenFluxClientShared),
  the Compose Multiplatform UI and models shared with
  [OpenFluxAndroid](https://github.com/OpenFluxTeam/OpenFluxAndroid).
- `OpenFlux/` → [OpenFlux](https://github.com/OpenFluxTeam/OpenFlux), the
  core this app bundles as a separate process.

## Building

Needs JDK 17 and Go (or Docker, for a build without Go). Core binaries are
not committed to git: `run`/`package*` build the one for this machine into
`desktopApp/resources/<windows|macos|linux>` automatically (from the
`OpenFlux/` submodule, `-PcoreDir=<path>`, or `../OpenFlux` next to this
repository) if it isn't already there. `-PskipCore=true` runs the app
without a core (Settings → Core then takes a file of your own).

```bash
./gradlew :desktopApp:run                              # run (builds the core on first run)
scripts/build-core.sh                                   # core by hand, from the OpenFlux/ submodule
./gradlew :shared:jvmTest                                # tests
./gradlew -PwindowsPackage=true :desktopApp:packageMsi   # Windows: .msi (also packageExe)
./gradlew :desktopApp:packageDmg                         # macOS: .dmg
./gradlew :desktopApp:packageDeb                         # Linux: .deb (needs fakeroot)
```

## Structure

```
desktopApp/   Window, tray, shortcuts, packaging, icons
shared/       Submodule: models, service interfaces, design system, screens
  jvmMain/
    core/     Core process launch, IPC status, Yandex checks
    node/     The node wizard (core --node-wizard over stdin/stdout)
    web/      The built-in browser: off-screen KCEF, a proxy router, a log
    platform/ Windows system proxy (WinInet), admin rights
    data/     Settings and profile storage
OpenFlux/     Submodule: the core (CLI), spawned as a subprocess
scripts/      build-core.sh
```

## Credits

- **[meepo161](https://github.com/meepo161)** — author of
  [OpenFluxClient](https://github.com/meepo161/OpenFluxClient), the source of
  this app's UI and logic (`desktopApp/`, `shared/`): multi-transport
  sessions with failover, AES-256-GCM encryption, the built-in browser and
  check/captcha flow, the node-deployment wizard, and the Compose design
  system. Thank you!
- **[p1neappleXpress](https://github.com/p1neappleXpress)** — author of the
  [OpenFlux](https://github.com/p1neappleXpress/OpenFlux) core this app
  bundles: the tunnel, transports, and negotiation protocol.
- **[damnurmum](https://github.com/damnurmum)** — author of
  [OpenFlux-Android](https://github.com/damnurmum/OpenFlux-Android) and,
  in the core, `openflux://` links and QR codes, and the cups.online
  transport, which this app's share and scan screens build on. Thank you!

## License

GNU General Public License v3.0 or later — see [LICENSE](LICENSE).
