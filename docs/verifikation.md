# Phase 3 – Verifikation und Fehlersuche

## Test 1: BGP-Control-Plane

Auf beiden PEs:

```cisco
show bgp vpnv4 unicast summary
show bgp vpnv4 unicast
show running-config router bgp 65000
```

**Erwartung:** IPv6-BGP-Nachbarschaft Established. Beide lokalen LAN-Routen sind als VPNv4-Routen verfügbar, RD und Route Target konsistent. Falls `summary` je nach Image anders geparst wird: `show bgp ?` verwenden.

## Test 2: VRF-RIB

```cisco
show route vrf KUNDE_A
show route vrf KUNDE_A 192.168.91.0/24
show route vrf KUNDE_A 192.168.11.0/24
```

Auf XRv-1 sollte `192.168.11.0/24` lokal (statisch) und `192.168.91.0/24` remote (BGP) erscheinen; auf XRv-9 umgekehrt.

## Test 3: SRv6-Service-SID

```cisco
show segment-routing srv6 locator detail
show segment-routing srv6 sid
show bgp vpnv4 unicast
```

**Entscheidend:** Die Service-SID muss dem VPNv4-Präfix zugeordnet und im Forwarding tatsächlich nutzbar sein. uN-Locator allein ist **kein** Nachweis einer uDT4-Funktion.

## Test 4: Ende-zu-Ende

- Von Client-11 das LAN auf Client-91 erreichen.
- Von Client-91 die Gegenrichtung testen.
- Wenn kein echtes LAN hinter CEs vorhanden ist, geeignete CE-Loopbacks als /32-Testnetze nutzen und die statischen Routen entsprechend anpassen.
- Optional `ping vrf KUNDE_A` vom PE aus (plattformabhängige Befehlsargumente mit `?` prüfen).

## Fehlersuchmatrix

| Symptom | Prüfen |
|---|---|
| BGP-Session down | IPv6-Reachability, ASN, `update-source`, AF-Aktivierung |
| BGP up, kein VPNv4-Prefix | Statische Route tatsächlich in VRF-RIB? `redistribute static`? |
| VPNv4 da, kein Import | Route Target auf beiden PEs, RD, VRF-AF |
| Keine Service-SID | Locator-Kapazität, SRv6-BGP-Konfiguration, Image-/Plattform-Unterstützung |
| VRF-Routen da, Ping scheitert | SRv6-Datenebene, CE-Gateway und Rückroute, MTU, Filter |
| Nur bei Ausfall Probleme | IS-IS-Unterlay, TI-LFA-Backup und Konvergenz separat untersuchen |

!!! danger "Nicht verwechseln"
    Ein funktionierendes MP-BGP VPNv4 beweist **nicht**, dass SRv6-L3VPN in der virtuellen XRv-9000-Datenebene funktioniert. Der Erfolgsnachweis erfordert eine tatsächlich weiterleitende uDT4-SID und beidseitigen Kundendatenverkehr.
