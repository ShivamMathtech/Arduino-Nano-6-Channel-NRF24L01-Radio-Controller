# Arduino Nano 6-Channel NRF24L01 Radio Controller

A compact six-channel wireless control system built with two Arduino Nano boards and NRF24L01+PA+LNA 2.4 GHz transceivers. The transmitter reads two dual-axis joysticks and two potentiometers. The receiver converts the transmitted values into six control outputs.

> **Project status:** Educational prototype. Test the circuit without actuators first and add suitable power protection before installing it in a vehicle or robot.

## Features

- Six proportional control channels
- Four joystick axes plus two auxiliary potentiometers
- NRF24L01+PA+LNA two-way radio hardware
- Separate transmitter and receiver circuits
- Compact Arduino Nano implementation
- 100 µF radio-supply decoupling capacitor
- Six receiver signal headers

## System overview

```text
Two joysticks + two potentiometers
                │
                ▼
       Transmitter Arduino Nano
                │ SPI
                ▼
          NRF24L01 radio
          2.4 GHz wireless
          NRF24L01 radio
                │ SPI
                ▼
        Receiver Arduino Nano
                │
                ▼
          Channels 1 to 6
```

## Required hardware

### Transmitter

| Quantity | Component |
| ---: | --- |
| 1 | Arduino Nano |
| 1 | NRF24L01+PA+LNA transceiver with antenna |
| 1 | NRF24L01 5 V adapter/regulator board |
| 2 | PS2-style dual-axis joystick modules |
| 2 | 10 kΩ linear potentiometers |
| 1 | 100 µF, 16 V electrolytic capacitor |
| 1 | Suitable battery, switch, and connector |
| — | Perfboard, headers, and hookup wire |

### Receiver

| Quantity | Component |
| ---: | --- |
| 1 | Arduino Nano |
| 1 | NRF24L01+PA+LNA transceiver with antenna |
| 1 | NRF24L01 5 V adapter/regulator board |
| 1 | 100 µF, 16 V electrolytic capacitor |
| 6 | Three-pin output headers (`Signal`, `+5V`, `GND`) |
| 1 | 2-cell 7.4 V Li-ion battery pack |
| 1 | Regulated 5 V BEC appropriate for the connected load |

## NRF24L01 SPI connections

Use the same radio wiring on the transmitter and receiver.

| NRF24L01 signal | Arduino Nano | Purpose |
| --- | --- | --- |
| GND | GND | Ground |
| VCC | 5V through the adapter input | Adapter power |
| CE | D9 | Radio enable |
| CSN | D10 | SPI chip select |
| SCK | D13 | SPI clock |
| MOSI | D11 | SPI controller output |
| MISO | D12 | SPI controller input |
| IRQ | Not connected | Optional interrupt |

### NRF24L01 2×4 header orientation

Always confirm the orientation printed on the specific radio module before applying power.

| Pin | Signal | Pin | Signal |
| ---: | --- | ---: | --- |
| 1 | GND | 2 | VCC |
| 3 | CE | 4 | CSN |
| 5 | SCK | 6 | MOSI |
| 7 | MISO | 8 | IRQ |

> **Important:** A bare NRF24L01 module uses **3.3 V**, not 5 V. The diagrams assume a compatible NRF24L01 adapter board whose input accepts 5 V and regulates it for the radio. Never connect a bare radio's VCC pin directly to 5 V.

Connect the 100 µF capacitor across the adapter input supply: capacitor `+` to adapter VCC and capacitor `−` to GND. Observe capacitor polarity.

## Transmitter pin mapping

### Joystick 1

| Joystick pin | Arduino Nano |
| --- | --- |
| VCC | 5V |
| GND | GND |
| VRx | A0 |
| VRy | A1 |
| SW | Not connected |

### Joystick 2

| Joystick pin | Arduino Nano |
| --- | --- |
| VCC | 5V |
| GND | GND |
| VRx | A2 |
| VRy | A3 |
| SW | Not connected |

### Auxiliary potentiometers

Each potentiometer has two outer terminals and one center wiper.

| Control | Supply terminals | Wiper |
| --- | --- | --- |
| Potentiometer 1 | 5V and GND | A6 |
| Potentiometer 2 | 5V and GND | A7 |

If a channel moves in the opposite direction, swap that potentiometer's two outer terminals or invert the channel in firmware.

### Transmitter power

- Connect a suitable 7–9 V source to `VIN` and `GND`, or use a regulated 5 V source at the `5V` pin.
- Never apply battery voltage to the `5V` pin.
- Add an inline power switch and reverse-polarity protection.
- All module grounds must be connected together.

