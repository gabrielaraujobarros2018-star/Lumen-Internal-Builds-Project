# FIRMWARE.md
Lumen OS – Firmware Architecture and Policy

This document defines how Lumen OS **interacts with, depends on, and constrains**
device firmware on the Samsung Galaxy A05s (Snapdragon 680).

Firmware is treated as **immutable infrastructure**, not part of the OS.

---

## 1. Definition of Firmware in Lumen

In Lumen, *firmware* refers to all binary components that:
- Execute outside the main OS control
- Are required for hardware functionality
- Cannot be safely replaced or re-signed

Firmware is **not** part of Lumen’s trust boundary, but it *is* part of its runtime reality.

---

## 2. Firmware Philosophy

- Firmware is **opaque**
- Firmware is **assumed correct**
- Firmware is **never modified**
- Firmware is **never reverse-engineered internally**

Lumen adapts to firmware behavior; firmware never adapts to Lumen.

---

## 3. Firmware Components Inventory

### Qualcomm / Samsung Firmware (Untouched)

| Component        | Role |
|------------------|------|
| PBL              | Primary boot ROM |
| XBL              | Secondary bootloader |
| XBL_CONFIG       | Boot configuration |
| ABL              | Android bootloader |
| TZ               | TrustZone / secure services |
| HYP              | Hypervisor |
| MODEM            | Cellular baseband |
| DSP              | Audio, sensor, compute offload |
| PMIC firmware    | Power and charging |

These components remain **byte-identical** to stock firmware.

---

## 4. Firmware Ownership Boundary

| Layer            | Owned by |
|------------------|----------|
| Boot ROM         | Qualcomm |
| Secure boot      | Qualcomm / Samsung |
| Firmware blobs   | Qualcomm / Samsung |
| Linux kernel     | Lumen |
| Userspace        | Lumen |

Lumen starts **after** firmware initialization is complete.

---

## 5. Boot Flow (Simplified)

1. Qualcomm PBL executes
2. XBL loads firmware components
3. TrustZone and HYP initialized
4. ABL verifies boot image
5. Lumen kernel is loaded
6. Lumen userspace starts

Firmware is fully active **before** Lumen executes.

---

## 6. Verified Boot & AVB

Policy:
- AVB verification may be disabled or permissive
- vbmeta is controlled by the developer
- Firmware-level secure boot is not bypassed

Lumen does **not** attempt to break hardware root-of-trust.

---

## 7. Modem Firmware Interaction

- Modem firmware runs independently
- Communication via:
  - QMI
  - QRTR
  - IPC Router
- No direct memory access
- No firmware patching

Failure isolation:
- Modem crashes must not crash Lumen
- Modem restarts must be tolerated

---

## 8. GPU Firmware Interaction

- GPU firmware is loaded by kernel driver
- Userspace communicates via standard ioctl paths
- Vendor blobs are required

Fallback:
- Software rendering allowed
- No firmware dependency for early bring-up

---

## 9. Power & Thermal Firmware

- PMIC firmware controls charging and safety
- Thermal firmware enforces hard limits

Lumen policy:
- Never override thermal kill paths
- Never bypass charging safety
- Respect firmware-imposed caps

Hardware safety overrides OS intent.

---

## 10. TrustZone & Secure Services

TrustZone provides:
- Key storage
- Secure timers
- Crypto acceleration
- DRM primitives

Lumen:
- Uses exposed interfaces only
- Does not introspect secure world
- Treats failures as non-fatal where possible

---

## 11. Firmware Updates

Policy:
- Firmware updates are **manual**
- No OTA firmware flashing by Lumen
- Firmware version is pinned per build

Reason:
- Prevent accidental hard-bricks
- Preserve reproducibility

---

## 12. Firmware Debugging Rules

Allowed:
- Logging firmware-facing kernel calls
- Timing analysis
- Failure observation

Forbidden:
- Binary patching
- Instruction-level reversing
- Fault injection into firmware

---

## 13. Firmware Failure Handling

Lumen must handle:
- Modem unavailability
- GPU driver failure
- Secure service denial

Graceful degradation is mandatory.

Examples:
- No cellular → data disabled
- No GPU → software UI
- No keymaster → reduced security mode

---

## 14. Firmware as a Contract

Firmware defines:
- What hardware exists
- What is allowed
- What is forbidden

Lumen conforms to this contract.
Breaking it risks permanent device loss.

---

## 15. Portability Implications

Because firmware is device-specific:
- Lumen kernel is device-specific
- Lumen userspace is portable
- Firmware abstractions must be isolated

Porting effort focuses on kernel + DT.

---

## 16. Completion Criteria

Firmware integration is considered complete when:
- Lumen boots reliably
- Firmware components remain untouched
- Hardware operates within safe limits
- No firmware regressions observed

---

## 17. Final Rule

Firmware is **infrastructure**, not software.

Lumen runs *on top of reality*, not instead of it.
