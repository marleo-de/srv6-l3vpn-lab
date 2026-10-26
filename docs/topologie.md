# Topologie und IP-Plan

## Rollen

| Knoten | Rolle im Ausbau | Bemerkung |
|---|---|---|
| XRv-1 | PE | Client-11; Startpunkt |
| XRv-2 bis XRv-8 | P / potentieller PE | Bestehendes SRv6-Underlay zunächst unverändert |
| XRv-9 | PE | Client-91; Endpunkt |
| XRv-10 | P / potentieller PE | Bestehendes SRv6-Underlay zunächst unverändert |

**Hinweis:** P und PE sind Service-Rollen. Ein PE bleibt Teil des IS-IS-/SRv6-Underlays. Die Topologie wird nicht umverkabelt.

## Aus dem ursprünglichen Leitfaden

| Merkmal | XRv-1 | XRv-9 |
|---|---|---|
| IPv6 Loopback0 | `2001:db8:1:1::1/128` | `2001:db8:1:1::9/128` |
| IPv4 Loopback0 | `1.1.1.1/32` | `1.1.1.9/32` |
| uSID Locator | `5f00:0:1::/48` | `5f00:0:9::/48` |
| IS-IS | `LABisis` | `LABisis` |
| uSID Locator-Name | `LABlocator` | `LABlocator` |

Die ursprüngliche PDF enthält neben der Topologie keine vollständige Zuordnung aller CE-Anschlussports und CE-IP-Adressen.

## Neuer Vorschlag für das Kunden-VPN

| Einstellung | Wert |
|---|---|
| AS | 65000 |
| VRF | `KUNDE_A` |
| Route Target Import/Export | `65000:100` |
| RD auf XRv-1 | `65000:11` |
| RD auf XRv-9 | `65000:91` |
| LAN hinter Client-11 | `192.168.11.0/24` |
| LAN hinter Client-91 | `192.168.91.0/24` |
| IPv4 PE–CE-Link XRv-1 | `172.16.11.1/30` (CE `.2`) |
| IPv4 PE–CE-Link XRv-9 | `172.16.91.1/30` (CE `.2`) |

**Diese IPv4-Adressen sind neu vorgeschlagene Lab-Adressen und müssen mit vorhandenen Adressen abgeglichen werden.** Portnummern sind absichtlich Platzhalter.

## Später

Weitere angeschlossene Clients (gemäß Topologiegrafik aus dem PDF) können als zusätzliche Kundenstandorte und für zweite VRF hinzugefügt werden.
