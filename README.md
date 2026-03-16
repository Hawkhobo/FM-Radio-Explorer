---
layout: single
title: "FM Radio Explorer"
permalink: /
author_profile: true
header:
  overlay_color: "#000"
  overlay_filter: "0.5"
  overlay_image: /assets/images/header-radio.jpg # Add a header image if you have one!
excerpt: "An embedded systems prototype integrating the TI CC3200, Last.fm API, and an FM Stereo Radio Module."
toc: true
toc_sticky: true
---

<style>
  details {
    margin-bottom: 1rem;
    border: 1px solid #333;
    padding: 0.5rem 1rem;
    border-radius: 4px;
  }
  summary {
    outline: none;
  }
  details[open] summary {
    border-bottom: 1px solid #444;
    margin-bottom: 0.5rem;
    padding-bottom: 0.5rem;
  }
</style>

# FM Radio Explorer
**Developed by: Jacob Feenstra & Chun-Ho Chen**

[![View Demo](https://img.shields.io/badge/Demo-Video-red?style=for-the-badge&logo=google-drive)](https://drive.google.com/file/d/109MXLAsd9JzWsAcxi5jvkkKnfmKOH1J9/view?usp=sharing)

The FM Radio Explorer is a hardware-software prototype that offers an exploratory music listening experience by bridging traditional FM signals with modern web metadata.

---
<details>
  <summary style="font-size: 1.5em; font-weight: bold; cursor: pointer;">
   Description 
  </summary>
The FM Radio Explorer has changed due to time & hardware constraints from it's initial proposal, but it still offers an exploratory music listening experience. In its current stage, it is a prototype with real capability to become a production-value system. Last.fm's public-facing API is still used to query metadata for a particular song, and the radio module is capable of tuning into FM radio signals and performing playback. Unfortunately, working with the RDA5870M radio module model proved more difficult than we anticipated, and we were not able to successfully solder it to the rest of the system. The replacement radio module we opted for does not support the Radio Data System protocol (RDS), which is crucial for being able to display information for the currently playing song. 

The current prototype is as follows: Last.fm offers a wide variety of metadata and points of musical exploration, a subset of which is displayed for a song of our choice, by querying the API endpoints with the track and artist name. The S10-S3 Univeral Remote and IR Receiver is configured for numerical & punctuation input, and writes to the radio module and plays a selected FM broadband (for example, 90.3 would correlate to 90.3 FM). The same remote will be used to switch between different OLED Display views, each of which displays different output from the Last.fm API (to be discussed in Section [sec:Design]). An antenna boosts signal gain, and a cheap 3.5 mm auxiliary headset can plug directly into the headphone jack of the radio module. The API developed for the OLED UI, paired with the IR Receiver code, enables the user to switch between different FM radios seamlessly. Note all of this is orchestrated with our Texas Instrument's CC3200 LaunchPad.
</details>

## System Overview
The current prototype leverages the **TI CC3200 Launchpad** and an **Arduino Nano** to coordinate between physical radio signals and cloud-based metadata.

### Core Components
| Component | Function |
|:---|:---|
| **TEA5767 FM Chip** | Handles FM signal tuning and audio playback. |
| **Last.fm API** | Fetches track metadata and artist biographies. |
| **SSD1351 OLED** | 128x128 SPI-driven display for the UI. |
| **S10-S3 Remote** | IR-based user input for frequency and navigation. |

---

## Architecture & Design
The system logic is divided between hardware-specific drivers and high-level UI management.

### System Flow
![System Architecture](report/state_diagram.png)
*Architecture of the functional specification and hardware abstraction.*

### 1. The View System (OLED API)
The firmware (specifically `oled_ui.c`) manages seven distinct views:
* **Radio View:** Current frequency and track progression.
* **Album Cover:** 118×118 JPEG rendering using `stb_image` for bilinear interpolation.
* **Synced Lyrics:** Uses LRClib timestamps synced via a system GP Timer.
* **Discovery:** Similar artists and genre tags fetched via HTTP GET.

### 2. Universal Remote Control
Input is handled via an IR Receiver using the **RC-6 protocol** (or similar logical mapping):
* **Numerical (0-9):** Specify frequencies (e.g., `90.3`).
* **Arrows:** Navigation through the "Banner Menu" and vertical text scrolling.
* **Mute/Last:** Controls for the TEA5767 driver.

---

## Implementation Details

### The Text Engine
To handle biographies and lyrics, we built a custom text engine on top of **Adafruit GFX**. It handles:
* **Soft Wrapping:** Prevents word-break mid-line.
* **Scrolling Indicators:** Magenta markers at screen corners indicate more content.
* **Optimization:** Only lines within the visible window are rendered to save cycles.

### Album Art Pipeline
The **CC3200** has limited RAM, making JPEG decompression difficult. 
> **Technical Note:** We implemented a static pool allocator for `stb_image` to avoid heap fragmentation during the 8,000+ lines of decompression logic.

---

## Challenges & Hardware Lessons
* **The "Progressive" JPEG Problem:** Last.fm's CDN serves progressive JPEGs, which many baseline libraries (TJpegDec) fail to decode. Switching to `stb_image` solved this.
* **I2C Voltage Mismatch:** The TEA5767 module forced 5V logic on I2C lines, requiring a hardware bridge (Arduino + Voltage Divider) to protect the TI-Launchpad's 3.3V pins.
* **The SMD Struggle:** Low-quality pads on early RRD-102 modules lifted during soldering, leading to the switch to the plug-and-play TEA5767 package.

---

## Future Work
* **RDS Support:** Integrating metadata directly from the FM carrier.
* **Potentiometer Integration:** Analog volume control via amplifier gain.
* **Session History:** Using RAM to store a history of visited signals.
