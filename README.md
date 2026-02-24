# TAL --- Transport Abstraction Layer for WireGuard

**TAL (Transport Abstraction Layer)** is a lightweight transport shim
that allows WireGuard traffic to be carried over alternate network
transports (e.g., TCP 443) without modifying WireGuard itself.

The project explores adaptive, pluggable transports for personal
sovereign exit nodes operating under adversarial network conditions.

------------------------------------------------------------------------

## Motivation

Commercial VPN providers are easily blocked because they:

-   Operate from known IP ranges\
-   Use predictable protocols\
-   Scale in ways that make fingerprinting trivial

Operating a personal WireGuard server provides sovereignty, but
UDP-based tunnels are often blocked or throttled in restrictive
networks.

TAL addresses this by inserting a programmable transport layer between
WireGuard and the network, enabling:

-   UDP passthrough (baseline mode)\
-   TCP encapsulation (e.g., port 443)\
-   Future pluggable transports (TLS, WebSocket, obfuscation)

WireGuard remains unchanged.

------------------------------------------------------------------------

## Architecture

Baseline WireGuard flow:

    Windows WireGuard
        ↓ UDP
    VPS:51820

With TAL (TCP mode):

    WireGuard (UDP localhost)
        ↓
    TAL client (framing + TCP 443)
        ↓
    Internet
        ↓
    TAL server (decapsulation)
        ↓ UDP
    WireGuard (wg0)

**Key principle:** WireGuard is unaware of the transport layer beneath
it.

TAL only moves opaque encrypted packets.

------------------------------------------------------------------------

## Features (Current Prototype)

-   WireGuard over framed TCP\
-   TCP 443 transport\
-   Automatic reconnect logic\
-   Bidirectional packet statistics\
-   Clean separation between transport and VPN logic\
-   No WireGuard modifications required

------------------------------------------------------------------------

## Current Status

This is a functional prototype.

Implemented:

-   TCP transport mode\
-   Length-prefixed framing\
-   Thread-safe bidirectional forwarding\
-   Reconnect loop for TCP failures\
-   Observability via live packet counters

Not yet implemented:

-   TLS wrapping\
-   WebSocket transport\
-   Multi-transport fallback\
-   Health classification\
-   Multi-peer support\
-   Production hardening

------------------------------------------------------------------------

## Installation

### Requirements

-   Python 3.9+\
-   WireGuard configured normally\
-   VPS with TCP 443 allowed\
-   TCP 443 open in firewall

------------------------------------------------------------------------

## Usage

### Server (VPS)

Run:

``` bash
sudo python3 server.py
```

This:

-   Listens on TCP 443\
-   Forwards framed packets to 127.0.0.1:51820\
-   Relays UDP responses back over TCP

Ensure:

-   WireGuard is running on port 51820\
-   Nothing else is using TCP 443

------------------------------------------------------------------------

### Client (Windows)

Set WireGuard endpoint to:

    Endpoint = 127.0.0.1:51821

Then run:

``` bash
python tal.py
```

Activate WireGuard as usual.

------------------------------------------------------------------------

## Design Principles

1.  Do not modify WireGuard.\
2.  Preserve packet boundaries via explicit framing.\
3.  Maintain transport-layer isolation.\
4.  Keep prototype lightweight and inspectable.\
5.  Prefer clarity over abstraction complexity.

------------------------------------------------------------------------

## Roadmap

### Short-term

-   Transport abstraction interface\
-   UDP + TCP mode switching\
-   Health detection and auto-fallback\
-   Structured logging

### Medium-term

-   TLS-wrapped TCP mode\
-   WebSocket-over-HTTPS transport\
-   Multi-server failover\
-   Clean CLI interface

### Long-term

-   Adaptive censorship classification\
-   Pluggable transport registry\
-   Framework for censorship-resilient personal exit nodes

------------------------------------------------------------------------

## Security Notes

-   WireGuard encryption is end-to-end.\
-   TAL does not inspect or modify packet contents.\
-   TCP mode does not provide TLS encryption.\
-   Production hardening (authentication, rate limiting, TLS) not yet
    implemented.

Use in trusted environments during prototype phase.

------------------------------------------------------------------------

## Disclaimer

This project is experimental.

It is intended for research and personal infrastructure experimentation.

It is not a production VPN replacement.

------------------------------------------------------------------------

## License

(To be determined)
