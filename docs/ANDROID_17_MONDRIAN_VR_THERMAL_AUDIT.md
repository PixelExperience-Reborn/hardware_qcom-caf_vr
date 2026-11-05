# PE-Reborn Android 17 — Qualcomm VR HAL source audit (mondrian)

**2026-11-05 | P2: hardware/qcom-caf/vr | historical source preserved, NOT enabled.**

## Exact historical source

- PE `seventeen` at `a2c6ad6f6443dee7ec4b48f788a2c8f7ee351aa2` is historical Qualcomm VR support, with latest code commit in September 2019.
- `Android.mk` defines `vr.$(TARGET_BOARD_PLATFORM)`, gated by the legacy `project-path-for(qcom-vr)` path map; conditionally adds SoC implementation files for **msm8998** and **sdm845** only. Mondrian uses **sm8475 / sm8450 family**, not either declared board variant.
- `vr.c` uses the legacy `hardware/vr.h` module entry point and dynamically loads `vendor/lib64/libthermalclient.so`, then calls the `thermal_client_config_query`/`thermal_client_config_set`/`thermal_client_config_cleanup` functions. These are unrelated to the modern AIDL v3 Thermal HAL source contract.
- `vr-845.c` hard-codes legacy thermal profiles `SKIN-HIGH-FLOOR`, `SKIN-MID-FLOOR`, `SKIN-LOW-FLOOR`, `VIRTUAL-SS-GPU-SKIN`, and `VR-EMMC`. It is **not** a validated sm8450 thermal profile; the old code can disable/enable thermal profiles when VR mode changes.

## Real device consumer check

The existing `LF52406/mondrian_android_device_xiaomi_sm8450-common` `lineage-24.0` `common.mk` was reviewed: its product package list includes modern `android.hardware.thermal-service.qti`, but **does not explicitly include `vr.$(TARGET_BOARD_PLATFORM)`**. Code search of the two cited mondrian/common repositories did not find a clear VR source consumer; this is not proof against every installed vendor blob or optional VR app.

Android 17 CDD treats high-performance VR requirements as conditional on a device supporting the VR mode; it does **not** require every smartphone to ship this legacy Qualcomm VR library. Do not infer support simply from the old `hardware/vr.h` or HIDL interface.

## Decision and gates

**Keep this repo as an archive/reference for original PE behavior; do not merge or add to staged AOSP17 profiles.** No compatibility shim, `libThermal_headers` substitute or artificial `vr.taro` package is justified.

Before reconsidering:
1. Identify an actual mondrian product/VINTF/vendor blob/VR app consumer, not merely a source directory.
2. Verify the complete Android17 VR service API/HAL path and any vendor bridge.
3. Establish that all named legacy thermal profiles exist on mondrian and that thermal protections remain effective; never disable mitigations as an unconditional VR optimization.
4. Validate module ABI, required proprietary symbols, SELinux enforcing, Android17 Soong build and device runtime.

**No AOSP17 compile, VR runtime or thermal-safety test has been performed.** Historical `seventeen` remains unmodified. This is a source-dependency audit, not an implementation or success claim.

**Next P2 matrix row:** `hardware/qcom-caf/wlan`.
