# SO101 Leader/Follower Robot Teleoperation

This repository contains complete instructions for **calibrating** and **teleoperating** SO101 leader/follower robotic arms using the `lerobot` package.

---

## Table of Contents

1. [Prerequisites](#prerequisites)
2. [System Overview](#system-overview)
3. [Calibration](#calibration)
   - [Leader Arm Calibration](#leader-arm-calibration)
   - [Follower Arm Calibration](#follower-arm-calibration)
4. [Teleoperation](#teleoperation)
5. [Troubleshooting](#troubleshooting)
6. [File Locations](#file-locations)

---

## Prerequisites

Before starting, ensure you have:

- **Python environment** with `lerobot` installed
  ```bash
  conda activate lerobot
  ```
- **USB connections** for both arms (see System Overview below)
- **Write permissions** to calibration directory:
  ```bash
  mkdir -p ~/.cache/huggingface/lerobot/calibration/
  ```

---

## System Overview

```
┌─────────────────────────────────────────────────────────────┐
│                    TELEOPERATION SETUP                       │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  LEADER ARM                          FOLLOWER ARM           │
│  ├─ Type: so101_leader               ├─ Type: so101_follower│
│  ├─ Port: /dev/ttyACM0               ├─ Port: /dev/ttyACM1  │
│  ├─ ID: my_awesome_leader_arm        ├─ ID: my_awesome_...  │
│  └─ Role: Controller (Input)         └─ Role: Actuator      │
│                                                              │
│          Leader movements  ──────────>  Follower mirrors    │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

### Hardware Connections

| Arm      | Type              | USB Port         | Unique ID                  |
|----------|-------------------|------------------|----------------------------|
| Leader   | `so101_leader`    | `/dev/ttyACM0`   | `my_awesome_leader_arm`    |
| Follower | `so101_follower`  | `/dev/ttyACM1`   | `my_awesome_follower_arm`  |

---

## Calibration

**Important:** Each arm must be calibrated before teleoperation. Calibration data is saved automatically and only needs to be done once (unless you reset the arms).

### Leader Arm Calibration

1. **Connect** the leader arm to `/dev/ttyACM0`

2. **Run calibration command:**
   ```bash
   lerobot-calibrate \
       --teleop.type=so101_leader \
       --teleop.port=/dev/ttyACM0 \
       --teleop.id=my_awesome_leader_arm
   ```

3. **Follow on-screen instructions:**
   - Move the arm to the **middle of its range**
   - Press `ENTER`
   - Move **each joint** (except `wrist_roll`) through its **full range of motion**
   - Press `ENTER` when finished

4. **Verify calibration saved:**
   ```bash
   ls ~/.cache/huggingface/lerobot/calibration/teleoperators/so_leader/
   ```
   You should see: `my_awesome_leader_arm.json`

### Follower Arm Calibration

1. **Connect** the follower arm to `/dev/ttyACM1`

2. **Run calibration command:**
   ```bash
   lerobot-calibrate \
       --robot.type=so101_follower \
       --robot.port=/dev/ttyACM1 \
       --robot.id=my_awesome_follower_arm
   ```

3. **Follow on-screen instructions:**
   - Move the arm to the **middle of its range**
   - Press `ENTER`
   - Move **each joint** through its **full range of motion**
   - Press `ENTER` when finished

4. **Verify calibration saved:**
   ```bash
   ls ~/.cache/huggingface/lerobot/calibration/robots/so_follower/
   ```
   You should see: `my_awesome_follower_arm.json`

---

## Teleoperation

Once both arms are calibrated, start teleoperation:

```bash
lerobot-teleoperate \
    --robot.type=so101_follower \
    --robot.port=/dev/ttyACM1 \
    --robot.id=my_awesome_follower_arm \
    --teleop.type=so101_leader \
    --teleop.port=/dev/ttyACM0 \
    --teleop.id=my_awesome_leader_arm
```

### What Happens

- **Leader arm** acts as the controller (you move it manually)
- **Follower arm** mirrors the leader's movements in real-time
- Press `Ctrl+C` to stop teleoperation

### Key Parameters Explained

| Parameter           | Value                      | Description                          |
|---------------------|----------------------------|--------------------------------------|
| `--robot.type`      | `so101_follower`           | Arm that will be controlled          |
| `--robot.port`      | `/dev/ttyACM1`             | Follower's USB port                  |
| `--robot.id`        | `my_awesome_follower_arm`  | Follower's calibration ID            |
| `--teleop.type`     | `so101_leader`             | Arm used as controller               |
| `--teleop.port`     | `/dev/ttyACM0`             | Leader's USB port                    |
| `--teleop.id`       | `my_awesome_leader_arm`    | Leader's calibration ID              |

---

## Troubleshooting

### 1. Invalid Type Error

**Error:** `Invalid robot type` or `Invalid teleop type`

**Solution:**
- Leader arm must always use type: `so101_leader`
- Follower arm must always use type: `so101_follower`
- During calibration, use `--teleop.type` for leader, `--robot.type` for follower
- During teleoperation, specify both types correctly

### 2. USB Port Issues

**Error:** `Cannot connect to port` or `Permission denied`

**Solution:**

Check connected devices:
```bash
# Linux
ls /dev/ttyACM*

# macOS
ls /dev/tty.usbmodem*
```

Verify correct ports:
```bash
# Check which device is which
dmesg | grep tty
```

Fix permissions (Linux):
```bash
sudo usermod -a -G dialout $USER
# Then log out and log back in
```

### 3. Calibration Not Saving

**Error:** Calibration completes but file doesn't exist

**Solution:**

Check directory permissions:
```bash
ls -la ~/.cache/huggingface/lerobot/calibration/
```

Create directories manually if needed:
```bash
mkdir -p ~/.cache/huggingface/lerobot/calibration/teleoperators/so_leader/
mkdir -p ~/.cache/huggingface/lerobot/calibration/robots/so_follower/
```

### 4. Follower Arm Doesn't Move During Teleoperation

**Possible causes:**

1. **Calibration missing** - Recalibrate both arms
2. **Wrong ports** - Verify USB connections match configuration
3. **IDs don't match** - Ensure calibration IDs match teleoperation command IDs

**Diagnostic steps:**
```bash
# 1. Verify calibration files exist
ls ~/.cache/huggingface/lerobot/calibration/teleoperators/so_leader/
ls ~/.cache/huggingface/lerobot/calibration/robots/so_follower/

# 2. Test ports individually
python -c "import serial; print(serial.Serial('/dev/ttyACM0'))"
python -c "import serial; print(serial.Serial('/dev/ttyACM1'))"
```

### 5. Arm Moves Erratically

**Solution:**
- Recalibrate the arm
- Ensure you moved through the **full range** during calibration
- Check for loose USB connections

### 6. Camera FPS Issues (OpenCV)

**Problem:** Camera only achieves 5 fps instead of requested 30 fps

**Solution:**

If your camera supports MJPG (Motion-JPEG) encoding, specify it in the camera config:

```bash
lerobot-teleoperate \
    --robot.type=so101_follower \
    --robot.port=/dev/ttyACM1 \
    --robot.id=my_awesome_follower_arm \
    --robot.cameras="{ front: {type: opencv, index_or_path: 0, width: 1920, height: 1080, fps: 30, fourcc: 'MJPG'}}" \
    --teleop.type=so101_leader \
    --teleop.port=/dev/ttyACM0 \
    --teleop.id=my_awesome_leader_arm \
    --display_data=true
```

**Check your camera's supported formats:**
```bash
v4l2-ctl --device=/dev/video0 --list-formats-ext
```

Look for `'MJPG'` (Motion-JPEG) entries which typically support higher fps at full resolution. YUYV format is slower and often limited to 5 fps at 1080p.

**Example camera capabilities (USB2.0_CAM1):**

| Format | Resolution | Supported FPS |
|--------|-----------|---|
| **MJPG** | 1920x1080 | ✅ 30, 25, 20, 15, 10, 5 fps |
| **MJPG** | 1440x1080 | ✅ 30, 20, 15, 10, 5 fps |
| **MJPG** | 1280x960 | ✅ 30, 20, 15, 10, 5 fps |
| **MJPG** | 1280x720 | ✅ 30, 20, 15, 10, 5 fps |
| **MJPG** | 800x600 | ✅ 30, 25, 20, 15, 10, 5 fps |
| **MJPG** | 640x480 | ✅ 30, 25, 20, 15, 10, 5 fps |
| YUYV | 1920x1080 | ❌ 5 fps only |
| YUYV | 1280x720 | 10, 5 fps |
| YUYV | 640x480 | ✅ 30, 25, 20, 15, 10, 5 fps |

**Recommended configurations:**
- Full HD: `width: 1920, height: 1080, fps: 30, fourcc: 'MJPG'`
- High speed lower res: `width: 640, height: 480, fps: 30, fourcc: 'MJPG'` (works with both MJPG and YUYV)

---

## File Locations

### Calibration Files

```
~/.cache/huggingface/lerobot/calibration/
├── teleoperators/
│   └── so_leader/
│       └── my_awesome_leader_arm.json
└── robots/
    └── so_follower/
        └── my_awesome_follower_arm.json
```

### What's in a Calibration File?

Each calibration file contains:
- Joint angle ranges (min/max)
- Neutral positions
- Calibration timestamp
- Hardware configuration

**Example structure:**
```json
{
  "shoulder_pan": {"min": -150, "max": 150, "neutral": 0},
  "shoulder_lift": {"min": -90, "max": 90, "neutral": 0},
  ...
}
```

---

## Quick Reference Commands

### Full Workflow

```bash
# 1. Calibrate leader arm
lerobot-calibrate \
    --teleop.type=so101_leader \
    --teleop.port=/dev/ttyACM0 \
    --teleop.id=my_awesome_leader_arm

# 2. Calibrate follower arm
lerobot-calibrate \
    --robot.type=so101_follower \
    --robot.port=/dev/ttyACM1 \
    --robot.id=my_awesome_follower_arm

# 3. Start teleoperation
lerobot-teleoperate \
    --robot.type=so101_follower \
    --robot.port=/dev/ttyACM1 \
    --robot.id=my_awesome_follower_arm \
    --teleop.type=so101_leader \
    --teleop.port=/dev/ttyACM0 \
    --teleop.id=my_awesome_leader_arm
```

---

## Additional Resources

- **lerobot Documentation:** [https://github.com/huggingface/lerobot](https://github.com/huggingface/lerobot)
- **SO101 Hardware Manual:** Check manufacturer documentation
- **Support:** Open an issue in this repository

---

## License

[Add your license information here]

## Contributors

[Add contributor information here]

---

**Ready to get started?** Make sure both arms are connected, then begin with [Calibration](#calibration)!
