# Phase 1 – SRv6-Underlay prüfen

Für **KUNDE_A und KUNDE_B** sind XRv-5 und XRv-10 die Provider Edges. KUNDE_C nutzt XRv-4 und XRv-7.

## Auf XRv-5 und XRv-10

```cisco
show version
show isis neighbors
show segment-routing srv6 locator detail
show running-config interface Loopback0
show route ipv6 2001:db8:1:1::5/128
show route ipv6 2001:db8:1:1::10/128
show running-config segment-routing
```

**Auf XRv-5** (Loopbacks zuerst überprüfen):

```cisco
ping ipv6 2001:db8:1:1::10 source 2001:db8:1:1::5
traceroute ipv6 2001:db8:1:1::10
```

Von XRv-10 in Gegenrichtung testen.

## Erwartung

- IS-IS Level 2 funktioniert und bietet IPv6-Reachability.
- `LABlocator` auf XRv-5 entspricht dem Schema `5f00:0:5::/48`, auf XRv-10 `5f00:0:10::/48`.
- uSID-Locator ist Up; der IPv6-Transport zwischen PE-Loopbacks funktioniert.
- Flex-Algo und TI-LFA im Underlay bleiben unverändert.

!!! danger "Abbruchkriterium"
    Solange das Underlay nicht funktioniert, keine VPN-Konfiguration debuggen. XRv-9000-uDT4-Service-SIDs müssen später zusätzlich im **Forwarding** bestätigt werden.
