Covending Machine: Open-Source Distributed COVID-19 Test Dispensing System


**License: MIT License – Open for community contributions and iterations**  
**Overview**: This document provides one possible path for building and deploying vending machines capable of dispensing a public health tool in the form of rapid antigen tests for COVID-19. Hereafter we refer to this public health innovation as a Covending Machine. It is aimed at integrating hardware, software, and operational details for rapid prototyping. Wherever possible, components are low-cost, open-source compatible, and sourced from US vendors to minimize build friction. Again, this innovation is designed to serve as a public health tool and is not a substitute for FDA authorized and/or approved, or CLIA authorized and/or approved diagnostics.

---

### Executive Technical Summary

The Covending Machine system is here proposed as an innovative, open-source, decentralized platform for dispensing free rapid antigen COVID-19 tests as a public health tool. Conceived in 2020 and open-sourced for community iteration, this system attempts to addresses, however inadequate, the need for frictionless access to tests while generating anonymized epidemiological data to identify potential transmission hotspots. Each machine could function as a node in a mesh network, enabling inter-machine communication for inventory alerts and data aggregation to a central cloud platform. The design prioritizes speed of implementation, low cost, and open-source compatibility, using commercially available components to facilitate rapid prototyping by makers and deployment in suitable environments.

**System Design Overview**: The hardware is built around a Raspberry Pi 4 as the core microcontroller for its ease of use, GPIO expandability, and community support. Dispensing uses a belt conveyor mechanism to handle fragile test kits (estimate: 6x3x1 inches per kit, or boxed 9x6.5x3.25 inches for 25 kits), with capacity for 200-300 kits across multiple slots. Power is solar-assisted (10W panel with battery backup) for outdoor sustainability, enclosed in a weatherproof steel cabinet. User interaction is intended to be frictionless: scan a QR code via a mobile app to select quantity, then dispense via Bluetooth Low Energy (BLE) or SMS. Sensors monitor inventory, temperature (to maintain 2-30°C for test efficacy), and security. Networking includes WiFi/4G for cloud uplink, LoRa-based mesh for inter-machine resilience, and anonymous data transmission.

**Key Technical Features**:
- **Hardware**: Raspberry Pi 4, stepper motors for dispensing, ultrasonic sensors for inventory, DHT22 for environmental monitoring, BLE module (ESP32 add-on), cellular modem for SMS.
- **Software**: Firmware in Python (using libraries like RPi.GPIO, bleak for BLE), cross-platform Flutter app for QR/BLE/SMS, OpenRemote cloud for fleet management and analytics.
- **Data and Analytics**: Anonymized timestamps and geolocation data stored in a time-series database (InfluxDB), with anomaly detection algorithms (e.g., using something like Pandas/Scikit-learn) to flag usage spikes. Privacy adheres to HIPAA-like anonymization, with no personal identifiers collected.
- **Deployment**: Outdoor-focused, ADA-compliant, with site selection based on demographics and accessibility. Regulatory framing as a public health tool, not a diagnostic device. It is not intended to replace FDA authorized and/or approved diagnostic testing or CLIA certified and authorized testing. This is solely intended as a public health tool and not a registered diagnostic. 

