# Phase 1 – Underlay prüfen

Vor der L3VPN-Erweiterung müssen IPv6-Transport und uSID-Funktion von PE zu PE bereits funktionieren.

Auf **XRv-1 und XRv-10**:

```cisco
show version
show ipv6 interface brief
show isis neighbors
show route ipv6 2001:db8:1:1::10/128
show route ipv6 2001:db8:1:1::1/128
show segment-routing srv6 locator detail
show running-config segment-routing
```

Von XRv-1:

```cisco
ping ipv6 2001:db8:1:1::10 source 2001:db8:1:1::1
traceroute ipv6 2001:db8:1:1::10
```

Von XRv-10 in Gegenrichtung mit getauschten Adressen.

## Erwartetes Ergebnis

- Alle notwendigen IS-IS-Nachbarschaften stehen.
- Loopback und Locator des jeweils anderen PE sind per IPv6 erreichbar.
- `LABlocator` hat Status **Up**, Präfix für XRv-1 `5f00:0:1::/48` und für XRv-10 `5f00:0:10::/48`.
- Der Locator zeigt uSID-Micro-Segment-Verhalten (uN); der vorhandene Flex-Algo-Locator darf daneben bestehen.

!!! warning "Stoppkriterium"
    Ohne funktionierenden IPv6-Transport zwischen beiden PEs **kein** L3VPN debuggen. Zuerst Underlay reparieren.

Die vorhandenen IS-IS-, Flex-Algo- und TI-LFA-Konfigurationen müssen für die erste VPN-Phase nicht geändert werden.
