# SRv6 IPv4-L3VPN Lab

**Cisco XRv 9000 · IOS XR 25.1.1 · IS-IS Level 2 · SRv6 uSID · MP-BGP VPNv4**

## Bestätigte Router-Images

| Funktion | Plattform | Software |
|---|---|---|
| P- und PE-Router | Cisco IOS XRv 9000 | IOS XR 25.1.1 (`show version` auf XRv-3 bestätigt) |
| Client-/CE-Router | Cisco IOSv | IOS 15.9(3)M4, Image `vios-adventerprisek9-m` (`show version` auf Client-11 bestätigt) |

Auf den PE-Routern wird mit `configure` und `commit` gearbeitet, auf den IOSv-Clients mit klassischer IOS-Konfiguration (`configure terminal`). Die Versionen sind für die gezeigten Geräte belegt; auf weiteren Routern gegebenenfalls ebenfalls `show version` prüfen.


Diese Dokumentation erweitert den bereits durchgeführten SRv6-Leitfaden um ein **IPv4 Layer-3-VPN über SRv6**. Das bestehende Underlay bleibt erhalten.

## Zielbild

- Zwei zunächst aktive Provider-Edges: **XRv-1** und **XRv-10**
- Zwei angeschlossene Kundengeräte: **Client-11** und **Client-101**
- VRF **KUNDE_A**, BGP AS **65000**, Route Target **65000:100**
- VPNv4-Routenaustausch via iBGP zwischen IPv6-Loopbacks
- SRv6-Service-SID je VRF (**uDT4**) aus dem vorhandenen uSID-Locator
- Test von VPN-Isolation, Datenebene und später TI-LFA/Flex-Algo

!!! warning "Wichtiger Plattformhinweis"
    Die Syntax orientiert sich an Cisco IOS XR 25.1.x-Dokumentation für NCS/ASR/8000-Plattformen. **XRv 9000 ist damit nicht als unterstützte SRv6-L3VPN-Datenebene bestätigt.** Die Anleitung ist ein zu verifizierender Lab-Entwurf; die Installation einer SRv6-uDT4-Service-SID und echte CE-zu-CE-Konnektivität sind harte Erfolgskriterien.

## Ablauf

1. [Topologie, Rollen und Adressplan prüfen](topologie.md)
2. [Vorhandenes Underlay verifizieren](underlay.md)
3. [VRF, PE-Interfaces, MP-BGP und SRv6 konfigurieren](l3vpn.md)
4. [Control-Plane und Datenebene prüfen](verifikation.md)
5. [Zweite VRF KUNDE_B auf den zweiten CE-Ports](kunde-b.md)
6. [Weiterführende Testfälle](erweiterungen.md)

!!! info "Kennzeichnung"
    Mit **LAB-WERT ERSETZEN** markierte Interface-Nummern oder IP-Adressen können aus dem vorliegenden Leitfaden nicht sicher abgeleitet werden. Zuerst auf dem jeweiligen Gerät mit `show interfaces brief` beziehungsweise `show running-config interface` ermitteln.

## Dokumentationsgrundlage

Ausgangs-PDF: *SRv6 LAB – Leitfaden / Notizen* (16 Seiten, bereitgestellt im Gespräch). Darin: IS-IS Level 2, LABlocator, Umstellung auf uSID, Flex-Algo 128 und TI-LFA. Abweichungen vom PDF werden in dieser Erweiterung als neue Vorschläge ausgewiesen.

**Referenzen:** [Cisco NCS 5500 SRv6 Micro-SID Guide 25.x](https://www.cisco.com/c/en/us/td/docs/iosxr/ncs5500/segment-routing/25xx/configuration/guide/b-segment-routing-cg-ncs5500-25xx.pdf) · [Cisco ASR 9000 SRv6 Micro-SID Guide 25.x](https://www.cisco.com/c/en/us/td/docs/routers/asr9000/software/25xx/segment-routing/configuration/guide/b-segment-routing-cg-asr9000-25xx/configure-srv6-micro-sid.html)
