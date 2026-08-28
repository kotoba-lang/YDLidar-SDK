# Kotoba binding

In-language YDLidar scan/sample codec and command framing.

Kotoba cannot FFI the C++ driver. v1 is a protocol subset: hex strings in,
hex strings and records out. The host owns the serial or TCP port. This
directory does not open USB, a filesystem, or a network capability; the
default empty policy (`:policy/allow #{}`) is enough.

It sits next to [`python/`](../python/) and [`csharp/`](../csharp/). Those
bindings wrap the C++ SDK. This one does not.

Protocol authority: [YDLidar SDK Communication Protocol](../doc/YDLidar-SDK-Communication-Protocol.md).

## What this is

- Command frames: `0xA5` + cmd, optional payload + XOR (`encode-cmd`,
  `encode-cmd-payload`, `cmd-from-hex`)
- Answer headers: `0xA5 0x5A` + 30-bit size + 2-bit subtype + type
  (`encode-ans-header`, `decode-device-info`, `decode-health`)
- Scan packets: `PH=0x55AA`, `CT`, `LSN`, `FSA`, `LSA`, `CS`, 2-byte or
  3-byte samples (`decode-scan`, `encode-scan`, `checksum-ok?`)
- Triangle distance `Si / 4` (integer millimetres), intensity
  `((S1 & 0x03) << 8) | S0`, first-level angle `(FSA >> 1) / 64` stored as
  Q6 ticks and millidegrees
- Zero-packet scan frequency `SF = (CT >> 1) / 10` as decihertz

It is not a session client. Timeouts, reconnect, and motor DTR stay on the
host. Second-level triangle `atan` correction is out of v1 (no admitted
float/`atan` on this surface).

## Encode and decode

```clojure
(ns hello (:export [main]))

(defn main [] :string
  (encode-cmd (cmd-scan)))
;; => A560
```

```clojure
(ns hello (:export [main]))

(defn main [] :i64
  (if (checksum-ok? "AA55010153AE53AEAB540000")
    (sample-distance-mm "AA55010153AE53AEAB540000" 0)
    -1))
;; => 0
```

`examples/encode_scan_cmd.kotoba` and `examples/decode_zero_packet.kotoba`
are those two programs.

## Kotoba surface (verified on kotoba v0.7.2)

Language authority: [kotoba-lang/kotoba-lang](https://github.com/kotoba-lang/kotoba-lang).
CLI/implementation: [kotoba-lang/kotoba](https://github.com/kotoba-lang/kotoba)
tag **v0.7.2**.

Admitted pieces this library uses:

- typed `ns` / `defn`, documents, keywords, strings, `i64`
- `bit-and`, `bit-xor`, `quot`
- `document`, `document-get`, `document-assoc`, `document-vector-conj`,
  `document-edn-read`, `document-edn-print`
- `string-concat`, `string-substring`, `string-code-point-at`, `string=?`,
  `string-from-i64`

`require` is forbidden, so `ydlidar` is one compilation unit.
`kotoba run` on this source hits the EDN-IR adapter, which does not implement
documents or `string=?`. Compile to wasm or restricted ESM instead.

## Build

```sh
kotoba compile src/ydlidar.kotoba --target wasm --output ydlidar.wasm --json
```

Accept `kotoba.cli/ok?` true and `kotoba.cli/code` `emitted`. Fixture
execution uses `--target web` and `instantiateKotoba` (empty grant set).

```sh
scripts/ci.sh
```

From the SDK tree, `-DBUILD_KOTOBA=ON` (default) adds the `ydlidar_kotoba`
CMake target and `ydlidar_kotoba_fixtures` CTest when the `kotoba` CLI is
on `PATH`.

Install the CLI from the v0.7.2 release tarball or
`brew tap kotoba-lang/kotoba && brew trust kotoba-lang/kotoba && brew install kotoba`.

## Tests

`test/fixtures.kotoba` covers the protocol examples: official zero packet,
intensity sample `1F E5 6F`, triangle 1000/8000 mm pair, first-level
angles, zero-packet frequency, XOR checksum reject, command/payload
framing, and device-info/health answer headers. Vectors are vendored under
`test/vectors/`. CI does not open a LiDAR.
