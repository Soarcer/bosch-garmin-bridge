# Bosch E-bike to Garmin Bridge

A reverse-engineered solution for connecting Bosch e-bike systems to Garmin cycling computers.

## Overview

This project enables communication between Bosch e-bike motors and Garmin devices by reverse engineering the Bluetooth protocol used by the Bosch Flow app. Data is captured from paired Bosch motors and retransmitted to Garmin devices via Bluetooth peripheral emulation for standard cycling metrics or Connect IQ custom data fields for e-bike specific information.

## The Challenge

Bosch motors use a proprietary protocol to communicate with their Flow app, without exposing standard APIs or common Bluetooth device protocols. This makes direct integration with third-party devices like Garmin computers impossible.

### Pairing Complexity

The system implements a secret key exchange during the pairing handshake between the app and Bosch motor. Until this pairing process is better understood and the key is extracted, we are currently limited to using a phone previously paired using the Bosch app.

**Current Limitations:**
- Cannot use a simple ESP32 or Arduino solution as a proxy for messaging
- Requires a device that has been previously paired with the Bosch Flow app

**The Silver Lining:**
Once a phone has been paired by running the Bosch Flow app, we can run Bluetooth sniffers like NRF Connect on that same phone to capture raw Bluetooth traces for analysis.

## Solution Architecture

This project implements a mobile app that acts as a bridge between the Bosch motor and Garmin device on a previously paired phone. The app connects to the Bosch motor and parses Bluetooth data, then relays it to the Garmin device.

### Data Types Supported

#### 1. Native Garmin Fields
**Speed, cadence, power** - Since Garmin devices can support external Bluetooth devices providing this information, the app needs to present a Bluetooth peripheral that can be paired with the Garmin device and pass along the relevant data.

- **Benefits:** This data will show up on the relevant native data fields and will also be stored in the activity log file (.FIT, .TCX etc.)
- **Tradeoffs:** Having the phone present itself as a Bluetooth peripheral significantly increases app complexity and reduces the types of devices that can be used

#### 2. Custom Data Fields  
**Additional fields like assist level and battery percentage** can make use of the custom data field + companion app capabilities to provide data from the phone to the Garmin device and display and store them in the activity record.

- **Benefits:** Access to Bosch-specific data not available in standard cycling protocols
- **Tradeoffs:** This data will need a separate viewer to be properly visualized
- **Reference:** [Garmin Connect IQ Data Fields Guidelines](https://developer.garmin.com/connect-iq/user-experience-guidelines/data-fields/)

## Protocol Analysis

### Bluetooth Service Details
All relevant real-time updates seem to be transmitted by:
- **Primary Service:** `00000010-eaa2-11e9-81b4-2a2ae2dbcce4`
- **Data Characteristic:** `00000011-eaa2-11e9-81b4-2a2ae2dbcce4`

### Decoded Data Patterns

#### 1. Assist Level Value (High Confidence)
- **Data structure:** `30-04-98-09-08-XX` where XX indicates assist level from 0-4

#### 2. Assist Level ASCII Names (Medium Confidence)
- **Data structure:** `30-22-18-0D-C0-80-12-0A-03-[MODE_NAMES]` where MODE_NAMES are equivalent to ("OFF"/"ECO"/"TOUR"/"eMTB"/"TURBO") or whatever user has configured
- **Validation:** eMTB user @robbydobs has found the same pattern

#### 3. Power Levels (Low Confidence)
- **Data structure:** `A2-43-XX-YY-ZZ`

**Structure breakdown:**
- `A2` = Sensor data prefix
- `43` = Power sensor type identifier  
- `XX` = Constant byte (08)
- `YY` = Power value in watts
- `ZZ` = Additional data

**Note:** More reliable testing data is needed. The ZZ values may help provide power levels beyond the maximum of 255 watts we can get from the YY values alone.

#### 4. Battery Level 
- **Priority:** Medium
- **Status:** Not yet investigated

#### 5. Cadence
- **Priority:** Low  
- **Status:** Not yet investigated

## Prerequisites
- Android/iOS device previously paired with Bosch Flow app
- Garmin cycling computer with Connect IQ support
- Bluetooth sniffing capabilities (NRF Connect or similar)

## Current Limitations
- Requires pre-paired device (cannot use standalone ESP32/Arduino)
- Power data needs further validation and testing
- Battery and cadence protocols not yet decoded
- Phone must present itself as Bluetooth peripheral for native Garmin fields

## Contributing

We welcome contributions in the following areas:
- **Protocol analysis findings** - Help decode remaining data patterns
- **Testing data** - Capture traces from different Bosch motor models
- **Code contributions** - Mobile app development for the bridge solution
- **Documentation** - Improve protocol documentation and setup guides

## Roadmap

1. **Phase 1:** Complete power protocol analysis and validation
2. **Phase 2:** Decode battery level and cadence data patterns
3. **Phase 3:** Develop mobile bridge application
4. **Phase 4:** Create Garmin Connect IQ data field and companion app
5. **Phase 5:** Investigate pairing key extraction for standalone solutions

## Acknowledgments
- @robbydobs for eMTB mode name confirmation and validation
