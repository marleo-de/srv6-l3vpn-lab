# Topologie und IP-Plan

## Interface-Nummern aus dem Leitfaden

Auf **Seite 3** der Ausgangs-PDF ist die rote Ziffer am XRv-Router die letzte Stelle von `GigabitEthernet0/0/0/X`. Die beiden horizontal gezeichneten CE-Anbindungen an XRv-1 und XRv-10 sind **getrennte physische Links**.

| PE | Interface | Gegenstelle |
|---|---|---|
| XRv-1 | `Gi0/0/0/4` | Client-11 `G0/0` |
| XRv-1 | `Gi0/0/0/5` | Client-11 `G0/1` |
| XRv-10 | `Gi0/0/0/4` | Client-101 `G0/0` |
| XRv-10 | `Gi0/0/0/5` | Client-101 `G0/1` |

Weitere direkt verbundene Clients aus derselben Grafik:

| XRv-Router | Rolle | CE-Interfaces |
|---|---|---|
| XRv-2 | späterer PE | `Gi0/0/0/4`, `Gi0/0/0/5` ↔ Client-21 |
| XRv-4 | späterer PE | `Gi0/0/0/4`, `Gi0/0/0/5` ↔ Client-41 |
| XRv-5 | späterer PE | `Gi0/0/0/4`, `Gi0/0/0/5` ↔ Client-51 |
| XRv-7 | späterer PE | `Gi0/0/0/4`, `Gi0/0/0/5` ↔ Client-71 |

Die roten Nummern sind **nicht** die letzten Ziffern der IPv4-Adressen. Vor der Konfiguration auf allen Ports `show interfaces brief` und `show running-config interface` prüfen.

## Rollen für den Basistest

| Knoten | Rolle | Bemerkung |
|---|---|---|
| XRv-1 | **PE** | Client-11 |
| XRv-10 | **PE** | Client-101 |
| XRv-2 bis XRv-9 | P (anfangs) | Bestehendes IS-IS-/SRv6-Underlay |
| XRv-2, XRv-4, XRv-5, XRv-7 | weitere PE-Kandidaten | Bereits mit Clients verbunden |

**Hinweis:** P und PE sind Service-Rollen; ein PE bleibt Teil des IS-IS-/SRv6-Underlays. Keine Umverkabelung notwendig.

## Adressen aus dem bestehenden SRv6-Leitfaden

| Merkmal | XRv-1 | XRv-10 |
|---|---|---|
| IPv6 Loopback0 | `2001:db8:1:1::1/128` | `2001:db8:1:1::10/128` |
| IPv4 Loopback0 | `1.1.1.1/32` | `1.1.1.10/32` |
| uSID Locator | `5f00:0:1::/48` | `5f00:0:10::/48` |
| IS-IS | `LABisis` | `LABisis` |
| Locator-Name | `LABlocator` | `LABlocator` |

## Neue Kundennetz-Adressen (Vorschlag)

| Einstellung | Wert |
|---|---|
| Provider-AS | `65000` |
| VRF | `KUNDE_A` |
| Route Target Import/Export | `65000:100` |
| RD auf XRv-1 | `65000:11` |
| RD auf XRv-10 | `65000:101` |
| LAN hinter Client-11 | `192.168.11.0/24` |
| LAN hinter Client-101 | `192.168.101.0/24` |
| PE–CE XRv-1 Port 4 | `172.16.11.1/30` ↔ Client-11 `172.16.11.2/30` |
| PE–CE XRv-10 Port 4 | `172.16.101.1/30` ↔ Client-101 `172.16.101.2/30` |

Die IPv4-Adressen sind *neu vorgeschlagene* Lab-Adressen. Port 5 ist physisch bereits angeschlossen und wird in der [zweiten VRF KUNDE_B](kunde-b.md) verwendet.

## Später

**KUNDE_B:** Port 5, RT `65000:200`, RD `65000:12`/`65000:102`, PE-CE `172.16.12.0/30` bzw. `172.16.102.0/30`, CE-Test-LAN `192.168.12.0/24` und `192.168.102.0/24`. Siehe [Konfiguration](kunde-b.md). CE-Router benötigen dafür VRF-Lite oder getrennte Routingkontexte.
