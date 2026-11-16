# Audio Subsystem Architecture 0.1

**Status:** Draft proposal. This document defines architecture and required contracts; it does not claim that an audio server, mixer, codec stack, or hardware driver is implemented.

## 1. Purpose

Define a userspace-first audio system for Wicofe OS with explicit device ownership, stream lifecycle, format negotiation, routing, latency and clock behavior. The architecture must support ordinary mixed desktop audio while preserving an optional, verifiable bit-perfect path for compatible playback devices.

## 2. Repository ownership

| Component | Responsibility |
|---|---|
| Wicofe-Kernel | Privileged device, memory, interrupt, scheduling and synchronization primitives |
| Wicofe-Drivers | Audio controller, codec, USB Audio Class and device-specific driver support |
| Wicofe-ABI | Versioned userspace device, stream and synchronization contracts |
| Wicofe-Audio | Userspace device service, policy, routing, mixing, stream APIs and diagnostics |
| WDE | Desktop volume, mute, output selection and audio settings UI |
| Wicofe-Services | Service lifecycle, activation and system integration |
| Wicofe-Specs | Cross-repository protocol and compatibility decisions |
| Wicofe-Tests | Conformance, latency, format and hardware regression tests |

Hardware register access and interrupt handling do not belong in the userspace audio service. Desktop UI does not own the audio engine or device policy.

## 3. Layered architecture

1. **Hardware/driver layer:** exposes device capabilities, endpoints, supported formats, clocking and transport-specific events.
2. **Kernel interface:** provides safe device access, memory/buffer mapping, scheduling and synchronization. The exact public contract is owned by Wicofe-ABI.
3. **Audio device service:** enumerates endpoints, tracks hotplug, opens/closes hardware streams and reports capabilities.
4. **Audio engine/policy:** routes streams, mixes when requested, applies per-stream or system processing, and manages default devices and priorities.
5. **Client API:** provides application-facing stream creation, negotiation, buffering, timestamps and error notifications.
6. **WDE integration:** exposes controls and status through service APIs; WDE must not directly manipulate hardware.

The service should be restartable without requiring a kernel restart. Device loss and service restart must have defined client-visible outcomes.

## 4. Device and endpoint model

Separate a physical device from its logical endpoints. A device may expose multiple playback/capture endpoints and alternate configurations. Each endpoint should report:
- direction (playback or capture);
- supported sample formats and valid bit depths;
- supported sample rates or ranges;
- channel count/layout;
- period and buffer constraints;
- clock domain and synchronization capabilities where known;
- transport and driver identity;
- current availability and access mode.

Use stable identifiers derived from a documented identity policy, not transient enumeration order. Friendly names are presentation metadata, not identity.

Hotplug must generate ordered add/remove/change events. On removal, outstanding streams become disconnected and must return a defined error; stale handles must never silently bind to a different device.

## 5. Stream negotiation and lifecycle

A stream lifecycle should cover create, capability query, parameter negotiation, buffer setup, prepare, start, running, drain/stop, reset and close. State transitions and legal operations must be specified before a stable client API is published.

Negotiation must distinguish:
- requested parameters;
- parameters accepted by the engine;
- parameters actually programmed into hardware;
- any conversion inserted into the path.

Do not silently claim native-rate playback when resampling occurred. Unsupported parameters should produce an explicit error or an explicitly negotiated fallback.

## 6. Formats, rates and channel layouts

The format model must distinguish container width from valid sample bits (for example, 24 valid bits in a 32-bit container). Define signedness, endianness, interleaving/planar layout, channel positions and alignment.

The initial supported matrix is an implementation decision and must be reported from actual driver capabilities. Do not promise universal sample rates, bit depths or channel counts. Conversion and channel remixing must be explicit stages with measurable behavior.

## 7. Mixing, routing and processing

The shared desktop path may mix multiple clients, apply volume controls and perform format/rate conversion. Policy should define default output, per-application routing, capture source selection, mute/volume semantics and priority handling.

The engine must make processing stages inspectable. A path that mixes, resamples, changes bit depth, applies DSP or alters sample values is not bit-perfect.

## 8. Bit-perfect and direct playback

Provide a distinct exclusive/direct mode only when the driver and device can guarantee the requested format and clock configuration. The service must:
- negotiate the exact device format/rate/channel layout;
- avoid software mixing, resampling, sample-rate conversion, DSP, volume scaling and format conversion on that path;
- expose the actual negotiated hardware parameters;
- report whether the path is bit-perfect, rather than infer it from a user preference;
- define exclusivity, preemption, device loss and fallback behavior.

A direct path does not by itself prove bit-perfect output. Validation requires checking the complete data path and device behavior. USB DAC support must be evaluated per USB Audio Class version, alternate setting, endpoint synchronization mode, feedback handling, packet scheduling and device quirks. Do not claim support for every USB DAC from generic USB enumeration alone.

## 9. Clocking, buffering and latency

Document the clock source for each stream and how asynchronous devices are reconciled. Buffer and period sizes must be negotiated within driver constraints. Timestamps should identify their clock domain and meaning (capture time, presentation time or software observation).

Define latency targets by use case rather than one universal number. Handle underrun, overrun, drift, missed deadlines and device clock changes with explicit counters/events. Real-time scheduling must be bounded and must not permit an unprivileged client to starve the system.

## 10. Security and permissions

- Gate capture access through explicit session/user permissions and indicators.
- Validate stream parameters and buffer sizes with overflow-safe arithmetic.
- Isolate client buffers and device handles.
- Define which clients may request exclusive mode or change system routing.
- Clean up streams on client exit, service restart and device removal.
- Avoid exposing raw device control to ordinary applications.

## 11. Power, lifecycle and recovery

Define suspend/resume behavior, device reinitialization, default-device changes and service crash recovery. A stream must not continue to report running when the hardware is unavailable. Recovery may require client renegotiation; do not silently change a stream's format or route without an event.

## 12. Testing and validation

Required test layers:
- unit tests for format negotiation, channel maps, conversion and state transitions;
- mock-device tests for hotplug, disconnect, unsupported formats and clock drift;
- loopback tests for routing, mixing, mute and volume policy;
- latency and xruns measurement under load;
- bit-perfect verification using known test vectors and a documented capture/measurement method;
- USB Audio Class tests across isochronous endpoint modes and feedback behavior;
- emulator tests where meaningful and hardware tests for every claimed device;
- fuzzing of client requests and device descriptors.

Report build, emulator, loopback and physical-hardware evidence separately. A successful stream open is not proof of bit-perfect output or low-latency reliability.

## 13. Open decisions

- Canonical stream/device ABI and whether it is exposed through file descriptors, capability handles or a service IPC protocol.
- Initial audio engine and third-party dependencies/license compatibility.
- Initial PCM format/rate/channel support matrix.
- Mixer policy, per-application routing and session integration.
- Exclusive-mode arbitration and permissions.
- Clock/timestamp model and drift correction.
- USB Audio Class versions and supported synchronization modes.
- Codec/firmware policy, suspend/resume and real-time scheduling requirements.
- Objective latency targets and bit-perfect validation hardware/method.

## 14. Completion criteria

Implementation should begin only after Wicofe-ABI, Wicofe-Kernel, Wicofe-Drivers, Wicofe-Services and Wicofe-Specs agree on device/stream ownership, buffer and clock semantics, error handling, permission model and conformance tests. All claimed format, latency and bit-perfect capabilities must be backed by reproducible test evidence.
