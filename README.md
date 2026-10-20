<div align="center">
  <h1>🎮 4XCLOUD (PSBox for PS4)</h1>
  <p><b>Play your Xbox Game Pass games natively on your jailbroken PlayStation 4.</b></p>

  <!-- Remplace les liens ci-dessous par tes propres badges quand tu auras des releases -->
  <img src="https://img.shields.io/badge/Platform-PS4%20(Orbis)-003791?style=for-the-badge&logo=playstation" alt="Platform">
  <img src="https://img.shields.io/badge/License-GPL--3.0-blue?style=for-the-badge" alt="License">
  <img src="https://img.shields.io/badge/Status-Work%20in%20Progress-orange?style=for-the-badge" alt="Status">
</div>

---

## 📖 Overview

**4XCLOUD** is a fully native homebrew application that brings the Xbox Cloud Gaming (xCloud) experience to the PlayStation 4. 

Unlike simple web wrappers, 4XCLOUD is built from the ground up in C++. It implements the entire WebRTC stack (ICE, DTLS, SRTP, SCTP) required to negotiate a session with Microsoft's servers, extracts the H.264 video stream, and renders it directly to the PS4's framebuffer using SIMD-accelerated YUV conversion. 

Beyond streaming, 4XCLOUD aims to be a **modern launcher** for your console, unifying your local PS4 games and your Xbox Cloud library in a single, sleek, 60fps interface.

> ⚠️ **Disclaimer:** This is an educational homebrew project. You need a valid Xbox Game Pass Ultimate subscription to use it. This project is not affiliated with, endorsed by, or connected to Microsoft or Sony.

---

## ✨ Features

- **Native xCloud Support:** Full WebRTC implementation handling OAuth device-code authentication, SDP negotiation, and media streams.
- **Hardware/CPU Decoding:** H.264 video decoding with optimized, SIMD-accelerated YUV conversion for smooth 1080p/60fps playback.
- **Modern Launcher UI:** A complete replacement for the PS4 home menu (WIP), allowing you to browse your PS4 games and xCloud games in one unified, dark-themed interface.
- **Background Audio:** Music player integration (WIP) with background playback and `.mp4` audio extraction.
- **Controller Support:** Native DualShock 4 input forwarding via WebRTC Data Channels.

---

## 🛠️ The Technical Journey (Why this is hard)

Porting WebRTC and modern video playback to **Orbis** (the PS4 OS) is a nightmare. The OpenOrbis toolchain and the console's libc present several undocumented pitfalls. This project required solving major platform-specific issues to prevent silent crashes:

- **usrsctp ABI Conflicts:** Fixed `sockaddr_conn` struct layout mismatches between the public header (Linux) and internal headers (BSD). The family was at offset 0 instead of offset 1, causing `EAFNOSUPPORT` (errno 47) on bind.
- **Silent Crashes (Stack Overflows):** The PS4 allocates only **64 KiB of stack** to library threads. We patched `libjuice`, `usrsctp`, and FFmpeg to enforce larger stacks (1 MiB+) and avoid silent process deaths. (e.g., `SctpTransport::doRecv` had a 65KB stack buffer).
- **errno Mismatches:** The precompiled `libc++` expects Linux `ETIMEDOUT` (110), but Orbis returns FreeBSD `ETIMEDOUT` (60). We wrote a `pthread_cond_timedwait` wrapper to prevent `std::condition_variable` from throwing fatal exceptions.
- **Static Initialization Deadlocks:** Patched the `__cxa_guard` to avoid thread ID collisions (due to `syscall(20)` differences) during static initialization.
- **YUV Conversion Optimization:** The initial software rendering was taking 173ms per frame. We optimized it using SSE SIMD, fixed the 3:2 chroma subsampling, and added row caching, reducing the cost by **9.3×**.

*(For a deep dive into these bugs, check out the `docs/ps4-port.md` file in this repository.)*

---

## 📦 Installation

1. Download the latest `.pkg` file from the [Releases](https://github.com/yourusername/4XCLOUD/releases) page.
2. Transfer the `.pkg` to your PS4 (via USB or FTP) in `/data/pkg/`.
3. Install it using your preferred package installer (e.g., GoldHEN).
4. Launch the app, log in with your Microsoft account (via device code), and start playing!

---

## 🎮 Roadmap

- [x] Native WebRTC stack (ICE/SCTP/DTLS).
- [x] Video decoding and rendering (1080p 60fps).
- [x] Controller input forwarding.
- [ ] Hardware video decoding (`libSceVideodec2`) for lower CPU usage.
- [ ] Local PS4 game launcher integration.
- [ ] Background music player (`.mp4` audio extraction).
- [ ] Custom themes and UI enhancements.

---

## 🏗️ Building from source

**Requirements:**
- [OpenOrbis PS4 Toolchain](https://github.com/OpenOrbis/OpenOrbis-PS4-Toolchain)
- CMake (>= 3.15), Clang, LLD
- Python 3 (for packaging scripts)

**Build Instructions:**
```bash
# Clone the repository
git clone https://github.com/yourusername/4XCLOUD.git
cd 4XCLOUD

# Create a build directory
mkdir build && cd build

# Configure with CMake (ensure OO_PS4_TOOLCHAIN is set in your env)
cmake ..

# Build
make -j$(nproc)

# Package into a .pkg (see tools/ps4/package.sh)
cd ..
./tools/ps4/package.sh
