# Electronics: Power Testing

This guide covers the systematic testing of the electrical system to ensure everything is wired correctly before full system operation.

## Safety Precautions

⚠️ **Critical Safety Steps:**
- Have fire extinguisher nearby
- Work in well-ventilated area
- Keep hand on power switch for quick cutoff
- Wear safety glasses
- Have emergency procedures ready
- Never touch circuits while powered

## Required Equipment

### Testing Tools
- [ ] Multimeter (essential)
- [ ] Oscilloscope (recommended for encoders)
- [ ] Current meter or clamp meter
- [ ] Variable power supply (optional, for gradual power-up)
- [ ] Logic analyzer (optional, for debugging)

### Safety Equipment
- [ ] Safety glasses
- [ ] Fire extinguisher
- [ ] Insulated tools
- [ ] Anti-static mat

## Pre-Power Checks

### 1. Visual Inspection

Complete this checklist before applying power:

- [ ] All wiring connections are secure
- [ ] No exposed wire strands or conductors
- [ ] Correct polarity on all power connections
- [ ] Capacitors installed with correct polarity
- [ ] No solder bridges or shorts
- [ ] All components are properly mounted
- [ ] No damaged components visible
- [ ] Cable management is secure

### 2. Continuity Testing

**With power OFF:**

1. **Test power bus continuity**
   - Verify positive bus is continuous
   - Verify ground bus is continuous
   - Confirm no continuity between positive and ground
   - Resistance between power and ground should be >10kΩ

2. **Test servo connections**
   - Each servo power: continuity to power bus
   - Each servo ground: continuity to ground bus
   - Verify signal lines not shorted to power
   - Check each servo connector pins

3. **Test encoder connections**
   - Power continuity to regulator output
   - Ground continuity
   - Signal lines isolated from power
   - No shorts between signal lines

### 3. Resistance Measurements

**With power OFF and unplugged:**

1. **Measure system resistance**
   - Between power bus and ground: Should be >10kΩ
   - If <1kΩ: **STOP** - find and fix short
   - Between signal lines: Should be high resistance

2. **Check voltage regulator**
   - Input to ground: >1kΩ
   - Output to ground: >1kΩ
   - Input to output: Should show diode behavior

## Power-Up Sequence

### Step 1: Initial Power Application

1. **Set power supply to 11V** (slightly low for safety)
   - Start with reduced voltage
   - Verify current limit set to 5A
   - Power supply OFF

2. **Connect power supply**
   - Double-check polarity
   - Connect with power switch OFF
   - Have hand ready to disconnect

3. **Turn on power**
   - Monitor current draw immediately
   - Expected idle current: 100-200mA
   - If current >1A immediately: **CUT POWER**
   - Watch for smoke or heating

4. **Immediate checks**
   - Touch power components (should be cool)
   - Smell for burning
   - Listen for unusual sounds
   - Check LEDs (power LED should be on)

### Step 2: Voltage Verification

**With power ON at 11V:**

1. **Measure main power bus**
   - Should read ~11V
   - Verify stability (no drooping)
   - Check multiple points on bus

2. **Measure regulator output**
   - Should read 5.0V (±0.1V)
   - If not: **STOP** - check regulator
   - Verify stable under no load

3. **Measure at each servo**
   - Power pin: Should be ~11V
   - Ground pin: 0V
   - Signal pin: ~3-5V typically

4. **Measure at each encoder**
   - VCC: Should be 5.0V
   - GND: 0V
   - Signal lines: Variable, but within 0-5V

**If all voltages correct, proceed. If not, power off and troubleshoot.**

### Step 3: Increase to Full Voltage

1. **Raise power supply to 12V**
   - Increase gradually
   - Monitor current draw
   - Expected: 150-250mA idle

2. **Repeat voltage measurements**
   - Main bus: 12V
   - Regulator output: 5.0V
   - All components: correct voltages

3. **Monitor system**
   - Leave powered for 5 minutes
   - Check for heating
   - Monitor current (should be stable)
   - Any component hot? Investigate.

## Subsystem Testing

### 4. Microcontroller Test

1. **Verify microcontroller powered**
   - Check power LED on board
   - Measure VCC pin: 5V
   - Measure GND: 0V

2. **Upload test sketch**
   ```cpp
   // Simple blink test
   void setup() {
     pinMode(LED_BUILTIN, OUTPUT);
   }
   void loop() {
     digitalWrite(LED_BUILTIN, HIGH);
     delay(1000);
     digitalWrite(LED_BUILTIN, LOW);
     delay(1000);
   }
   ```

3. **Verify communication**
   - LED should blink
   - Serial monitor should connect
   - Board responds to uploads

### 5. Servo Power Test

**Test each servo individually:**

1. **Prepare test setup**
   - Connect only one servo signal
   - Others remain powered but no signal
   - Upload servo test sketch

2. **Servo test code**
   ```cpp
   #include <Servo.h>
   Servo testServo;
   
   void setup() {
     testServo.attach(2);  // Adjust pin number
     Serial.begin(9600);
   }
   
   void loop() {
     Serial.println("Moving to 90");
     testServo.write(90);
     delay(1000);
     Serial.println("Moving to 0");
     testServo.write(0);
     delay(1000);
   }
   ```

