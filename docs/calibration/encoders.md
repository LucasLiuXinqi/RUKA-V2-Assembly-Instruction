# Calibration: Encoder Calibration

Encoder calibration is essential for accurate position control of the RUKA-V2 robotic hand. This guide walks through the process of calibrating each finger's encoder system.

## Overview

Encoder calibration establishes the relationship between:
- Encoder counts and actual finger position
- Servo position and finger angle
- Zero position reference for each finger

## Prerequisites

Before starting calibration:
- [ ] Mechanical assembly is complete
- [ ] Electronics are wired and tested
- [ ] Power system is functioning correctly
- [ ] All servos respond to commands
- [ ] All encoders are reading values
- [ ] Microcontroller is programmed with calibration sketch

## Required Equipment

### Hardware
- Fully assembled RUKA-V2 hand
- Programmed microcontroller
- Serial connection to computer
- Protractor or angle measurement tool (optional)
- Notebook for recording values

### Software
- Arduino IDE or compatible
- Serial monitor or terminal program
- Calibration sketch (provided below)

## Calibration Concepts

### Understanding Encoder Counts

- **Encoder resolution**: Number of counts per revolution
- **Gear ratio**: Mechanical reduction between servo and finger
- **Range**: Total counts for full finger motion
- **Zero position**: Reference point (typically finger fully extended)

### Calibration Goals

1. **Establish zero position** - Define fully extended position
2. **Measure range** - Determine counts for full range of motion
3. **Create mapping** - Relate encoder counts to finger angles
4. **Store values** - Save calibration to EEPROM or config file

## Calibration Procedure

### Step 1: Setup

1. **Load calibration sketch**
   ```cpp
   // Basic calibration sketch framework
   #include <Servo.h>
   
   // Encoder pins for all fingers
   const int encoderPins[5][2] = {
     {18, 19},  // Thumb
     {20, 21},  // Index
     {22, 23},  // Middle
     {24, 25},  // Ring
     {26, 27}   // Pinky
   };
   
   volatile long encoderCounts[5] = {0};
   Servo servos[5];
   
   void setup() {
     Serial.begin(115200);
     // Initialize servos on pins 2-6
     for(int i = 0; i < 5; i++) {
       servos[i].attach(i + 2);
       pinMode(encoderPins[i][0], INPUT_PULLUP);
       pinMode(encoderPins[i][1], INPUT_PULLUP);
     }
     Serial.println("Calibration Ready");
   }
   
   void loop() {
     // Print encoder values
     for(int i = 0; i < 5; i++) {
       Serial.print("F");
       Serial.print(i);
       Serial.print(":");
       Serial.print(encoderCounts[i]);
       Serial.print(" ");
     }
     Serial.println();
     delay(100);
   }
   ```

2. **Connect to serial monitor**
   - Open serial monitor at 115200 baud
   - Verify encoder counts are displayed
   - Manually move fingers and observe counts change

### Step 2: Find Zero Position (Per Finger)

**For each finger (starting with thumb):**

1. **Extend finger fully**
   - Manually move finger to fully extended position
   - This will be the zero reference
   - Ensure finger is naturally extended, not forced

2. **Reset encoder count**
   - Send 'z' command via serial to zero the encoder
   - Or manually record current count as zero offset
   - Verify count resets or note the value

3. **Verify zero position**
   - Move finger slightly and return to zero
   - Encoder should return to 0 (or noted offset)
   - Repeat if not consistent

### Step 3: Measure Full Range

**For each finger:**

1. **Move to fully closed position**
   - Slowly curl finger to maximum flexion
   - Don't force past natural range
   - Monitor encoder count

2. **Record maximum count**
   - Note the encoder count at full flexion
   - This is your maximum range value
   - Example: Thumb = 4850 counts

3. **Test reversibility**
   - Extend finger back to zero
   - Verify return to zero position
   - Check count accuracy

### Step 4: Map Intermediate Positions

**Create position mapping:**

1. **Test at 25% closure**
   - Move finger to approximately 1/4 closed
   - Record encoder count
   - Note any non-linearity

2. **Test at 50% closure**
   - Move finger to half closed
   - Record encoder count
   - Compare to expected (should be ~50% of max)

3. **Test at 75% closure**
   - Move finger to 3/4 closed
   - Record encoder count

4. **Analyze linearity**
   - Plot points if needed
   - Determine if linear or need lookup table
   - Note any dead zones or non-linear regions

### Step 5: Servo-to-Encoder Mapping

