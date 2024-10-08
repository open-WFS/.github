# Open Wave Field Synthesis

Open Wave Field Synthesis (OpenWFS) is an open-source system for [Wave Field Synthesis](https://en.wikipedia.org/wiki/Wave_field_synthesis), acoustic beamforming, and other spatial rendering techniques. An OpenWFS system comprises of multiple playback modules, each of which is a 1m-long unit comprising 32 full-range drivers, connected over the [AVB](https://en.wikipedia.org/wiki/Audio_Video_Bridging) audio networking protocol.

The OpenWFS system comprises:

 - [Hardware schematics](https://github.com/open-WFS/open-WFS-hw) and [firmware](https://github.com/open-WFS/open-WFS-fw) to built an audio playback module, comprising of 32 full-range drivers (pending publication)
 - [Software spatialiser and utilities](https://github.com/open-WFS/open-WFS-software) that receives mono audio inputs over an audio loopback interface, and distributes them appropriately over the OpenWFS system

 The hardware system can also be used as an N-channel audio interface, with each driver addressable separately, for protyping and evaluation of arbitrary spatial panning algorithms.

## Requirements

 To create a complete OpenWFS system, you will require:

  - one or more OpenWFS modules, built following the schematics and firmware linked above
  - an AVB-compliant network switch, such as a [Netgear GS7xxT](https://support.biamp.com/Tesira/AVB/Installing_an_AVB_license_on_Netgear_GS7xxT_switches) with AVB firmware (cheapest but requires some manual config), [MOTU AVB Switch](https://motu.com/en-us/products/avb/avb-switch/) (mid-priced but only has 6 ports, i.e. 160 channels), or a [Netgear Pro AV switch](https://www.netgear.com/uk/business/solutions/av-over-ip/) (the best solution but most expensive).
  - an AVB audio interface, such as an [RME Digiface AVB](https://rme-audio.de/digiface-avb.html) or [Ultralite AVB](https://motu.com/products/avb/ultralite-avb), or a computer that supports AVB natively such as an Apple Silicon Mac Mini / Mac Studio.
    - Alternatively, you can interface with AVB on any modern macOS laptop if you [daisy-chain a USB-C to Thunderbolt 2 adapter with a Thunderbolt 2 to Ethernet adapter](https://discussions.apple.com/thread/254441948?sortBy=rank).
  - a ethernet cable (CAT5/CAT6) for each module, and an additional cable to connect the AVB host computer/interface to the audio network

You will also need the following software:

 - Reaper, to run the default soundcheck sessions. Reaper can be used as a free download.
 - A spatialiser system. This can be one of:
   - [Spat](https://forum.ircam.fr/projects/detail/spat/) (recommended): a mature, sophisticated and configurable spatial panning framework from IRCAM. Requires Max/MSP.
   - [OpenWFS Python panner](https://github.com/open-WFS/open-WFS-software/): a novel, open-source WFS/beamforming spatialiser created for this project. Requires Python and Node. Not yet recommended as Spat is more fully-featured and perceptually superior.
   - [WFSCollider](https://sourceforge.net/projects/wfscollider/): An open-source WFS framework for SuperCollider. Not yet evaluated with OpenWFS.

The [open-WFS-software](https://github.com/open-WFS/open-WFS-software/) repo also includes some utilities for working with OpenWFS, including:

 - a soundcheck procedure
 - a utility to export OpenWFS topology config files for Spat

## Setup

This guide will assume you are connecting 4 OpenWFS modules, for a total of 128 drivers.

### 1. Configure the AVB interface and switch

Power on the computer and AVB switch.

Note that you may need to configure the switch for AVB ([Instructions for Netgear M4250](https://support.biamp.com/Tesira/AVB/Netgear_M4250_-_Enabling_AVB_using_the_AV_user_interface))

If you are using an AVB interface, connect it to your computer via USB, and connect it to the AVB switch via a standard ethernet cable.

If you are connecting directly to a Mac with integrated AVB support, connect your Mac to the AVB switch via a standard ethernet cable.

### 2. Connect the AVB modules

Connect each of the OpenWFS modules to the AVB switch, and plug them in to power via their 24V barrel connectors. The green LED on each OpenWFS module should illuminate.

The modules will now advertise their presence, and will become visible to the AVB host computer.

### 3. Configure the AVB streams

Each AVB stream carries 8 channels of audio, and must be routed to the respective OpenWFS module. As each module plays 32 channels, this means that four streams will be routed to each module. Streams are numbered consecutively.

#### Using native macOS AVB config

If you are connecting an Apple Mac directly to the AVB network:

 - Open **Audio MIDI Setup** and select **Window > Network Device Browser**.
 - You should see 4 `openWFS` AVB devices appear in the browser.
 - For each of the devices, configure the input streams, assigning four streams (each carrying eight channels) to each module consecutively
 - Finally, create an aggregate device that collates all of the input streams.

#### Using an RME Digiface or other RME AVB device

Open **RME AVB Controller**. You should see an entry for the Digiface, followed by 4 `openWFS` AVB devices. 

For each of the devices, configure the input streams with four consecutive values. For example:

 - The first `openWFS` device in the list should be configured to accept streams 1, 2, 3, 4
- The second `openWFS` device should be configured to accept streams 5, 6, 7, 8
- The third `openWFS` device should be configured to accept streams 9, 10, 11, 12
- The fourth `openWFS` device should be configured to accept streams 13, 14, 15, 16

#### Ensuring that the stream ordering is consecutive

Whether using native macOS AVB or RME, it makes life easier to ensure that the boxes are physically ordered with sequential channel numbers -- for example, in the case of a linear row of four, such that the left-most module is channels 1..4, the subsequent module is 5..8, etc. However, it is not possibly a priori to identify which physical module corresponds to which virtual `openWFS` device in the AVB manager.

To address this, once the AVB modules have been set up for the first time, we recommend opening and playing the [Reaper Broadcast 1-8](https://github.com/open-WFS/open-WFS-examples) session, which will play the numbers 1 through 8 throught sequential sets of 32 channels -- that is, each module should be heard reading out a single number.

When the set is playing, you can then dynamically re-order the routing in the AVB controller (as described in the previous section), shuffling the arrangement around until the left-most module corresponds to "one", then "two", etc.

Having done this, you can later use the default spatial configurations without having to specify the coordinates of each module.

### 4. Configure the software systems and spatial panner

You're now ready to start setting up the WFS software systems and spatialiser following the docs in the [open-WFS-software](https://github.com/open-WFS/open-WFS-software/) repo.

