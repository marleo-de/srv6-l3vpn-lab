# Verifikation und Fehlersuche

## VPN-Zuordnung

| VPN | Lokaler PE | Lokales Testpräfix | Gegen-PE | Entferntes Testpräfix |
|---|---|---|---|---|
| KUNDE_A | XRv-5 | `51.1.0.0/16` | XRv-10 | `101.1.0.0/16` |
| KUNDE_B | XRv-5 | `192.168.52.1/32` (geplant) | XRv-10 | `192.168.102.1/32` (geplant) |
| KUNDE_C | XRv-4 | `192.168.41.1/32` (geplant) | XRv-7 | `192.168.71.1/32` (geplant) |

## 1. Control Plane

```cisco
show bgp vpnv4 unicast summary
show bgp vpnv4 unicast
show running-config router bgp 65000
```

Erwartung: **XRv-6 als VPNv4 Route Reflector** hat Nachbarschaften zu XRv-4, XRv-5, XRv-7 und XRv-10. Jeder PE peert mit XRv-6, nicht direkt mit dem anderen Standort. Details: [RR-Verifikation](route-reflector.md). RDs unterscheiden Standorte, RTs unterscheiden Kunden. Nur Routen mit zum VPN passenden RT importieren.

## 2. VRF-Routing auf XRv-5 und XRv-10

```cisco
show route vrf KUNDE_A
show route vrf KUNDE_B
show segment-routing srv6 locator detail
show segment-routing srv6 sid
```

Für KUNDE_A ist auf XRv-5 `51.1.0.0/16` lokal statisch und `101.1.0.0/16` remote über BGP zu erwarten; auf XRv-10 umgekehrt. Für KUNDE_B die getrennten /32-Testnetze prüfen.

## 3. CE-Tests (IOSv)

Von Client-51:

```cisco
show ip vrf
show ip route vrf KUNDE_A
show ip route vrf KUNDE_B
ping vrf KUNDE_A 101.1.0.1 source Loopback104
ping vrf KUNDE_B 192.168.102.1 source Loopback152
```

Von Client-101 jeweils die Gegenrichtung prüfen. Ping über die Loopbacks funktioniert erst, wenn die Rückrouten in den passenden CE-VRFs eingerichtet sind.

## 4. Isolation

- In `KUNDE_A` dürfen keine Routen aus RT `65000:200` oder `65000:300` auftauchen.
- In `KUNDE_B` keine Routen aus RT `65000:100` oder `65000:300`.
- Auf gemeinsam genutzten Client-51/101 müssen `Gi0/0` und `Gi0/1` **separaten CE-VRFs** zugewiesen sein.
- Auf XRv-4/7 entsprechend [KUNDE_C](kunde-c.md) prüfen.

## Fehlersuchmatrix

| Problem | Erster Check |
|---|---|
| BGP VPNv4 down | IPv6-Transport, Loopback0, AS 65000, Update Source |
| BGP steht, keine Routen | Statische VRF-Routen, BGP-Redistribution, RT |
| Keine SRv6-Service-SID | Locator, `alloc mode per-vrf`, Image-/Plattformunterstützung |
| Kein Ping | VRF-RIB, CE-VRF-Lite, Rückroute, SRv6-Forwarding |
| Unterschiedliche Mandanten erreichbar | Route-Target-Import, Route Leaking, CE-Routingtabellen |

!!! danger "BGP-Routen sind kein Forwarding-Beweis"
    Erst die tatsächliche Service-SID (uDT4) und **beidseitigen IPv4-Verkehr** über SRv6 bestätigen. XRv-9000-Support ist hier nicht allein durch die dokumentierte IOS-XR-Syntax bewiesen.
