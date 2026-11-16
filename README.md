# Wicofe-Audio

Userspace audio services and integration for Wicofe OS.

## Scope

- Audio device discovery and policy.
- Audio routing, mixing, and stream lifecycle.
- User and application audio APIs.
- Device configuration and diagnostics.
- Low-latency playback/capture and audio validation tests.

## Boundaries

Audio hardware drivers and kernel device primitives belong in Wicofe-Kernel or Wicofe-Drivers. Desktop controls may be integrated by WDE, while this repository owns reusable audio services and userspace interfaces.

## Design requirements

Specify supported formats, sample rates, channel layouts, clock handling, buffer policy, latency targets, device hotplug behavior, permissions, and error recovery. Distinguish bit-perfect paths from paths that resample or mix.

## Status

Initial repository baseline; no complete audio server or hardware support is claimed.

## License

Original project code targets MPL-2.0; third-party audio libraries and codecs require license review.

## Documentation

See [docs/INDEX.md](docs/INDEX.md) for this repository's technical documentation. The [central repository documentation map](https://github.com/Waizusora/Wicofe-Docs/blob/main/REPOSITORY_DOCUMENTATION_MAP.md) identifies canonical ownership and cross-repository boundaries.
