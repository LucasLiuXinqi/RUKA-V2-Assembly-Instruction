# Calibration: Motion Check

After encoder calibration, this guide helps you verify and tune the motion control system to ensure smooth, accurate finger movements.

## Overview

Motion checking validates:
- Smooth finger articulation
- Accurate position control
- Appropriate response speed
- Absence of oscillations or jitter
- Coordinated multi-finger movements

## Prerequisites

- [ ] Mechanical assembly complete
- [ ] Electronics tested and working
- [ ] Encoder calibration complete
- [ ] Control software loaded
- [ ] Safety procedures reviewed

## Required Equipment

- Calibrated RUKA-V2 hand
- Computer with serial connection
- Stopwatch or timer
- Ruler or caliper (optional)
- Camera for recording (optional)
- Notebook for observations

## Motion Control Basics

### Control Loop

```
Desired Position → Controller → Servo → Finger Movement
                      ↑                       ↓
                      └───── Encoder ─────────┘
```

### Key Parameters

- **Position gain (Kp)**: How aggressively to correct position error
- **Velocity limit**: Maximum movement speed
- **Acceleration limit**: Maximum rate of speed change
- **Deadband**: Acceptable position error before correction

## Basic Motion Tests

### Test 1: Single Finger Movement

**Objective**: Verify individual finger control

**Procedure:**

1. **Command finger to target position**
   ```cpp
   void testSingleFinger(int finger, int targetAngle) {
     Serial.print("Testing finger ");
     Serial.println(finger);
     moveFingerTo(finger, targetAngle);
     delay(1000);
   }
   ```

2. **Test each finger individually**
   - Thumb: 0° → 90° → 0°
   - Index: 0° → 90° → 0°
   - Middle: 0° → 90° → 0°
   - Ring: 0° → 90° → 0°
   - Pinky: 0° → 90° → 0°

3. **Observe and record:**
   - Movement is smooth (yes/no)
   - Reaches target (yes/no)
   - Time to target (seconds)
   - Overshoot (yes/no)
   - Oscillation (yes/no)

**Success Criteria:**
- ✅ Smooth motion throughout range
- ✅ Reaches target within ±5°
- ✅ No overshoot >10%
- ✅ No oscillation or hunting

### Test 2: Multi-Position Movement

**Objective**: Test accuracy across range

**Procedure:**

1. **Move finger to multiple positions**
   - 0° (fully extended)
   - 30° (slightly closed)
   - 60° (half closed)
   - 90° (fully closed)

2. **For each position:**
   - Measure time to reach
   - Verify position with encoder
   - Check for stability
   - Note any issues

**Data Collection:**

| Position | Target Count | Actual Count | Error | Settle Time |
|----------|--------------|--------------|-------|-------------|
| 0° | | | | |
| 30° | | | | |
| 60° | | | | |
| 90° | | | | |

### Test 3: Speed Control

**Objective**: Verify velocity limiting works

**Procedure:**

1. **Fast movement command**
   - Command 0° → 90° rapidly
   - Measure time to complete
   - Typical: 0.5-1.0 seconds

2. **Slow movement command**
   - Command 0° → 90° with reduced speed
   - Should take proportionally longer
   - Typical: 2-3 seconds for slow

3. **Verify speed limit**
   - Movement should not exceed set limit
   - Should be smooth, not jerky
   - No sudden jumps

### Test 4: Direction Reversal

**Objective**: Test response to direction changes

**Procedure:**

1. **Start moving in one direction**
   - Command finger to 90°
   - While moving, command to 0°

2. **Observe response:**
   - Should reverse smoothly
   - No sudden stops or jerks
   - No overshoot at reversal

3. **Repeat multiple times**
   - Test consistency
   - Check for mechanical issues

### Test 5: Coordinated Motion

**Objective**: Test multi-finger coordination

**Procedure:**

1. **Sequential movement**
   - Close fingers one at a time
   - Thumb → Index → Middle → Ring → Pinky
   - Then reverse

2. **Simultaneous movement**
   - Close all fingers together
   - Should reach targets at similar time
   - No interference between fingers

3. **Grip patterns**
   - Test common grips:
     - Pinch (thumb + index)
     - Precision (thumb + index + middle)
     - Power grip (all fingers)
     - Open hand (all extended)

## Advanced Motion Tests

### Test 6: Step Response

**Objective**: Characterize system response

**Procedure:**

1. **Command step input**
   - Sudden change from 0° to 45°
   - Record encoder position vs time

2. **Analyze response:**
   - Rise time (10% to 90%)
   - Settling time (within ±2% of target)
   - Overshoot percentage
   - Oscillations (if any)

3. **Ideal response characteristics:**
   - Rise time: 200-500ms
   - Settling time: <1 second
   - Overshoot: <10%
   - No sustained oscillation

### Test 7: Tracking Performance

**Objective**: Test continuous motion tracking

**Procedure:**

1. **Slow sinusoidal input**
   ```cpp
   void sinusoidalTest(int finger) {
     float t = 0;
     while(t < 10) {  // 10 seconds
       float angle = 45 + 45 * sin(t);
       moveFingerTo(finger, angle);
       t += 0.1;
       delay(100);
     }
   }
   ```

2. **Monitor tracking error**
   - Difference between commanded and actual
   - Should be small (<5° typical)
   - Consistent throughout motion

