# Apollo + Artemis Game Streaming Guide

> A practical setup and troubleshooting guide for streaming PC games with **Apollo** on Windows and **Artemis** on Android / Google TV, based on the setup I actually use.

![Apollo](https://img.shields.io/badge/Host-Apollo-blue?style=flat-square)
![Artemis](https://img.shields.io/badge/Client-Artemis-purple?style=flat-square)
![Windows](https://img.shields.io/badge/Host-Windows-0078D6?style=flat-square)
![Android TV](https://img.shields.io/badge/Client-Android%20TV-green?style=flat-square)
![GTX 1650 Super](https://img.shields.io/badge/GPU-GTX%201650%20Super-76B900?style=flat-square)

## My setup

| Component | Setup |
|---|---|
| Host | Windows gaming PC |
| GPU | NVIDIA GeForce GTX 1650 Super 4 GB |
| RAM | 32 GB |
| Streaming host | Apollo |
| Main client | TCL C6K 55" Google TV / Android TV |
| Client app | Artemis |
| Network | Home LAN on an SLT Fiber router/network |
| Goal | Smooth low-latency couch gaming from PC to TV |

This README focuses on **real setup, tuning, and troubleshooting**, especially the case where the PC is fast enough but the TV/client still does not feel perfectly smooth.

---

## What are Apollo and Artemis?

**Apollo** is a Sunshine-derived self-hosted game-streaming host. It supports hardware encoding on NVIDIA, AMD, and Intel GPUs and includes a Web UI, client permissions, and integrated virtual-display support.

**Artemis** is an Android GameStream client derived from Moonlight and designed to integrate closely with Apollo.

Useful Apollo + Artemis features include:

- low-latency local game streaming;
- hardware video encoding;
- custom resolutions and bitrates;
- per-client permissions;
- keyboard/mouse/controller input;
- clipboard integration;
- Apollo virtual-display integration;
- client-specific display identity;
- automatic resolution/refresh-rate matching;
- HDR on compatible host/client combinations.

Apollo uses **SudoVDA** for its Windows virtual display.

---

## Architecture

```text
┌──────────────────────────────────────┐
│          Windows Gaming PC           │
│                                      │
│  Game → GPU Render → NVENC → Apollo  │
│                           │          │
│                        SudoVDA       │
└───────────────────────────┬──────────┘
                            │
                     Local network
                            │
┌───────────────────────────▼──────────┐
│        TCL C6K 55" Google TV         │
│                                      │
│              Artemis                 │
│                 ↓                    │
│        Hardware video decoder        │
│                 ↓                    │
│          TV display + audio          │
└──────────────────────────────────────┘
```

For local streaming, the important part is the **LAN path between host, router/AP, and client**. Your Internet download speed is not the main factor.

---

## Table of contents

1. [Requirements](#1-requirements)
2. [Install Apollo](#2-install-apollo)
3. [Open the Apollo Web UI](#3-open-the-apollo-web-ui)
4. [SudoVDA / virtual display](#4-sudovda--virtual-display)
5. [Install Artemis](#5-install-artemis)
6. [Pair Artemis with Apollo](#6-pair-artemis-with-apollo)
7. [Client permissions](#7-client-permissions)
8. [Add Desktop and games](#8-add-desktop-and-games)
9. [Recommended settings for GTX 1650 Super](#9-recommended-settings-for-gtx-1650-super)
10. [Recommended TCL C6K settings](#10-recommended-tcl-c6k-settings)
11. [Bitrate guide](#11-bitrate-guide)
12. [Codec guide](#12-codec-guide)
13. [Controller setup](#13-controller-setup)
14. [Latency tuning](#14-latency-tuning)
15. [Fixing stutter / rough motion](#15-fixing-stutter--rough-motion)
16. [Virtual-display problems](#16-virtual-display-problems)
17. [Black screen / game launch problems](#17-black-screen--game-launch-problems)
18. [Network troubleshooting](#18-network-troubleshooting)
19. [My baseline configuration](#19-my-baseline-configuration)
20. [Credits](#20-credits)

---

# 1. Requirements

## Host

- Windows 10/11
- Apollo
- supported NVIDIA / AMD / Intel encoder
- recent GPU driver
- local network connection
- Ethernet strongly recommended for the host

### My host

```text
Windows PC
├── NVIDIA GTX 1650 Super 4 GB
├── 32 GB RAM
└── Apollo
```

The GTX 1650 Super provides hardware H.264 and HEVC encoding through NVENC, which makes it a good fit for this type of streaming.

---

## Client

My main client:

```text
TCL C6K 55"
└── Google TV / Android TV
    └── Artemis
```

Artemis can also be used on Android phones and tablets.

---

## Network layouts

Best:

```text
PC ── Ethernet ── Router/Switch ── Ethernet ── TV
```

Also good:

```text
PC ── Ethernet ── Router/AP ── 5 GHz/6 GHz Wi-Fi ── TV
```

More difficult to keep consistent:

```text
PC ── Wi-Fi ── Router/AP ── Wi-Fi ── TV
```

Every wireless hop can add jitter, interference, retransmissions, or latency variation.

---

# 2. Install Apollo

Official repository:

https://github.com/ClassicOldSong/Apollo

Releases:

https://github.com/ClassicOldSong/Apollo/releases

On Windows you can also install the community-maintained WinGet package:

```powershell
winget install ClassicOldSong.Apollo
```

After installation, launch Apollo.

Allow it through Windows Firewall on the network profile you actually use.

---

# 3. Open the Apollo Web UI

Apollo normally exposes its local configuration page at:

```text
https://localhost:47990
```

You may see a local HTTPS certificate warning in the browser.

From another device on the LAN, the address is normally:

```text
https://HOST-PC-IP:47990
```

provided Apollo's Web UI access setting and the firewall allow LAN access.

---

# 4. SudoVDA / virtual display

Apollo's Windows virtual-display integration uses **SudoVDA**.

When working correctly, the flow is:

```text
Artemis requests a stream
        ↓
Apollo creates/configures virtual display
        ↓
Windows sees a client-specific monitor
        ↓
Resolution / refresh rate match client request
        ↓
Game renders to that display
```

The display normally appears when the stream starts and disappears when it ends.

### If no virtual display appears

Check that the SudoVDA driver is installed and that Apollo has been restarted after installation.

Apollo's own FAQ points to SudoVDA first when no virtual display is created.

### Avoid multiple virtual-display drivers while troubleshooting

Apollo's documentation recommends removing unrelated virtual-display solutions when debugging because several virtual adapters can cause confusing monitor-selection behaviour.

---

## If Windows mirrors the physical monitor

Press:

```text
Win + P
```

Choose:

```text
Extend
```

Then close the streamed app/session and reconnect.

Windows usually remembers the display identity afterward.

---

# 5. Install Artemis

Upstream Artemis Android repository:

https://github.com/ClassicOldSong/moonlight-android

Artemis was previously known as **Moonlight Noir**.

Install the current APK on the Android/Google TV device.

For Android TV, common options are:

- project release APK;
- ADB sideload;
- transferring the APK to the TV and installing it locally.

Open Artemis after installation.

---

# 6. Pair Artemis with Apollo

Put the PC and TV on the same LAN.

Artemis should normally discover Apollo automatically.

If not, manually add the host PC's local IPv4 address.

Example:

```text
192.168.1.50
```

Select the PC in Artemis.

Artemis displays a PIN.

Complete the pairing request through Apollo.

---

# 7. Client permissions

Apollo supports per-client permissions.

If the TV can see the host but cannot launch games or send input, check the permissions assigned to that Artemis client.

Relevant permissions can include:

- View Streams
- List Apps
- Launch Apps
- Mouse Input
- Keyboard Input
- controller/input permissions

The first paired client can have broader default access than later paired clients.

A **Permission Denied** message is therefore not automatically a network problem.

---

# 8. Add Desktop and games

In Apollo's Web UI, add the apps you want to expose.

Useful first entries:

- Desktop
- Steam
- individual games
- other launchers if required

### Why add Desktop?

Desktop is the easiest diagnostic target.

If Desktop streams correctly but one game fails, then:

```text
Apollo + network + Artemis
        = probably working
```

and the problem is more likely game/display/launcher specific.

---

# 9. Recommended settings for GTX 1650 Super

My GPU:

```text
NVIDIA GeForce GTX 1650 Super 4 GB
```

### Preferred codec

Start with:

```text
HEVC / H.265
```

Fallback:

```text
H.264
```

The GTX 1650 Super is **not an AV1 hardware-encode GPU**, so AV1 is not the target for this host.

### First resolution

```text
1920×1080 @ 60 FPS
```

Once stable:

```text
2560×1440 @ 60 FPS
```

Only then test:

```text
3840×2160 @ 60 FPS
```

A stable 60 FPS stream usually feels better than forcing 4K while the client decoder or LAN is struggling.

---

# 10. Recommended TCL C6K settings

My client is the built-in Google TV platform on a **TCL C6K 55"**.

The TV panel can support high refresh rates, but game-stream smoothness also depends on the TV's:

- MediaTek/TV SoC;
- hardware decoder;
- Android/Google TV overhead;
- Artemis frame pacing;
- picture processing;
- network interface.

So the TV can become the bottleneck even when the gaming PC has no problem rendering the game.

### Start here

```text
Resolution: 1920×1080
FPS:        60
Codec:      HEVC
Bitrate:    20–35 Mbps
HDR:        Off
```

If perfectly smooth:

```text
Resolution: 2560×1440
FPS:        60
Codec:      HEVC
Bitrate:    35–55 Mbps
```

Then try:

```text
Resolution: 3840×2160
FPS:        60
Codec:      HEVC
Bitrate:    50–80 Mbps
```

These are tuning ranges, not hard requirements.

---

## TV picture settings

Use **Game Mode / low-latency mode** when available.

While tuning, reduce or disable heavy post-processing such as:

- motion interpolation;
- noise reduction;
- excessive sharpening;
- cinema motion processing.

Those can add latency or make streamed motion feel inconsistent.

---

# 11. Bitrate guide

| Resolution | FPS | Good starting range |
|---|---:|---:|
| 720p | 60 | 10–20 Mbps |
| 1080p | 60 | 20–35 Mbps |
| 1440p | 60 | 35–55 Mbps |
| 4K | 60 | 50–80 Mbps |

Higher is not automatically better.

An unnecessarily high bitrate can increase:

- Wi-Fi retransmissions;
- decoder pressure;
- buffer bursts;
- latency spikes;
- visible stutter.

First make the stream stable. Then increase quality.

---

# 12. Codec guide

## HEVC / H.265

My preferred first choice.

Advantages:

- better compression than H.264;
- useful for 1440p/4K;
- good quality at lower bitrate.

If the TV's HEVC decoder path behaves badly, switch to H.264 as a test.

---

## H.264

Use H.264 when:

- HEVC stutters;
- decode latency looks high;
- compatibility matters more than efficiency;
- you want a clean troubleshooting baseline.

If H.264 is smooth and HEVC is not, that strongly points toward the client decode path/configuration rather than raw PC performance.

---

## AV1

AV1 is useful on newer supported GPUs, but the GTX 1650 Super does not provide modern AV1 hardware encoding.

Use the codec that your **host can encode and client can decode consistently at low latency**, not simply the newest codec.

---

# 13. Controller setup

Typical path:

```text
Controller
   ↓
TCL / Android TV
   ↓
Artemis
   ↓
LAN
   ↓
Apollo
   ↓
Windows game
```

Connect the controller to the TV/client.

Then test:

1. Android TV navigation;
2. Artemis navigation;
3. Desktop stream;
4. game input;
5. analog sticks/triggers.

If Android sees the controller but Windows does not, check Apollo client/input permissions.

Bluetooth is convenient. USB/wired input can be useful when diagnosing inconsistent input latency.

---

# 14. Latency tuning

End-to-end latency is roughly:

```text
controller/input
+ client-to-host network
+ game render
+ host encode
+ host-to-client network
+ client decode
+ TV display processing
```

Best improvements:

1. Wire the host PC with Ethernet.
2. Use Ethernet for the TV if that interface performs well.
3. Otherwise use strong 5 GHz/6 GHz Wi-Fi.
4. Enable TV Game Mode.
5. Keep game FPS stable.
6. Avoid excessive bitrate.
7. Use hardware encoding.
8. Disable unnecessary image processing.
9. Pause large LAN downloads while testing.

---

# 15. Fixing stutter / rough motion

This is the main real-world issue I investigated with my TCL C6K.

If the game looks smooth on the PC but the TV feels less smooth, the bottleneck can be anywhere here:

```text
Game render
    ↓
NVENC encode
    ↓
LAN
    ↓
TV hardware decode
    ↓
Artemis/frame pacing
    ↓
TV processing/panel
```

Change **one thing at a time**.

---

## Test A — lower resolution

```text
4K60 → 1080p60
```

If 1080p becomes smooth, the higher-resolution decode/network path is likely part of the problem.

---

## Test B — lower bitrate

Example:

```text
80 Mbps → 30 Mbps
```

If that helps, inspect the LAN and client decode path before increasing bitrate again.

---

## Test C — change codec

```text
HEVC → H.264
```

If H.264 is smooth, investigate HEVC decode performance/settings on the TV.

---

## Test D — disable HDR

Get SDR stable first.

HDR introduces additional variables including 10-bit video and display-mode switching.

---

## Test E — use performance statistics

Enable Artemis's performance/statistics overlay when available.

Watch:

- FPS;
- network latency;
- packet loss;
- decode time;
- dropped frames.

High decode time suggests the client.

Network spikes suggest the LAN.

---

## Test F — compare another client

If the exact same Apollo host is smooth on another Android device but not on the TCL, the TV/client path is the likely bottleneck.

This is why a stronger external Android/Fire TV-class streaming device can sometimes outperform a television's built-in smart-TV hardware.

---

# 16. Virtual-display problems

## No virtual display

Check:

- SudoVDA installed;
- Apollo restarted;
- Apollo logs;
- competing virtual-display drivers.

---

## Display keeps toggling on/off

Quit Apollo completely.

For stuck display state, Apollo maintainers have recommended removing:

```text
Apollo install directory/
└── config/
    └── display_device.state
```

Then start Apollo again.

Use this only when troubleshooting display-state problems.

---

## Stream mirrors the main monitor

```text
Win + P
→ Extend
```

Then reconnect.

---

## Game opens on the wrong monitor

In Apollo, inspect:

```text
Audio/Video
→ Advanced Display Device Options
```

Apollo includes a mode that can activate the virtual display and make it primary for the session.

Exact wording can change between versions, so use the description shown in your installed build.

---

# 17. Black screen / game launch problems

If Artemis connects but the picture is black:

1. test **Desktop**;
2. confirm Windows sees the virtual display;
3. inspect Apollo logs;
4. try H.264;
5. disable HDR;
6. try 1080p60;
7. update the GPU driver;
8. disable third-party overlays temporarily;
9. test borderless fullscreen.

Borderless fullscreen is often easier to diagnose than exclusive fullscreen.

---

## Game works locally but not in stream

Check:

- game monitor selection;
- Windows primary display;
- Apollo display-device configuration;
- remembered game resolution;
- HDR state;
- launcher behaviour;
- capture restrictions/anti-cheat.

---

# 18. Network troubleshooting

## Find host IPv4 address

On Windows:

```powershell
ipconfig
```

Look at the active Ethernet/Wi-Fi adapter.

Example:

```text
192.168.1.x
```

---

## Ping test

From another LAN computer/device:

```bash
ping 192.168.1.x
```

Look for:

- low latency;
- consistent latency;
- no packet loss.

Consistency matters more than one unusually low result.

---

## Wi-Fi tuning

Prefer:

- 5 GHz / 6 GHz;
- strong signal;
- uncongested channel;
- client reasonably close to router/AP.

Avoid testing while:

- another device is saturating Wi-Fi;
- large downloads are running;
- the TV has weak signal;
- both host and client are on poor Wi-Fi.

---

# 19. My baseline configuration

For my current setup, I start here:

```text
HOST
Windows PC
GTX 1650 Super 4 GB
32 GB RAM
Apollo
NVENC hardware encoding

CLIENT
TCL C6K 55"
Google TV / Android TV
Artemis
TV Game Mode enabled

STREAM
1920×1080
60 FPS
HEVC
20–35 Mbps
HDR off

NETWORK
Host wired where possible
Client wired or strong 5 GHz Wi-Fi
```

Then increase one variable at a time:

```text
1080p60
   ↓
1440p60
   ↓
higher bitrate
   ↓
4K60
   ↓
HDR
```

This makes it much easier to identify the exact point where smoothness gets worse.

---

## Quick diagnosis table

| Symptom | First test |
|---|---|
| Random stutter | Lower bitrate |
| Consistently rough motion | Check decode time/frame pacing |
| High input latency | Game Mode + network |
| Black screen | Desktop + display selection |
| HEVC bad / H.264 good | Client HEVC path |
| 1080p good / 4K bad | Client/network load |
| Controller missing | Apollo input permissions |
| No virtual display | SudoVDA |
| Mirrored display | Win + P → Extend |
| Cannot launch app | Launch Apps permission |
| TV bad / another client good | TV SoC/decoder path |

---

# 20. Credits

### Apollo

- Repository: https://github.com/ClassicOldSong/Apollo
- Releases: https://github.com/ClassicOldSong/Apollo/releases
- Wiki: https://github.com/ClassicOldSong/Apollo/wiki

### Artemis

- Repository: https://github.com/ClassicOldSong/moonlight-android

### Related upstream projects

- Sunshine: https://github.com/LizardByte/Sunshine
- Moonlight: https://github.com/moonlight-stream

All software belongs to its respective authors and contributors.

This repository is only a user-written setup/troubleshooting guide.

---

## Contributing

Useful performance reports should include:

```text
Host GPU:
Host OS:
Apollo version:
Client:
Artemis version:
Ethernet/Wi-Fi:
Resolution:
FPS:
Codec:
Bitrate:
HDR:
Network latency:
Decode time:
Problem:
```

---

## Final checklist

### Host

- [ ] Apollo installed
- [ ] Web UI opens
- [ ] SudoVDA installed
- [ ] Hardware encoder detected
- [ ] Desktop entry works
- [ ] Games added
- [ ] Firewall allows Apollo

### Client

- [ ] Artemis installed
- [ ] Host discovered/added
- [ ] Pairing complete
- [ ] Permissions correct
- [ ] Controller works
- [ ] Performance overlay tested

### Stream

- [ ] 1080p60 tested first
- [ ] HEVC tested
- [ ] H.264 tested as fallback
- [ ] Bitrate tuned
- [ ] Game Mode enabled
- [ ] SDR stable before HDR
- [ ] Higher resolutions tested incrementally

---

**Maintained by [ish4ra](https://github.com/ish4ra)**

If this guide helped you, consider starring the repository.
