## Description


<img src="https://github.com/andreipopescufilimon/Line-Follower-16-sensors-array/blob/d06ab8c9a251becfe6cc96c24f56829ca2392df1/media/PCB.png" width="600" />


This is a compact curved array of sixteen reflective sensors designed for line follower robots.  
The curved geometry improves detection consistency through turns and increases overall line position resolution compared to straight sensor layouts.

Each sensor channel includes its own onboard status LED that reflects the detection state in real time. This makes sensor alignment, calibration, and debugging much easier during setup and tuning.

You can find the Gerber File and EasyEDA Project file on: [https://oshwlab.com/bicicleta11/line-follower-16-sensors-array](https://oshwlab.com/bicicleta11/line-follower-16-sensors-array)

---

## Hardware Summary

- Sixteen analog reflective sensors arranged in a curved layout
- One dedicated status LED per sensor channel
- Two-layer PCB with carefully routed analog signals
- FFC interface for clean and compact connections
- Designed for direct connection to a microcontroller ADC

---

## Design Notes

- The curved sensor placement provides smoother line tracking on sharp curves
- Individual LEDs allow quick visual verification of sensor response
- Analog routing is kept short and isolated to reduce noise
- FFC connector minimizes wiring bulk and improves mechanical reliability

## Sensor to Multiplexer Channel Table

The table below shows how each reflective sensor maps to the multiplexer channel, the required logic levels on the select lines, and the position threshold range used by the firmware.

- **S0–S3** correspond to the multiplexer select pins  
- `0` = LOW, `1` = HIGH  
- Position thresholds scale from left (0) to right (15000)

| Sensor No | MUX Channel | S0 | S1 | S2 | S3 | Position Threshold |
|----------:|------------:|:--:|:--:|:--:|:--:|-------------------:|
| 1  | 0  | 0 | 0 | 0 | 0 | 0     |
| 2  | 1  | 0 | 0 | 0 | 1 | 1000  |
| 3  | 2  | 0 | 0 | 1 | 0 | 2000  |
| 4  | 3  | 0 | 0 | 1 | 1 | 3000  |
| 5  | 4  | 0 | 1 | 0 | 0 | 4000  |
| 6  | 5  | 0 | 1 | 0 | 1 | 5000  |
| 7  | 6  | 0 | 1 | 1 | 0 | 6000  |
| 8  | 7  | 0 | 1 | 1 | 1 | 7000  |
| 9  | 8  | 1 | 0 | 0 | 0 | 8000  |
| 10  | 9  | 1 | 0 | 0 | 1 | 9000  |
| 11 | 10 | 1 | 0 | 1 | 0 | 10000 |
| 12 | 11 | 1 | 0 | 1 | 1 | 11000 |
| 13 | 12 | 1 | 1 | 0 | 0 | 12000 |
| 14 | 13 | 1 | 1 | 0 | 1 | 13000 |
| 15 | 14 | 1 | 1 | 1 | 0 | 14000 |
| 16 | 15 | 1 | 1 | 1 | 1 | 15000 |

---

## Multiplexer Sensor Reading

The robot uses a 16-channel analog multiplexer to read all reflective sensors through a single ADC pin on the microcontroller.

Four digital lines select the active sensor channel, while the analog output of the multiplexer is connected to **A4**. Channel selection is done using direct port manipulation on **PORTC**, which is significantly faster than `digitalWrite()` and allows high-speed sensor sampling.

### Channel Selection

```cpp
inline void selectMuxChannel(uint8_t ch) {
  PORTC = (PORTC &amp; 0xF0) | (ch &amp; 0x0F);
}
```

- PC0 to PC3 are used as multiplexer select lines
- Each channel corresponds to one reflective sensor
- Only the lower 4 bits of PORTC are modified
*you can also do 4 digital writes on the 4 MUX control pins where you select the channel via combinations of 0 and 1*


## Fast Analog Read

To reduce ADC overhead, a custom fast analog read routine is used instead of analogRead().

```c++
inline uint16_t fastAnalogRead(uint8_t ch) {
  ADMUX = (1 &lt;&lt; REFS0) | (ch &amp; 0x0F);
  ADCSRA |= (1 &lt;&lt; ADSC);
  while (ADCSRA &amp; (1 &lt;&lt; ADSC));
  uint8_t low = ADCL;
  uint8_t high = ADCH;
  return (high &lt;&lt; 8) | low;
}
```

- ADC reference is set to AVcc
- Prescaler is configured for faster conversion
- Sensor data is read from analog channel A4
- Weak reflectance values are ignored by thresholding in firmware

*but also simple analog read does the job :)*

## PCB Schematic
<img src="https://github.com/andreipopescufilimon/Line-Follower-16-sensors-array/blob/d06ab8c9a251becfe6cc96c24f56829ca2392df1/media/schematic.webp" width="600" />

## PCB Layout
<img src="https://github.com/andreipopescufilimon/Line-Follower-16-sensors-array/blob/d06ab8c9a251becfe6cc96c24f56829ca2392df1/media/layout.webp" width="600" />