**Feasibility and Validation**: Components may be sourced from US vendors like Adafruit, Digi-Key, and Amazon for low-cost accessibility (<$500/unit prototype). Design draws from existing open-source projects (e.g., Intel's Intelligent Vending Machine on GitHub, Instructables Arduino Vending) adapted for medical kits. Stress testing includes 1000 dispense cycles, environmental exposure (-10°C to 40°C), and FCC/UL compliance checks. Scalability from pilot (1-10 machines) to citywide (100+) may be achievable via mesh networking and cloud auto-scaling.

**Economic and Sustainability**: With uncertainty related to viral transmission, second and third order effects impacting source materials, supplies and logistics, it is impossible to state with anything like certainty what the exact cost and ROI might be. However, we can share an estimate that will, of course, change as circumstances change. It may be possible to build the prototype BOM ~$450, with scaling to <$300/unit at volume. Sustainability via solar power could reduce operational costs. There may also be available funding through grants or partnerships that could defray the cost. Potential ROI via public health benefits: anonymity may reduce stigma and therefore increase willingness to test. Due to the rapid antigen tests being free and accessible to the end-user, this may also increase willingness, frequency, and intensity of use. Due to increased testing volume, even if rapid tests were found not to be equivalent to "Gold Standard" PCRs (*see comment below*), it is still conceivable that a person testing more regularly with rapid tests could derive benefit by detecting early signals of infection. As a public health tool, earlier detection of possible infection could help people seek appropriate testing and care via licensed doctors. Earlier signal detection and confirmation via FDA & CLIA authorized and/or approved testing could help reduce transmission by interrupting chains of transmission. This would prove especially helpful if testing is carried out regularly, even among "asymptomatic" individuals. *Keep in mind that current "Gold Standard" PCR testing takes some time to return results. It is therefore conceivable that if a person were to wait until symptomatic to seek out PCR testing, the patient may be required to wait for results before receiving treatment*. 

While this document contains certain technical specifications and speculations, it is no way intended to serve as, suggest, imply, replace in whole or in part, any medical guidance, advice, or public health messaging. This is an open-source project related to building *Covending machines* under MIT license for community contributions.

---

### Possible System Architecture with Diagrams and Schematics

The architecture is modular: hardware layer for physical dispensing, embedded control for local operations, networking for connectivity, software for user/app interaction, and cloud for analytics/fleet management.

**High-Level System Diagram** (Text Description; imagine a block diagram):
- User Phone/App → QR Scan → BLE/SMS Command → Covending Machine (RPi Core) → Dispense Motor → Test Kit Output.
- Machine Sensors → RPi → Local DB → Mesh/Cloud Uplink → Analytics Dashboard.
- Multiple Machines ↔ Mesh Network (LoRa) → Alerts (Low Inventory).

ASCII Art Representation:
```
[User Phone/App (Flutter)] --QR/BLE/SMS--> [Covending Machine Enclosure]
                                       |
                                       v
[Raspberry Pi 4] <--Sensors (Inventory, Temp)--> [Stepper Motors + Belt Conveyor]
 |    ^                                         |
 |    |                                         v
[Local SQLite DB] <--> [Firmware (Python)] --> [Dispensed Test Kits]
 |
 v
[Networking: BLE Module, Cellular Modem, LoRa for Mesh] --> [OpenRemote Cloud]
                                                            |
                                                            v
[Analytics Engine (InfluxDB + Algorithms)] --> [Dashboard + Alerts]
```

**Hardware Architecture**:
- Core: Raspberry Pi 4 Model B (4GB RAM) controls all operations.
- Dispensing: Belt conveyor (adapted from existing designs) with 3-5 slots, each holding 50-100 kits. Stepper motors (NEMA 17) drive belts gently to avoid damaging fragile nasal swab kits.
- Sensors: HC-SR04 ultrasonic for inventory per slot, DHT22 for temp/humidity, PIR for tamper detection.
- Power: 10W solar panel → MPPT controller → 12V Li-ion battery (3x18650) → 5V/12V regulators. Integrate a 12V solar setup with MPPT controller; add a buck converter (LM2596) for 5V output to Pi/sensors. Total consumption: ~10W idle, 25W during dispense (steppers peak at 1.7A).
- Display: 7" LCD touchscreen for status/QRs.
- Security: Electromagnetic lock, with potential for webcam installation for surveillance.

**Dispensing Mechanism Details**: The belt conveyor is refined with details from a proven patent design for endless conveyor belts in vending machines. This ensures minimal drop impact (e.g., shelves move in a controlled oval path, reducing acceleration on kits). Key mechanical details:
- Belt Structure: Composed of identical, molded polypropylene elements (H-shaped with shelves). Each element has dowel pins and tubular rollers for smooth, bearing-like motion. Shelves are spaced 4-6 inches apart to hold 1-5 kits per slot (capacity: 200-300 kits total across 3-5 belts).
- Path: Vertically elongated oval track (raceway ~1 inch wide) with inner/outer walls to guide rollers. Drive via sprocket wheel (U-shaped cutouts engage ribs on belt elements) rotated step-by-step (e.g., 1/4 turn per dispense).
- Adaptation for Fragility: Add protective webs/plates to prevent kits from shifting; use yieldable resin for shock absorption. Sourcing: Polypropylene sheets (~$20/sq ft) from U.S. suppliers; sprockets/rollers (~$10 each).
- Figures Breakdown (from patent schematics):
  - Fig. 1: Perspective of column with belt and magazine walls.
  - Fig. 2: Side elevation showing oval path.
  - Fig. 3: Sprocket sectional view.
  - Fig. 4: Belt portion with pin-roller mating.
  - Fig. 5: Single element with shelf and dowels.

ASCII Representation of Conveyor Path:
```
[Magazine Wall]
   |
   v
[Sprocket Wheel] --> [Belt Start] --> [Oval Track Up]
                      ^              |
                      |              v
                     [Shelf w/ Kits] --> [Dispense Slot]
                      ^                  |
                      |                  v
                     [Return Down] <-- [Rollers in Raceway]
```

**Schematics** (Text Description with Pinouts; based on adapted Instructables and GitHub projects):
- Wiring for Raspberry Pi:
  - Stepper Motor Driver (A4988): Enable (GPIO 17), Step (GPIO 18), Dir (GPIO 27), connected to NEMA 17 motor.
  - Ultrasonic Sensor: Trig (GPIO 23), Echo (GPIO 24).
  - DHT22: Data (GPIO 4), VCC (5V), GND.
  - BLE (HC-05 module): TX (GPIO 14), RX (GPIO 15).
  - Cellular Modem (SIM800L): TX/RX to UART, power from 5V.
  - LoRa Module (SX1278): SPI pins (GPIO 10 MOSI, 9 MISO, 11 SCLK, 8 CE0), DIO0 (GPIO 25).

ASCII Schematic for Dispensing Circuit:
```
RPi GPIO 17 --[Enable]--> A4988 Driver --[IN1-IN4]--> Stepper Motor --> Belt Conveyor
RPi 5V/GND ----------------^---------------------|
Ultrasonic Trig/Echo -------|                     |
                           Feedback (Product Detect)
```

**Wiring Diagram Addition**: Based on a detailed visual schematic from an open-source DIY vending project (adapted from Arduino Mega to Raspberry Pi), the original image shows connections for LCD, buttons, IR sensor (for "coin" simulation or QR trigger), steppers (NEMA 17 via A4988), servos (for optional gates), and microswitches (for position feedback). Direct image URL reference: https://howtomechatronics.com/wp-content/uploads/2017/11/DIY-Vending-Machine-Arduino-based-Mechatronics-Project-Cirucit-Schematic.png.

Key Wiring (Adapted to Raspberry Pi GPIO):
- Power: 12V input → LM2596 Buck Converter → 5V out to Pi GPIO 5V/GND.
- LCD (16x2): RS (GPIO 27), Enable (26), D4-D7 (25-22).
- Buttons (x4 for quantity select): To GPIO 13-10 (pull-up enabled in code).
- IR Proximity Sensor (for dispense trigger/simulated access): Out (GPIO 9), 5V/GND.
- Stepper Motors (NEMA 17, Horizontal/Vertical axes for belt positioning): A4988 drivers – Step H (GPIO 3), Dir H (2); Step V (1), Dir V (0). 12V to VMOT, capacitor for smoothing.
- Microswitches (H/V limits): To GPIO 14-15 (GND when triggered).
- Servos (DS04-NFC for optional kit gates, x4): PWM (GPIO 4-7), 5V/GND.

ASCII Wiring Diagram:
```
12V Supply --> [LM2596 Buck] --> 5V --> [RPi GPIO 2/3/5V/GND]
                                      |
                                      v
[LCD Pins: RS(27), EN(26), D4-7(25-22)]  [Buttons: 13-10 to GND]
                                      |
                                      v
[IR Sensor: Out(9), 5V/GND] <-- [A4988 H: Step(3), Dir(2), 12V/5V/GND] --> [NEMA17 H Motor]
                                      |
                                      v
[Microswitches: 14-15 to GND] <-- [A4988 V: Step(1), Dir(0), 12V/5V/GND] --> [NEMA17 V Motor]
```

**Embedded Control Architecture**: Real-time tasks handled by Python scripts on Raspbian OS. RTOS not needed for simplicity; use threading for concurrency.

**Communication Architecture**: BLE for phone-machine (using GATT profiles), SMS via AT commands on modem, Mesh via Meshtastic on LoRa for alerts, 4G/WiFi for cloud (MQTT protocol).

**Software Architecture**: Layered – Firmware (local control), Mobile App (user interface), Cloud (backend).

---

### Component Specifications with Updated Sourcing Information

**Bill of Materials (BOM)** (US-based, low-cost; total ~$450 for prototype):
- Raspberry Pi 4 (4GB): $45, Adafruit/Digi-Key (PID: 4295).
- NEMA 17 Stepper Motors (x3): $15 each, Amazon (B07PZFNRXB).
- A4988 Drivers (x3): $5 each, Adafruit (ID: 3296).
- HC-SR04 Ultrasonic (x3): $3 each, Amazon (B01JG09DDE).
- DHT22 Sensor: $10, Adafruit (ID: 385).
- 7" LCD Touchscreen: $60, Amazon (B07V3W7QNW).
- HC-05 BLE Module: $8, Amazon (B01G9ZOF9U).
- SIM800L Modem: $12, Amazon (B07N7J9J4D).
- SX1278 LoRa Module: $15, Amazon (B07R3FXF5W).
- 10W Solar Kit (DFRobot): $35, DFRobot.com.
- 18650 Batteries (x3): $6 each, Amazon (B07QXV3WNR).
- Weatherproof Enclosure (Steel, 24x36x12"): $100, IconEnclosures.com (or equivalent from McMaster-Carr).
- Belt Conveyor Kit: $50, adapted from robotic parts, RobotShop (RB-Rbo-01).
- Other (wires, resistors, PCB): $20, Digi-Key.

**Component Specs Update**:
- Stepper Driver (L298N): 2A, 5-35V, 25W max – $5, Adafruit.
- Weight Sensor: 24-bit accuracy, ±40mV input, <10mA – For inventory confirmation post-dispense.
- Relay: 12V, 250mW, 16A capacity – For motor control isolation.

Multiple Vendors: Adafruit (open-source friendly), Amazon (fast), Digi-Key (bulk).

Performance: Motors tested to 1RPM for gentle dispense; sensors accurate to 1cm/0.5°C.

Standards: FCC for wireless, UL for power, ADA for height/access.

---

### Software Requirements Documentation

**Mobile Application**: Flutter-based (cross-platform), open-source extension of Simple QR. Features: QR scan to connect, input test quantity (1-5 limit), send via BLE or SMS. Flow: Scan → Auth (anonymous) → Select → Dispense. Code: Use ZXing for scan, bleak for BLE, Twilio API for SMS fallback.

**Machine Firmware**: Python on RPi, using RPi.GPIO, smbus, bleak. Structure: Main loop monitors sensors, handles BLE/SMS commands, controls motors. Update via OTA (balenaOS). Fault tolerance: Watchdog timer.

**Firmware Code Snippet Adaptation**: From an open-source Raspberry Pi vending firmware, adapt the engine structure for concurrency (e.g., handle BLE/SMS commands while monitoring sensors). Original uses Go; here's a Python equivalent for Pi ease.

```python
# Adapted Python Firmware for Covending Dispense (using RPi.GPIO)
import RPi.GPIO as GPIO
import time
import threading  # For concurrency

GPIO.setmode(GPIO.BCM)
# Pin setups from wiring above...

def dispense_kit(quantity):
    for _ in range(quantity):
        GPIO.output(stepPinHorizontal, GPIO.HIGH)  # Step motor
        time.sleep(0.001)
        GPIO.output(stepPinHorizontal, GPIO.LOW)
        # Repeat for steps needed (e.g., 200 steps/rev for NEMA17)
    print("Dispensed kits")

def monitor_sensors():
    while True:
        if GPIO.input(ir_sensor_pin) == 0:  # Trigger on access
            threading.Thread(target=dispense_kit, args=(1,)).start()  # Concurrent dispense
        time.sleep(0.1)

# Main engine
threading.Thread(target=monitor_sensors).start()
# Add BLE/SMS listeners here...
```

**Cloud Platform**: OpenRemote (open-source). Backend: Node.js/Express APIs, InfluxDB for time-series. Analytics: Python scripts for anomaly detection (e.g., if usage >2x average, alert). Dashboard: Web UI for inventory, maintenance.

**Cloud Analytics Enhancement**: Adapt IoT architecture from a cloud-based vending platform: User → QR Scan → Pi (local validation) → AWS EC2 (Django/PostgreSQL for anomaly detection). Data Flow: Anonymized timestamps → InfluxDB → Scikit-learn for spikes (e.g., if usage >2x avg, alert). Diagram Description:
- Layers: Device (Pi + Sensors) → Gateway (MQTT) → Cloud (EC2/Nginx/Gunicorn) → Analytics (DB + API).
- Adaptation: For Covending, add HIPAA anonymization at device layer; use for hotspot alerts.

**Privacy Framework**: Data anonymized at source (hash timestamps + location), no user IDs. Consent via app opt-in for location.

---

### Deployment Planning Guide

**Site Selection**: High-traffic areas (parks, transit); use GIS tools for demographics. Criteria: Foot traffic >500/day, ADA access, power/sun exposure.

**Installation**: Concrete foundation, bolt enclosure, connect solar. Permitting: Municipal health dept coordination.

**Maintenance**: Remote diagnostics via cloud, restock supplies as needed. Protocols: Clean weekly, battery check quarterly.

**Supply Chain**: Forecast via usage analytics, distribute via local partners.

**Regulatory**: Ask to be considered as a CLIA-waived tool, coordinate with FDA for non-diagnostic labeling.

**Build Protocol Additions** (Step-by-Step from DIY Project):
1. Assemble enclosure (MDF/acrylic, laser-cut per 3D model: https://thangs.com/m/44129).
2. Wire per schematic (test with multimeter).
3. Install Raspbian; run firmware script.
4. Calibrate belt: 200 steps = one kit dispense.
5. Test: Simulate QR via button; monitor weight sensor for confirmation.

---

### Implementation Roadmap with Milestones

- **Phase 1 (Weeks 1-4)**: Prototype build – Assemble hardware, test dispensing.
- **Phase 2 (Weeks 5-8)**: Software integration – Firmware, app, cloud setup.
- **Phase 3 (Weeks 9-12)**: Testing – Stress, environmental, pilot deploy 1 machine.
- **Phase 4 (Weeks 13-16)**: Scaling – Mesh/cloud, open-source release.
- **Phase 5 (Ongoing)**: Deployment – Citywide, community contributions.

---

### Risk Assessment and Mitigation Strategies

- **Technical**: Dispense failure – Mitigate with redundant sensors.
- **Regulatory**: FDA/CLIA scrutiny – Document as public health tool, consult lawyers.
- **Security**: Tampering – Locks, cameras, encryption. Add tamper detection via PIR sensor (low power, 50uA idle); if triggered without access, lock dispense.
- **Privacy**: Data breach – Anonymization, audits.
- **Economic**: High costs – Low-cost sourcing, grants.

---

### Appendices with Detailed Technical References

- **References**: GitHub repos; FDA CLIA docs; Solar specs.
- **Build Instructions**: Step-by-step from Instructables, adapted: 1. Assemble enclosure. 2. Wire RPi per schematics. 3. Install OS/firmware. 4. Test motors/sensors. 5. Integrate app/cloud.
- **Code Snippets**: From libraries – Library install; Custom Python for dispense: `import RPi.GPIO as GPIO; GPIO.output(17, True); # Enable motor`.

For contributions, fork and submit PRs via Github.


####Legal Disclaimer for Covending Machine Open-Source Project
IMPORTANT: READ CAREFULLY BEFORE USING OR IMPLEMENTING THIS PROJECT.

This document and the associated Covending Machine project (the "Project") are provided solely for informational, educational, and experimental purposes as an open-source prototype concept developed in response to the COVID-19 pandemic. The Project is intended to facilitate community-driven innovation in creation of public health tools, such as decentralized dispensing of rapid antigen tests, but it is NOT a finished product, medical device, or diagnostic tool. It is a conceptual prototype and should be treated as such.

No Medical Advice or Endorsement
The information, designs, specifications, and recommendations contained in this Project are NOT intended to provide medical advice, diagnosis, treatment, or prevention of any disease, including COVID-19. Rapid antigen tests dispensed via this system are for informational purposes only and are NOT a substitute for professional medical advice, FDA-authorized diagnostic testing (e.g., PCR tests), or consultation with qualified healthcare providers. Users must seek appropriate medical attention for any health concerns. The Project does NOT guarantee the accuracy, efficacy, safety, or suitability of any tests, hardware, software, or data generated. Reliance on this Project for medical decisions is at your own risk.

Open-Source Nature and "As Is" Provision
This Project is released under the MIT License (or equivalent open-source license) for community use, iteration, and contribution. All materials, including but not limited to hardware designs, schematics, software code, diagrams, and documentation, are provided "AS IS" and "WITH ALL FAULTS," without any representations or warranties of any kind, express or implied. This includes, but is not limited to, warranties of merchantability, fitness for a particular purpose, non-infringement, title, accuracy, completeness, reliability, or freedom from errors, viruses, or defects. The creators, contributors, and distributors (including Kyle McNease and any affiliates) make no claims regarding the performance, safety, or regulatory compliance of any built prototypes.

No Liability for Damages or Negative Externalities
To the fullest extent permitted by law, the creators, contributors, distributors, and any related parties disclaim all liability for any direct, indirect, incidental, consequential, special, punitive, or exemplary damages arising from or related to the use, misuse, or inability to use this Project. This includes, but is not limited to:

Personal injury, death, or health risks from building, operating, or interacting with prototypes (e.g., electrical hazards, mechanical failures, exposure to contaminants, or improper test handling).
Property damage, data loss, or financial losses.
Legal consequences, such as violations of federal, state, or local laws (e.g., FDA regulations, CLIA requirements, medical device laws, privacy laws like HIPAA, or municipal permitting).
Epidemiological or public health risks, including inaccurate data leading to misguided decisions on transmission hotspots.
Intellectual property disputes, patent infringements, or third-party claims.
You assume all risks associated with downloading, accessing, building, deploying, or modifying this Project. If you choose to implement it, you do so entirely at your own discretion and expense.

Regulatory Compliance and User Responsibility
This Project is NOT approved, certified, or endorsed by any regulatory body, including the FDA, CDC, WHO, or equivalent agencies. It is framed as a non-diagnostic public health tool prototype, but compliance with all applicable laws, standards (e.g., FCC, UL, ADA), and regulations is solely your responsibility. Users must:

Obtain necessary approvals, permits, and certifications before deployment.
Ensure adherence to privacy laws (e.g., anonymization of data) and obtain user consent where required.
Verify component sourcing, assembly, and operation to prevent hazards.
Adapt the Project for local contexts, including GDPR for non-US use if applicable.
The Project does not constitute legal advice. Consult qualified legal, medical, engineering, and regulatory professionals before proceeding.

Indemnification
By using this Project, you agree to indemnify, defend, and hold harmless the creators, contributors, and distributors from any claims, losses, liabilities, expenses, or demands arising from your use or implementation.

Changes and Updates
This disclaimer may be updated without notice. Continued use of the Project constitutes acceptance of any changes.

If any provision of this disclaimer is held invalid, the remainder shall remain in effect.

BY ACCESSING, USING, OR CONTRIBUTING TO THIS PROJECT, YOU ACKNOWLEDGE THAT YOU HAVE READ, UNDERSTOOD, AND AGREE TO BE BOUND BY THIS DISCLAIMER. IF YOU DO NOT AGREE, DO NOT USE THIS PROJECT.
