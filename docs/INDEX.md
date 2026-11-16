# Audio Documentation Index

This directory documents Wicofe userspace audio services, APIs, and integration.

## Scope
- Audio device discovery and policy.
- Routing, mixing, stream lifecycle, and application-facing APIs.
- Format/rate/channel/clock handling and buffer management.
- Low-latency playback/capture, diagnostics, and validation tests.

## Ownership
Audio hardware drivers and kernel device primitives belong to [Wicofe-Kernel](https://github.com/Waizusora/Wicofe-Kernel) and [Wicofe-Drivers](https://github.com/Waizusora/Wicofe-Drivers). [WDE](https://github.com/Waizusora/WDE) may provide desktop controls; this repository owns reusable audio services and interfaces.

## Audio guarantees
Distinguish bit-perfect paths from resampling/mixing paths. State supported formats, sample rates, channel layouts, clock behavior, latency targets, permissions, hotplug, and recovery behavior.

- [Central documentation index](https://github.com/Waizusora/Wicofe-Docs/blob/main/INDEX.md)
- [Repository documentation map](https://github.com/Waizusora/Wicofe-Docs/blob/main/REPOSITORY_DOCUMENTATION_MAP.md)
