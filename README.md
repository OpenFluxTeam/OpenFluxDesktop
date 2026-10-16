<p align="center">
  <img src="docs/images/icon.png" width="120" alt="OpenFlux">
</p>

<h1 align="center">OpenFluxDesktop</h1>

<p align="center">
  The desktop client of OpenFlux, a network stack research tool, for Windows, macOS and Linux.<br>
  An IPv4 TCP/UDP tunnel over pluggable transports, with end-to-end encryption, failover and your own exit nodes.
</p>

<p align="center">
  <a href="../../releases/latest"><img alt="Release" src="https://img.shields.io/github/v/release/OpenFluxTeam/OpenFluxDesktop?display_name=tag&color=4F7CFF"></a>
  <a href="LICENSE"><img alt="License" src="https://img.shields.io/badge/license-GPL--3.0--or--later-blue"></a>
  <img alt="Platforms" src="https://img.shields.io/badge/platforms-Windows%20%C2%B7%20macOS%20%C2%B7%20Linux-lightgrey">
</p>

<p align="center">
  <b>English</b> · <a href="README.ru.md">Русский</a>
</p>

<p align="center">
  <a href="https://github.com/OpenFluxTeam/OpenFlux">Core</a> · <a href="https://github.com/OpenFluxTeam/OpenFluxDesktop">Desktop</a> · <a href="https://github.com/OpenFluxTeam/OpenFluxAndroid">Android</a> · <a href="https://testflight.apple.com/join/BwnAcdus">iOS (TestFlight; the App Store release is coming)</a>
</p>

## What it is