1. **Set servo to 0 degrees**
   - Command servo to position 0
   - Record encoder count
   - This is servo-encoder offset

2. **Set servo to 90 degrees**
   - Command servo to position 90
   - Record encoder count
   - Calculate counts per servo degree

3. **Set servo to 180 degrees**
   - Command servo to maximum position
   - Record encoder count
   - Verify linearity of response

4. **Calculate scale factor**
   - Scale = (encoder_max - encoder_min) / (servo_max - servo_min)
   - This converts servo positions to encoder counts

## Calibration Data Sheet

Record this data for each finger:

| Parameter | Thumb | Index | Middle | Ring | Pinky |
|-----------|-------|-------|--------|------|-------|
| Encoder Zero (counts) | | | | | |
| Encoder Max (counts) | | | | | |
| Encoder Range (counts) | | | | | |
| Servo at Zero (degrees) | | | | | |
| Servo at Max (degrees) | | | | | |
| Scale Factor (counts/degree) | | | | | |
| Direction (1 or -1) | | | | | |

## Advanced Calibration

### Non-Linear Correction

If encoder response is non-linear:

1. **Create lookup table**
   - Measure 10-20 points across range
   - Store angle-to-count pairs
   - Use interpolation between points

2. **Polynomial fitting**
   - Fit 2nd or 3rd order polynomial
   - Calculate coefficients
   - Apply correction in software

### Hysteresis Compensation

Account for mechanical backlash:

1. **Measure closing direction**
   - Move finger from open to closed
   - Record encoder counts at positions

2. **Measure opening direction**
   - Move finger from closed to open
   - Record encoder counts at same positions

3. **Calculate hysteresis**
   - Difference between closing and opening
   - Typically 2-5% of range
   - Compensate in control algorithm

## Storing Calibration Data

### Option 1: EEPROM Storage

```cpp
#include <EEPROM.h>

struct CalibrationData {
  int encoderZero[5];
  int encoderMax[5];
  float scaleFactor[5];
  int servoDirection[5];
};

void saveCalibration(CalibrationData &cal) {
  EEPROM.put(0, cal);
}

void loadCalibration(CalibrationData &cal) {
  EEPROM.get(0, cal);
}
```

### Option 2: Configuration File

Save to SD card or send to host computer:
```
# RUKA-V2 Calibration Data
# Date: 2026-02-04
[THUMB]
encoder_zero=0
encoder_max=4850
scale_factor=32.5
direction=1

[INDEX]
encoder_zero=0
encoder_max=5200
...
```

## Verification

### Test Calibrated System

1. **Position accuracy test**
   - Command finger to specific position (e.g., 45°)
   - Verify encoder reads expected count
   - Repeat for multiple positions
   - Accuracy should be ±2-5% of range

2. **Repeatability test**
   - Move finger to position 10 times
   - Check encoder count consistency
   - Should vary by <2% of range

3. **Full range test**
   - Move through full range slowly
   - Verify smooth encoder tracking
   - No jumps or discontinuities

## Troubleshooting

### Encoder Count Drifts

**Symptoms**: Zero position changes over time

**Solutions:**
- Check for loose mechanical connections
- Verify encoder mounting is secure
- Check for electrical noise
- Add shielding to encoder wires

### Non-Linear Response

**Symptoms**: Encoder doesn't track linearly with position

**Causes:**
- Tendon slippage
- Mechanical binding at certain angles
- Worn components

**Solutions:**
- Adjust tendon tension
- Check for mechanical interference
- Use lookup table for correction

### Inconsistent Readings

**Symptoms**: Different counts for same position

**Causes:**
- Mechanical backlash
- Electrical noise
- Loose connections

**Solutions:**
- Tighten mechanical connections
- Add pull-up resistors to encoder signals
- Shield encoder cables
- Filter in software

### Encoder Counts Backwards

**Symptoms**: Count decreases when should increase

**Solution:**
- Swap encoder A and B channels
- Or multiply by -1 in software (set direction to -1)

## Calibration Frequency

- **Initial**: Complete calibration after assembly
- **After maintenance**: Re-calibrate after repairs or adjustments
- **Periodic**: Check calibration monthly for accuracy
- **As needed**: Re-calibrate if performance degrades

## Next Steps

After encoder calibration is complete, proceed to motion checking and tuning.

[Continue to Motion Check →](motion-check.md)

---

[← Previous: Power Testing](../electronics/power-test.md) | [← Back to Main Documentation](../README.md)