## Receiver pin mapping

| Output | Arduino Nano signal pin | Typical role |
| --- | --- | --- |
| Channel 1 | D2 | Primary control 1 |
| Channel 2 | D3 | Primary control 2 |
| Channel 3 | D4 | Primary control 3 |
| Channel 4 | D5 | Primary control 4 |
| Channel 5 (AUX 1) | D6 | Auxiliary control 1 |
| Channel 6 (AUX 2) | D7 | Auxiliary control 2 |

Arrange every receiver output header consistently:

```text
S  = channel signal
+  = regulated 5 V actuator rail
−  = common ground
```

### Receiver power

1. Connect the 7.4 V battery positive lead to Arduino `VIN`.
2. Connect the battery negative lead to Arduino `GND`.
3. Power servo or actuator `+` pins from a suitable external regulated 5 V BEC.
4. Connect BEC ground, battery ground, Arduino ground, radio-adapter ground, and output-header grounds together.
5. Do **not** power several servos from the Arduino Nano's onboard 5 V regulator.

The BEC current rating must exceed the combined stall current of all connected servos or actuators.

## Recommended firmware behavior

The transmitter firmware should:

1. Read `A0`, `A1`, `A2`, `A3`, `A6`, and `A7`.
2. Calibrate the center and end points of every input.
3. Pack six normalized channel values into one radio payload.
4. Transmit at a fixed update rate with a sequence number or checksum.

The receiver firmware should:

1. Read and validate each radio packet.
2. Apply limits and optional smoothing.
3. Generate the required output format on `D2` through `D7`.
4. Enter a predetermined failsafe state when packets stop arriving.

> The output format depends on the connected equipment. Standard hobby servos usually require servo pulses; another controller may expect PWM, PPM, SBUS, or simple digital outputs. Confirm the required interface before connecting hardware.

## Suggested radio configuration

Both boards must use matching settings:

- The same RF address
- The same channel
- The same data rate
- The same payload structure
- The same auto-acknowledgment setting

Begin testing at low transmit power and short range. Increase power only after reliable bench operation is confirmed.

## Assembly procedure

1. Build the power and ground rails with the battery disconnected.
2. Verify continuity between all ground points.
3. Check that `VIN`, regulated `5V`, and radio `3.3V` domains are not shorted.
4. Install the NRF24L01 adapter and capacitor.
5. Connect the SPI signals to `D9`–`D13` according to the table.
6. Wire the transmitter controls or receiver channel headers.
7. Inspect capacitor and battery polarity.
8. Power the Nano from USB first and test without servos or actuators.
9. Upload the transmitter and receiver firmware.
10. Confirm radio communication using the serial monitor.
11. Test channel values and failsafe behavior.
12. Connect the external BEC, then add one actuator at a time.

## Pre-power checklist

- [ ] No bare NRF24L01 VCC pin is connected directly to 5 V
- [ ] Battery positive connects to `VIN`, not `5V`
- [ ] Battery polarity is correct
- [ ] Electrolytic capacitor polarity is correct
- [ ] Transmitter and receiver share their required common grounds
- [ ] Receiver actuator power comes from an adequately rated BEC
- [ ] Channel connector polarity is consistent
- [ ] Antennas are attached before using PA+LNA radio modules
- [ ] No actuator is connected during the first communication test
- [ ] Firmware failsafe has been tested

## Troubleshooting

| Problem | Checks |
| --- | --- |
| Radio not detected | Verify CE/CSN pins, SPI pins, adapter orientation, and common ground |
| Intermittent packets | Place the capacitor close to the adapter, shorten power leads, and use a stable supply |
| Nano resets when servos move | Use a higher-current external BEC and improve grounding; do not use the Nano regulator for servo power |
| Channels reversed | Swap a potentiometer's outer pins or reverse the value in firmware |
| Joystick center drifts | Add calibration, deadband, and averaging in firmware |
| Short operating range | Check antenna connection, power integrity, RF settings, and physical obstructions |
| Outputs move after link loss | Implement and test a timed receiver failsafe |

## Safety

- Use protected Li-ion cells and a charger designed for a 2-cell pack.
- Add a fuse appropriate for the wiring and expected load.
- Never charge the battery while unattended.
- Keep exposed conductors insulated and strain-relieve battery leads.
- Do not operate a moving platform until link-loss behavior has been tested with the drive system raised or disconnected.
- This project is not intended for life-critical, aviation, or safety-certified control systems.

## License

Choose and add a license before redistributing the hardware design or firmware. The documentation may be adapted for educational and prototyping use.