[OpenFlux](https://github.com/OpenFluxTeam/OpenFlux) is a network stack research tool: an IPv4 TCP/UDP tunnel with pluggable transports (including a signed JS scripting engine for writing new ones), a batched + zstd codec, and two exit-node backends (L3 raw forward / L4 gVisor proxy).

OpenFluxDesktop is its desktop client. It bundles the core, runs it as a separate process and drives it over IPC; the app itself manages profiles, transports, keys and exit nodes.

```
programs ──> Wintun / utun  or  SOCKS5 / HTTP ──> OpenFlux core ──> session of transports ──> exit node ──> Internet
                                                  (bundled)         priorities, failover       --mode l4 | l3
```

## Capabilities

### Tunnel

- **Full tunnel** through a virtual network interface: Wintun on Windows, utun on macOS (needs administrator rights).
- **Proxy mode**: a local SOCKS5 (`127.0.0.1:1080`) and HTTP (`127.0.0.1:1081`) proxy for the programs you point at it. On Windows the app can set the system proxy (WinInet) for the time it is connected.
- **IPv4 TCP and UDP**; through an L3 exit, whole IP packets including ICMP.

### Transports and sessions

- **Native transports** of the core: Yandex Docs, Yandex Volga, Yandex Board, Mail.ru, cups.online, Telemost, Avito, MAX/OneMe, and `direct` — a plain TCP connection to your node.
- **JS transports**: signed packages for the core's scripting engine, installed from the bundle, from GitHub or by a link. The core checks each package's signature; the app shows its parameters and sends the package to your nodes.
- **Multi-transport sessions**: a profile holds several transports, each with a priority and the networks it may use. Negotiation is authenticated; when the active carrier stalls, the session moves to the next one without tearing the tunnel down.
- **Direct first**: while `direct` to the node is alive it carries the traffic ahead of the document carriers, with a 16000-byte tunnel MTU, pooled buffers and Reno congestion control on its streams.
- **Stripe groups**: several carriers of one or mixed types work as one stripe, one lane each.
- **Codec** per profile: batched + zstd, or the legacy per-packet LZ4 codec for old nodes.

### Encryption

- **AES-256-GCM end to end** between the client and the exit node, with a key per channel. With a key, the service that carries the documents sees only ciphertext.
- The wizard generates node keys; they travel inside the profile link and are not sent anywhere else.

### Exit nodes

- **Node wizard over SSH**: installs the core's exit node on a server (Linux amd64, arm64, armv7) and sets its backend and transports:
  - **L4** — a userspace gVisor TCP/IP stack that terminates connections and opens new ones; needs no root and runs anywhere;
  - **L3** — raw forwarding of IP packets with SNAT/DNAT, one connection end to end; needs root and iptables.
- **Verification**: after setup the app connects through the new node and checks that the exit address is the server's.
- **A server's channels**: list them, change their transports (the key, port and context stay), share or delete them, add new ones.
- **No server at all**: the exit can be a small PHP program on ordinary web hosting.
- **Self-updating nodes**: a node fetches new releases, checks them against `SHA256SUMS`, and rolls back if the new build does not start.
- **This computer as an exit**: the *Exit node* mode makes the app a node that other clients connect through (L4, or L3 with WinDivert on Windows).

### Also

- **`openflux://` links and QR codes** carry a whole profile: transports, priorities, key, codec.
- **Built-in browser** (off-screen Chromium via KCEF, about 230 MB, downloaded on first use) for the checks some services ask for, such as SmartCaptcha or a login wall.
- **Core log** with debug levels, readable in the app.
- **Updates** from GitHub releases, checked against the release's `SHA256SUMS.txt` and installed only when you agree.
- **Optional account** at openflux.dev for subscriptions; everything above works without one.

<!-- team:start -->
## What is new in the current release

The release **3.9** carries the OpenFlux core 0.14.0 (exit node `node-v1.7.0`).

- **Direct first**: a plain TCP connection to the node is used ahead of the document carriers while it is alive.
- **Speed**: a 16000-byte tunnel MTU on direct with a fallback on ICMP "fragmentation needed", pooled send buffers, gated zstd (never over TLS on direct), Reno on direct streams.
- **Stripe groups** in the core instead of the old bonding.
- **A restarted node is taken over at once**: a restarted exit answers a client that is mid-session.

The full list is in [CHANGELOG.md](CHANGELOG.md).
<!-- team:end -->

## Install

Download the file for your system from the [releases page](../../releases/latest).

| System | File |
|---|---|
| Windows 10/11 x64 | `.msi` or `-setup.exe` to install, `-portable.zip` to run without installing |
| Linux x64 | `.deb` (Debian, Ubuntu, Mint) or `.tar.gz` |
| macOS | `-macos-arm64.dmg` (Apple silicon) or `-macos-x64.dmg` (Intel) |

- The macOS builds are not signed: the first time, open the app with a right click → *Open*.
- Full-tunnel mode needs administrator rights (a UAC prompt on Windows, the password on macOS). On Linux, point programs at the SOCKS5 proxy `127.0.0.1:1080` or the HTTP proxy `127.0.0.1:1081`.

## Build from source

Needs JDK 17 and Go (or Docker, to build without Go).

```bash
git clone --recurse-submodules https://github.com/OpenFluxTeam/OpenFluxDesktop.git
cd OpenFluxDesktop
./gradlew :desktopApp:run          # run; builds the core for this machine on the first run
```

Already cloned without `--recurse-submodules`? Run `git submodule update --init --recursive`.

| Command | What it does |
|---|---|
| `./gradlew :desktopApp:run` | run the app |
| `scripts/build-core.sh` | build the core by hand from the `OpenFlux/` submodule |
| `./gradlew :shared:jvmTest` | run the tests |
| `./gradlew -PwindowsPackage=true :desktopApp:packageMsi` | Windows `.msi` (also `packageExe`) |
| `./gradlew :desktopApp:packageDmg` | macOS `.dmg` |
| `./gradlew :desktopApp:packageDeb` | Linux `.deb` (needs `fakeroot`) |

The core binaries are not committed to git: `run` and `package*` build the one for this machine into `desktopApp/resources/<windows|macos|linux>` if it is not there. `-PskipCore=true` runs the app without a core; *Settings → Core* then takes a file of your own.

## Structure

```
desktopApp/   Window, tray, shortcuts, packaging
shared/       Submodule: models, sessions, transports' settings, node wizard, screens
  jvmMain/
    core/     Core process launch, IPC status, Yandex checks
    node/     The node wizard (core --node-wizard over stdin/stdout)
    web/      The built-in browser: off-screen KCEF, a proxy router, a log
    platform/ Windows system proxy (WinInet), administrator rights
    data/     Settings, profiles and installed JS transports
OpenFlux/     Submodule: the core (CLI), started as a subprocess
scripts/      build-core.sh
```

The Android client is [OpenFluxAndroid](https://github.com/OpenFluxTeam/OpenFluxAndroid); both use the module [OpenFluxClientShared](https://github.com/OpenFluxTeam/OpenFluxClientShared).

## Credits

- **[meepo161](https://github.com/meepo161)** — author of [OpenFluxClient](https://github.com/meepo161/OpenFluxClient), the source of this app's logic (`desktopApp/`, `shared/`): multi-transport sessions with failover, encryption, the built-in browser and check flow, the node wizard. Thank you!
- **[p1neappleXpress](https://github.com/p1neappleXpress)** — author of the [OpenFlux](https://github.com/p1neappleXpress/OpenFlux) core this app bundles: the tunnel, the transports and the negotiation protocol.

The previous, independent desktop client that lived in this repository is kept at the [`legacy-native-app`](../../tree/legacy-native-app) tag.

## License

GNU General Public License v3.0 or later — see [LICENSE](LICENSE).
