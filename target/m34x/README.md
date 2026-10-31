# UN1CA for Galaxy M34 5G (m34x)

> [!CAUTION]
> **WORK IN PROGRESS / EXPERIMENTAL PORT — DO NOT FLASH AS A DAILY DRIVER!**
> This port is currently under active development. There are critical hardware abstraction layer (HAL) and telephony bugs. Please read the known issues section before attempting anything.

---

## 📱 Device Specifications

| Feature | Details |
| :--- | :--- |
| **Device Model** | Samsung Galaxy M34 5G (`SM-M346B2`, `m34x`, `m34xjxx`) |
| **SoC** | Samsung Exynos 1280 (`s5e8825`) |
| **Architecture** | ARM64-v8a |
| **Target Firmware Base** | `SM-M346B2_TUR` (`M346B2XUS8DYH1` - Bit 8) |
| **Port ROM Base** | Galaxy S22 (`SM-S901B` / `r0s`) — One UI 8.0 (Android 16) |
| **Maintainer** | encipher |

---

## ⚠️ CRITICAL KNOWN ISSUES & BUGS

Do not flash this ROM expecting a daily driver experience. The following issues are present:

### 1. ❌ RIL / Cellular Data Not Working (Subscription ID Mapping Failure)
* **Symptom:** SIM card is detected, operator name (e.g., Türk Telekom) and signal bars appear, and CS/PS registration state is `IN_SERVICE`. However, mobile data never turns on (`rmnet0` stays `DOWN`).
* **Technical Root Cause:**
  * Android 16's `SubscriptionManagerService` relies on `RadioConfig@1.3::getSimSlotsStatus()` to map SIM card ICCID to logical slot 0.
  * Samsung Shannon RIL on Exynos 1280 returns `REQUEST_NOT_SUPPORTED` for `getSimSlotsStatus()`.
  * Consequently, `UiccSlot.mIccId={0=}` remains empty, and Android's telephony registry cannot associate the active SIM with a valid subscription ID (`defaultDataSubId=-1`, `simSlotIndex=-1`).
  * Because `defaultDataSubId` is `-1`, data connection requests are rejected by the framework.

### 2. ⚠️ Camera Compatibility & Sensor Mismatch
* Galaxy M34 5G uses a **50 MP ISOCELL JN1** sensor (`4080x3060`), unlike Galaxy A53's 64 MP sensor (`4624x3468`).
* Applying the common A53 `raw_camera` patch (`libexynoscamera3.so` hex edits) corrupts buffer allocations on M34, causing `m_setNodeInfo: Invalid currentBufferCount(0)` and worker timeouts.
* Galaxy S22's stock `SamsungCamera.apk` crashes with `Resources$NotFoundException: Can't confirm shooting mode enable or not.` because S22 features shooting modes missing on M34.
* **Fix applied in this tree:** Stock M34 `SamsungCamera.apk` is pulled from target firmware, authentic `camera-feature.xml` resolutions are preserved, and blind hex patches are disabled (`target/m34x/patches/raw_camera/disable`).

### 3. ⚠️ Zygote Runtime & Blob Replacement Caveat
* Overriding S22 `libhwui.so`, `audioserver`, or core graphics libraries with M34 stock blobs breaks Zygote linking with `CANNOT LINK EXECUTABLE "app_process64": library "android.hardware.graphics.common-V7-ndk.so" not found`.
* S22 base runtime libraries must be retained.

### 4. 🔒 Bootloader & Fuse Bit 8
* Ensure your device bootloader binary matches Bit 8 (`M346B2XUS8DYH1`). Never flash bootloader binaries (`sboot.bin`, `param.bin`) from other bootloader revisions or devices.

---

## ✅ What Works

* **Boot:** Successfully completes full system boot (`sys.boot_completed=1`).
* **Display:** Native 1080x2340 resolution with 120Hz smooth high refresh rate.
* **UI/UX:** One UI 8.0 Launcher, Setup Wizard, navigation gestures, and animations.
* **Touch & Input:** Full touchscreen and hardware key support.
* **Wireless:** Wi-Fi (2.4GHz / 5GHz) and Bluetooth connectivity.
* **Audio:** Audio playback and media decoding.
* **Camera Viewfinder:** Operational with stock M34 camera APK and feature XML.

---

## 🛠️ Building for Galaxy M34 5G

```bash
# Set up UN1CA environment
./build.sh m34x
```
