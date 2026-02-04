# Electronics: Wiring Guide

This guide covers the electrical integration of the RUKA-V2 robotic hand, including power distribution, servo connections, and microcontroller setup.

## Safety First

⚠️ **Important Safety Warnings:**
- Disconnect power before making any connections
- Verify polarity before connecting power
- Use appropriate wire gauges for current loads
- Insulate all connections properly
- Double-check wiring before applying power

## Required Components

### Electronics
- [ ] Microcontroller board (Arduino Mega or compatible)
- [ ] Servo motors (5x, already installed in hand)
- [ ] Power supply (12V, 5A minimum)
- [ ] Voltage regulator (12V to 5V)
- [ ] Encoder modules (5x)
- [ ] Capacitors (1000µF, 25V)
- [ ] Diodes (flyback protection)

### Wiring Materials
- [ ] 22 AWG wire (red and black for power)
- [ ] 24 AWG wire (various colors for signals)
- [ ] Servo extension cables (if needed)
- [ ] Connector headers
- [ ] Heat shrink tubing
- [ ] Wire ferrules

### Tools
- Wire strippers
- Crimping tool
- Soldering iron and solder
- Multimeter
- Heat gun
- Wire cutters
- Label maker

## Power System Architecture

```
Power Supply (12V, 5A)
    │
    ├─→ Servo Power Bus (12V)
    │   └─→ Servos 1-5
    │
    └─→ Voltage Regulator
        └─→ Logic Power (5V)
            ├─→ Microcontroller
            └─→ Encoders
```

## Wiring Steps

### 1. Power Distribution

#### 1.1 Create Power Bus

1. **Prepare main power lines**
   - Cut red wire (~50cm) for positive bus
   - Cut black wire (~50cm) for negative/ground bus
   - Strip ends appropriately
   - Tin wire ends with solder

2. **Install bulk capacitor**
   - Solder 1000µF capacitor across power bus
   - Observe correct polarity (+ and -)
   - Place near servo connections
   - Insulate with heat shrink

3. **Add power input connector**
   - Attach barrel jack or screw terminal
   - Connect to power bus
   - Add inline fuse (5A) for protection
   - Verify polarity with multimeter

#### 1.2 Voltage Regulation

1. **Install voltage regulator**
   - Mount regulator on suitable location
   - Connect 12V input from power bus
   - Connect ground
   - Verify 5V output with multimeter

2. **Create logic power distribution**
   - Run 5V line to microcontroller
   - Branch to encoder modules
   - Add 100µF capacitor for stability
   - Ensure adequate current capacity

### 2. Servo Connections

#### 2.1 Servo Power Wiring

For each of the 5 servos:

1. **Connect servo power (red wire)**
   - Connect to 12V power bus
   - Use appropriate gauge wire (22 AWG minimum)
   - Ensure solid connections
   - Consider using distribution board

2. **Connect servo ground (black/brown wire)**
   - Connect to ground bus
   - Maintain common ground throughout system
   - Keep ground wiring low-resistance

3. **Add flyback diodes**
   - Install diode across each servo power connection
   - Cathode (stripe) to positive
   - Protects against voltage spikes
   - Use 1N4004 or equivalent

#### 2.2 Servo Signal Wiring

1. **Connect servo signals to microcontroller**
   
   | Servo | Finger | Arduino Pin | Wire Color |
   |-------|--------|-------------|------------|
   | 1 | Thumb | PWM 2 | Orange |
   | 2 | Index | PWM 3 | Yellow |
   | 3 | Middle | PWM 4 | Green |
   | 4 | Ring | PWM 5 | Blue |
   | 5 | Pinky | PWM 6 | Purple |

2. **Route signal wires**
   - Keep separate from power wires
   - Use shielded cable if EMI is a concern
   - Leave service loops
   - Label clearly

### 3. Encoder Integration

#### 3.1 Encoder Power

1. **Connect encoder power**
   - Use 5V from voltage regulator
   - Parallel connection for all 5 encoders
   - Add decoupling capacitor (100nF) at each encoder
   - Verify voltage at each encoder

#### 3.2 Encoder Signal Connections

For each encoder (quadrature output):

| Encoder | Finger | Arduino Pins | Signals |
|---------|--------|--------------|---------|
| 1 | Thumb | 18, 19 | A, B |
| 2 | Index | 20, 21 | A, B |
| 3 | Middle | 22, 23 | A, B |
| 4 | Ring | 24, 25 | A, B |
| 5 | Pinky | 26, 27 | A, B |

1. **Wire encoder channels**
   - Channel A to first pin
   - Channel B to second pin
   - Ground to common ground
   - Use twisted pair if possible

2. **Verify encoder operation**
   - Manually rotate servo
   - Check for signal pulses with multimeter
   - Verify both channels working
   - Confirm correct direction

### 4. Microcontroller Setup

#### 4.1 Board Installation

1. **Mount microcontroller**
   - Choose accessible location
   - Use standoffs or mounting plate
   - Ensure adequate ventilation
   - Protect from shorts

2. **Connect power**
   - VIN or 5V input from regulator
   - Connect ground
   - Verify voltage before connecting other wires
   - Add power LED for indication

#### 4.2 Additional Connections

1. **Communication interface**
   - USB connection for programming
   - Optional: Serial communication pins
   - Optional: I2C for sensors
   - Optional: SPI if needed

2. **Status indicators**
   - Power LED
   - Activity LED on digital pin
   - Optional: RGB LED for status codes

### 5. Cable Management

1. **Organize wiring**
   - Bundle by function (power, signals, encoders)
   - Use cable ties at regular intervals
   - Label all connection points
   - Create wiring diagram for reference

2. **Strain relief**
   - Secure cables at entry/exit points
   - Prevent tension on connections
   - Use grommets for panel pass-throughs
   - Allow for movement if needed

## Wiring Verification Checklist

### Before Powering On

- [ ] All connections are secure
- [ ] Polarity is correct everywhere
- [ ] No exposed conductors or shorts
- [ ] Capacitors are installed and polarized correctly
- [ ] Voltage regulator output verified (5V)
- [ ] All ground connections are common
- [ ] Servo power and signal connections verified
- [ ] Encoder connections verified
- [ ] Cable management is complete

### Visual Inspection

- [ ] No loose wires
- [ ] No solder bridges
- [ ] No damaged insulation
- [ ] All labels are in place
- [ ] Professional appearance

## Testing Procedure

See [Power Testing](power-test.md) for detailed power-on and testing procedures.

## Common Wiring Issues

### No Power to System
- Check power supply is plugged in
- Verify fuse is intact
- Check power switch position
- Test voltage at multiple points

### Servos Not Responding
- Verify servo power (12V present)
- Check signal connections
- Ensure common ground
- Test servo individually

### Erratic Servo Behavior
- Check for power supply sagging
- Add larger capacitor
- Verify adequate current rating
- Check for loose connections

### Encoders Not Working
- Verify 5V power to encoders
- Check signal wiring
- Test encoder output with scope
- Verify encoder is not damaged

## Best Practices

- **Color Coding**: Use consistent colors (red=power, black=ground, others=signals)
- **Labeling**: Label everything clearly
- **Documentation**: Take photos and create wiring diagram
- **Testing**: Test each subsystem before integration
- **Safety**: Always disconnect power when making changes

## Next Steps

After completing wiring, proceed to power testing.

[Continue to Power Testing →](power-test.md)

---

[← Back to Main Documentation](../README.md) | [Assembly Complete](../assembly/step-05-final.md)
