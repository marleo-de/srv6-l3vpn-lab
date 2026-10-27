# SRv6 IPv4-L3VPN Lab

**Cisco IOS XRv 9000 · IOS XR 25.1.1 · SRv6 uSID · IS-IS Level 2 · MP-BGP VPNv4**

Diese Lab-Erweiterung ergänzt das **bereits aufgebaute SRv6-Underlay** um BGP und drei getrennte IPv4-L3VPN-Dienste. Die bereitgestellten älteren Konfigurationsdateien (IOS XR 6.5.3 / IOS 15.7) dienen ausschließlich als Referenz für frühere Adressierung und eBGP-Nachbarn; sie sind **keine** wiederherzustellende Konfiguration. Ziel ist **PE–CE eBGP** und **MP-BGP VPNv4 über XRv-6 als zentralen Route Reflector**.

## Standorte und Kundenzuordnung

| VRF | PE links | CE links | PE rechts | CE rechts | Route Target |
|---|---|---|---|---|---|
| **KUNDE_A** | XRv-5 | Client-51 | XRv-10 | Client-101 | `65000:100` |
| **KUNDE_B** | XRv-5 | Client-51 | XRv-10 | Client-101 | `65000:200` |
| **KUNDE_C** | XRv-4 | Client-41 | XRv-7 | Client-71 | `65000:300` |

**Client-11 / XRv-1 gehören nicht zu KUNDE_A oder KUNDE_B.** Die bisherige Client-11-Konfiguration ist für diese VPN-Dienste abgelöst, bleibt aber unverändert auf dem Lab-Gerät.

**BGP-Rollen:** XRv-6 = Route Reflector, XRv-4/5/7/10 = RR-Clients (PEs). [RR-Konfiguration](route-reflector.md).

## Bestätigte Plattformen

| Funktion | Software | Nachweis |
|---|---|---|
| Provider-Router | Cisco IOS XRv 9000, **IOS XR 25.1.1** | `show version` auf XRv-3 |
| Client-/CE-Router | Cisco IOSv **15.9(3)M4** | `show version` auf Client-11 |

Andere Routerversionen sind anhand der bislang vorliegenden Ausgaben nicht einzeln bestätigt. Auf PE gilt `configure` + `commit`; auf IOSv CE gilt `configure terminal` + `end`/`write memory`.

## Ablauf

1. [Topologie und tatsächliche/bisher unbekannte IP-Adressen](topologie.md)
2. [SRv6-Underlay vor der Änderung testen](underlay.md)
3. [KUNDE_A auf XRv-5 und XRv-10](l3vpn.md)
4. [BGP-Design und Migration von der historischen Konfiguration](bgp-plan.md)
5. [XRv-6 als VPNv4 Route Reflector konfigurieren](route-reflector.md)
5. [KUNDE_B auf denselben PEs, aber getrennten Ports](kunde-b.md)
6. [KUNDE_C mit Client-41 und Client-71](kunde-c.md)
7. [Verifikation, Isolation und Fehlersuche](verifikation.md)

!!! warning "Noch nicht vollständig geprüft"
    XRv-5 `Gi0/0/0/4 = 10.5.51.5/24` und Client-51 `Gi0/0 = 10.5.51.51/24` sind bestätigt. Auch die XRv-9000-Unterstützung für SRv6-uDT4-Datenebenenweiterleitung ist in diesem Lab noch nachzuweisen. Die Beispielblöcke dürfen nicht vor den dokumentierten Vorprüfungen übernommen werden.

## Grundlage

Ausgangsmaterial: *SRv6 LAB – Leitfaden / Notizen*, 16 Seiten, einschließlich IS-IS, uSID, Flex-Algo und TI-LFA. [Topologie](topologie.md) trennt bestätigte Gerätekonfigurationen von neu vorgeschlagenen Werten.