3. **Try different frequencies**
   - Slow (0.1 Hz): Should track perfectly
   - Medium (0.5 Hz): Should track well
   - Fast (1.0 Hz): May have lag

### Test 8: Load Response

**Objective**: Test behavior under load

**Procedure:**

1. **Add external load**
   - Manually resist finger closing
   - Apply gentle, constant pressure

2. **Observe response:**
   - Controller should increase servo effort
   - Position should be maintained
   - No oscillation under load

3. **Release load**
   - Finger should settle quickly
   - No overshoot after release

## Control Tuning

### If Motion is Too Slow

**Symptoms:**
- Takes too long to reach position
- Sluggish response

**Solutions:**
1. Increase velocity limit
2. Increase position gain (Kp)
3. Check for mechanical binding
4. Verify servo power supply adequate

### If Motion Overshoots

**Symptoms:**
- Finger goes past target then corrects
- Oscillates around target

**Solutions:**
1. Reduce position gain (Kp)
2. Add damping (derivative term)
3. Reduce velocity limit
4. Check for mechanical looseness

### If Motion Oscillates

**Symptoms:**
- Continuous back-and-forth movement
- Never settles at target

**Solutions:**
1. Significantly reduce position gain (Kp)
2. Add or increase damping (Kd)
3. Increase deadband
4. Check encoder connections
5. Reduce velocity limit

### If Position Error is Large

**Symptoms:**
- Doesn't reach commanded position
- Consistent offset from target

**Solutions:**
1. Verify encoder calibration
2. Check tendon tension
3. Verify scale factor is correct
4. Check for mechanical slippage

## PID Tuning Guide

### Basic PID Structure

```cpp
float error = targetPosition - currentPosition;
float pTerm = Kp * error;
float dTerm = Kd * (error - lastError);
output = pTerm + dTerm;
```

### Tuning Procedure

1. **Start with Kp only** (Kd = 0)
   - Set Kp low (e.g., 0.5)
   - Increase until slight oscillation
   - Back off by 50%

2. **Add derivative term**
   - Set Kd to Kp/10
   - Increase if still oscillates
   - Should reduce overshoot

3. **Fine tune**
   - Adjust Kp and Kd together
   - Test with various movements
   - Document final values

**Typical values for RUKA-V2:**
- Kp: 0.5 - 2.0
- Kd: 0.05 - 0.2

## Motion Quality Checklist

A well-tuned system exhibits:

- [ ] Smooth, controlled movements
- [ ] Reaches target quickly (0.5-1s for full range)
- [ ] Minimal overshoot (<5% of range)
- [ ] No sustained oscillation
- [ ] Responds well to direction changes
- [ ] Accurate positioning (±2-5° or better)
- [ ] Consistent behavior across all fingers
- [ ] Stable under light loads
- [ ] Coordinated multi-finger motion

## Performance Metrics

Record these metrics for documentation:

| Metric | Target | Actual | Pass/Fail |
|--------|--------|--------|-----------|
| Position accuracy | ±5° | | |
| Settling time | <1.0s | | |
| Overshoot | <10% | | |
| Rise time | 0.2-0.5s | | |
| Max velocity | Set by design | | |
| Repeatability | ±2° | | |

## Common Issues and Solutions

### Jerky Movement

**Causes:**
- Low control update rate
- Velocity limit too high
- Mechanical binding

**Solutions:**
- Increase control loop frequency
- Smooth velocity commands
- Check mechanical assembly

### Inconsistent Behavior

**Causes:**
- Variable friction
- Tendon slippage
- Power supply issues

**Solutions:**
- Lubricate moving parts
- Re-tension tendons
- Check power supply capacity

### Slow Response

**Causes:**
- Low gains
- Insufficient servo power
- Mechanical resistance

**Solutions:**
- Increase Kp
- Check power supply voltage
- Reduce mechanical friction

### Cannot Hold Position

**Causes:**
- Low holding torque
- Servo not strong enough
- Mechanical slip

**Solutions:**
- Increase servo power
- Reduce load
- Check tendon attachment

## Safety Considerations

During motion testing:

- ⚠️ Keep hands clear of moving fingers
- ⚠️ Don't force fingers beyond natural range
- ⚠️ Monitor for overheating servos
- ⚠️ Be ready to cut power if needed
- ⚠️ Don't run at maximum speed continuously

## Documentation

Record all tuning parameters:

```
RUKA-V2 Motion Tuning Data
Date: YYYY-MM-DD
Firmware Version: X.X

Finger: Thumb
  Kp: 1.2
  Kd: 0.08
  Velocity Limit: 200 counts/sec
  Deadband: 5 counts
  Notes: Slightly increased Kd to reduce overshoot

[Repeat for each finger]
```

## Next Steps

With motion control verified and tuned:

1. **Save configuration** - Store all parameters
2. **Test applications** - Try various grip patterns and tasks
3. **Performance testing** - Measure actual task performance
4. **Continuous improvement** - Monitor and adjust as needed

For any issues, refer to the [Troubleshooting Guide](../troubleshooting.md).

## Success!

If all motion checks pass, your RUKA-V2 is fully calibrated and operational! 🎉

---

[← Previous: Encoder Calibration](encoders.md) | [← Back to Main Documentation](../README.md)