3. **For each servo:**
   - Connect signal wire
   - Upload and run test
   - Servo should move smoothly
   - Monitor current draw (1-2A per servo)
   - Check for overheating
   - Verify consistent behavior

4. **Record observations:**
   - Servo direction (CW/CCW)
   - Smooth operation (yes/no)
   - Current draw
   - Any issues

### 6. Encoder Test

**Test each encoder individually:**

1. **Encoder test code**
   ```cpp
   volatile long encoderPos = 0;
   
   void setup() {
     Serial.begin(9600);
     pinMode(18, INPUT_PULLUP);  // Channel A
     pinMode(19, INPUT_PULLUP);  // Channel B
     attachInterrupt(digitalPinToInterrupt(18), encoderISR, CHANGE);
   }
   
   void encoderISR() {
     if (digitalRead(18) == digitalRead(19)) {
       encoderPos++;
     } else {
       encoderPos--;
     }
   }
   
   void loop() {
     Serial.println(encoderPos);
     delay(100);
   }
   ```

2. **For each encoder:**
   - Upload test code with correct pins
   - Manually rotate servo shaft
   - Verify count increases/decreases
   - Check for missed counts
   - Verify direction is consistent

### 7. Integrated System Test

1. **Connect all servos and encoders**
   - All signal wires connected
   - All powered

2. **Upload basic control sketch**
   - Simple position control
   - Move each finger sequentially
   - Monitor encoder feedback

3. **System checks:**
   - All servos respond
   - All encoders report position
   - No excessive current draw (<4A total)
   - No overheating
   - Smooth coordinated motion

## Current Draw Analysis

Expected current consumption:

| State | Expected Current | Acceptable Range |
|-------|------------------|------------------|
| Idle (no servo load) | 150-250mA | 100-400mA |
| One servo active | 0.5-1.5A | 0.3-2A |
| All servos active | 2-4A | 1.5-5A |
| Peak (all servos stall) | 5-7A | 4-8A |

**Action if current is outside acceptable range:**
- Low: Check connections
- High: Check for shorts, binding, or damaged components

## Temperature Monitoring

After 10 minutes of operation:

| Component | Expected Temp | Max Safe Temp |
|-----------|---------------|---------------|
| Voltage regulator | Warm (40-50°C) | 70°C |
| Servos | Warm (40-60°C) | 80°C |
| Capacitors | Cool (25-35°C) | 50°C |
| Wiring | Cool (25-30°C) | 40°C |
| Microcontroller | Cool-Warm (30-45°C) | 60°C |

**If any component exceeds max safe temperature: STOP and investigate**

## Troubleshooting Guide

### Problem: High Current at Power-On

**Possible causes:**
- Short circuit in wiring
- Damaged component
- Incorrect voltage
- Servo jammed or binding

**Solutions:**
1. Power off immediately
2. Check resistance between power and ground
3. Disconnect servos one at a time to isolate issue
4. Inspect for visible damage

### Problem: Voltage Regulator Not Outputting 5V

**Possible causes:**
- Damaged regulator
- Incorrect wiring
- Overload on output

**Solutions:**
1. Check input voltage (should be 12V)
2. Verify regulator wiring
3. Test with no load
4. Replace regulator if damaged

### Problem: Servos Not Moving

**Possible causes:**
- No signal connection
- Wrong signal polarity
- Servo power issue
- Code problem

**Solutions:**
1. Verify servo power (12V at servo)
2. Check signal connection
3. Test with known good servo
4. Verify code is correct

### Problem: Encoders Not Reading

**Possible causes:**
- No power to encoder
- Signal wires swapped
- Damaged encoder
- Wrong pins in code

**Solutions:**
1. Verify 5V at encoder
2. Check signal wire connections
3. Test encoder signals with scope
4. Verify pin numbers in code

## Safety Issues to Watch For

**IMMEDIATE POWER OFF if you observe:**
- 🔥 Smoke or burning smell
- ⚡ Sparking
- 🔊 Unusual sounds (popping, sizzling)
- 🔥 Components too hot to touch
- ⚡ Current draw >6A continuously
- 💧 Any liquid near electronics

## Test Results Documentation

Create a test log with:

- Date and time of testing
- Power supply voltage and current settings
- Voltage measurements at key points
- Current draw measurements
- Servo operation notes
- Encoder operation notes
- Any issues encountered
- Resolutions applied
- Photos of setup

## Success Criteria

System passes power testing when:
- ✅ All voltage measurements are correct
- ✅ Current draw is within expected range
- ✅ No components overheat
- ✅ All servos respond correctly
- ✅ All encoders read position
- ✅ System operates stable for 10+ minutes
- ✅ No unusual sounds or smells
- ✅ Safe to proceed to calibration

## Next Steps

After successful power testing, proceed to calibration.

[Continue to Calibration: Encoders →](../calibration/encoders.md)

---

[← Previous: Wiring Guide](wiring.md) | [← Back to Main Documentation](../README.md)
