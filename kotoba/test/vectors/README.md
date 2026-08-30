# Fixture vectors

Hex strings only. No USB capture, no live device.

| File | Source |
| --- | --- |
| `zero_no_intensity.hex` | YDLidar-SDK Communication Protocol, no-intensity example (`PH=0x55AA`, `CT=1`, `LSN=1`, `FSA=LSA=0xAE53`, `CS=0x54AB`, `S0=0`). |
| `zero_intensity.hex` | Same packet with a 3-byte intensity sample (`I0=0`, `S0=0`). Checksum is unchanged because both extra fields are zero. |
| `two_sample_triangle.hex` | Protocol first-level angle example (`FSA=0x6FE5`, `LSA=0x79BD`) with triangle distances 1000 mm and 8000 mm (`Si = mm * 4`). |
| `zero_scan_freq.hex` | Zero packet with `CT = (100 << 1) \| 1` so `SF = (CT >> 1) / 10 = 10.0` Hz. |
| `intensity_two_sample.hex` | Protocol intensity sample `1F E5 6F` (intensity 287, distance 7161 mm) plus a zero sample. |
| `ans_device_info.hex` | Answer header `A5 5A` + size 20 + type `0x04` and a 20-byte `device_info` body. |
| `ans_health.hex` | Answer header size 3 + type `0x06` and a healthy `0x00 0x00 0x00` body. |

Checksums follow the protocol XOR order (`PH`, `FSA`, samples, `[CT\|LSN]`, `LSA`).
