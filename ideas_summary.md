# Home Automation Project — Design Notes

_Last updated: 9 November 2026_

## 1. Overall direction

The project is aimed at building a flexible, locally controlled home automation system, with an emphasis on:

- **Local operation:** avoid cloud-dependent devices and services wherever practical.
- **Open standards and open-source software:** prefer solutions that can be inspected, modified and maintained.
- **Edge processing:** perform work on individual devices where it is practical, rather than unnecessarily relying on a central server.
- **Low cost:** reuse existing hardware and avoid buying expensive equipment where a simpler solution will do.
- **Incremental development:** investigate and acquire components ahead of implementation, then build and test individual projects as time permits.

Home Assistant is expected to become the central automation platform, with a Linux machine providing supporting services.

---

## 2. Home Assistant platform

### Intended setup

Home Assistant has not yet been commissioned.

Possible hardware:
- Raspberry Pi 5.
- An existing laptop or other Linux machine.

The Linux environment being considered is Ubuntu Server 26.04.1, installed with a minimal CLI configuration. A lightweight desktop can be started manually with `startx` when needed.

The intention is to keep the underlying system relatively simple rather than building a complicated infrastructure around Home Assistant.

### General architecture

Home Assistant will provide:
- Central automation and device coordination.
- Zigbee device integration.
- Integration with locally hosted services.
- A common interface for controlling and monitoring devices.

Individual ESP32 devices should retain useful local functionality wherever practical, particularly when network connectivity or Home Assistant is unavailable.

---

## 3. Zigbee network

### General approach

Zigbee is the preferred wireless protocol for many low-power sensors and control devices.

The intention is to keep the Zigbee network local and avoid devices that require a manufacturer's cloud service.

### Coordinator options investigated

Several coordinator options have been considered:

| Coordinator | Approximate price discussed | Notes |
|---|---:|---|
| Home Assistant Connect ZBT-2 | $104.95 | Official Home Assistant option |
| Argon40 Pro / CC2652P | $52.90 | Lower-cost option |
| ZigStar UZG-01 | $96.25 | Ethernet/PoE coordinator |

The ZigStar UZG-01 is particularly interesting because it can be located away from the Home Assistant host using Ethernet and PoE.

A USB coordinator connected directly to a Raspberry Pi is simpler, but may be less convenient for placement in a double-brick house.

### PoE infrastructure

A small PoE switch has been considered to power the coordinator and potentially other network devices.

Switch options discussed included:
- TP-Link TL-SG1008P.
- Ubiquiti alternatives.

The target is a modest 5–8 port 802.3af/at switch.

### Zigbee coverage

The house is a 1970s double-brick Perth house, with internal brick walls that may attenuate radio signals.

Coordinator placement and the use of powered Zigbee routers will therefore matter.

Possible coordinator locations include:
- A suitable central position.
- A location connected by Ethernet and PoE.
- A position away from the router and other sources of interference.

A concealed installation is desirable, but radio performance and temperature must take priority.

---

## 4. Home network

### Existing equipment

The current network uses a Billion BiPAC 8700 VAXL modem/router with Aussie Broadband FTTN.

An older router has been repurposed as a wired switch and Wi-Fi access point, with its routing functions disabled.

The two access points have been configured with the same SSID, but the older unit only supports 2.4 GHz while the Billion supports both 2.4 and 5 GHz.

The desired replacement arrangement should provide:
- Wired Ethernet ports.
- Two Wi-Fi access points.
- Reliable roaming between access points.
- Clean handoff as devices move around the house.

### DNS

An internal DNS arrangement using `home.arpa` is being considered.

The intended approach is:
- Local devices resolve internal hostnames through a home DNS service.
- The router distributes the home DNS server to clients.
- External DNS queries are forwarded upstream.
- Local hostnames remain independent of external DNS services.

### IPv6 and mDNS

IPv6 has been enabled on the LAN.

Observed behaviour:
- Android devices obtain IPv6 addresses.
- IPv6 connectivity generally works.
- Two Ubuntu 26.04 devices can SSH to each other using IPv6 addresses.
- `ssh hostname.local` resolves and works over IPv4.
- `ssh -6 hostname.local` fails.

This suggests an mDNS/IPv6 resolution issue worth investigating.

There has also been intermittent failure loading some embedded web content, such as BlueSky posts, while connected to the LAN. Disabling Wi-Fi and reloading fixes the problem, even though the device remains capable of using IPv6 over another connection.

The router's IPv6 implementation is a possible cause. Its firmware/software version still needs checking.

---

## 5. Voice-controlled kitchen timer

This is one of the more developed device concepts.

### Intended functionality

A small voice-controlled kitchen device should support commands such as:

- "Set a timer for ten minutes for the potatoes."
- "Give the cauliflower another five minutes."
- "How long have the potatoes got?"
- "Cancel the chicken timer."
- "What timers are running?"

Multiple timers should run concurrently.

When a timer expires, the device should:
- Sound a buzzer or audible alarm.
- Announce the timer label, for example, "Potato timer done".
- Update its display.

An OLED display is planned, initially showing at least one timer. Cycling through multiple timers is a possible enhancement.

The speech system needs to understand general cooking vocabulary rather than relying on a fixed list of food names.

### Hardware

The intended microcontroller is an **ESP32-S3-N16R8**.

Potential audio components:

| Component | Purpose |
|---|---|
| INMP441 | I²S MEMS microphone |
| MAX98357A | I²S class-D speaker amplifier |
| Small 4–8 Ω speaker | Audio output |
| OLED | Timer display |
| Buzzer | Timer alarm |
| Button | Initial wake-up |
| Optional RGB LED | Device status |

