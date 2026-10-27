# BGP-Design – SRv6-L3VPN Erweiterung

## Zweck und Quellenstand

**Aktuelles Lab:** IOS XRv 9000 25.1.1 mit bereits eingerichtetem IPv6/IS-IS, SRv6/uSID, Flex-Algo und TI-LFA. Jetzt werden **MP-BGP VPNv4** zwischen den PEs und **eBGP IPv4** an den CEs ergänzt.

**Historische Referenz:** Die hochgeladenen Dateien enthalten ältere Konfigurationen aus dem Jahr 2019 (IOS XR 6.5.3 und IOS 15.7) mit MPLS/EVPN und Provider-AS **8412**. Sie werden **nicht** eingespielt und überschreiben **nicht** das bestehende SRv6-Underlay. Angaben aus den aktuellen Screenshots haben Vorrang.

## BGP-Adressfamilien und AS-Plan

| Strecke | Protokoll / AF | AS links | AS rechts | Hinweise |
|---|---|---|---|---|
| XRv-5 ↔ XRv-10 | iBGP VPNv4 über IPv6 Loopback0 | **65000** (neu) | **65000** (neu) | Dienste KUNDE_A und KUNDE_B |
| XRv-4 ↔ XRv-7 | iBGP VPNv4 über IPv6 Loopback0 | **65000** (neu) | **65000** (neu) | Dienst KUNDE_C |
| Client-51 ↔ XRv-5, KUNDE_A | eBGP IPv4 | **51** (historisch) | **65000** (neu) | Link `10.5.51.51 ↔ 10.5.51.5` |
| Client-101 ↔ XRv-10, KUNDE_A | eBGP IPv4 | **noch festzulegen**, z. B. 101 | **65000** (neu) | Link `10.10.101.101 ↔ 10.10.101.10` |
| Client-41 ↔ XRv-4, KUNDE_C | eBGP IPv4 | **41** (historisch) | **65000** (neu) | IPv4-PE `10.4.41.4` aus altem CE-Neighbor-Eintrag, **aktuell prüfen** |
| Client-71 ↔ XRv-7, KUNDE_C | eBGP IPv4 | **71** (historisch) | **65000** (neu) | IPv4-PE `10.7.71.7` aus altem CE-Neighbor-Eintrag, **aktuell prüfen** |

**Für KUNDE_B** auf Client-51/101 kann die CE-AS-Nummer je Router für beide lokalen VRFs gleich bleiben, solange jede eBGP-Nachbarschaft der richtigen **CE-VRF** und **PE-VRF** zugeordnet ist. Eine zweite CE-AS ist nicht nötig.

## Zu beachtende Unterschiede

- Die historischen Clients verwendeten `router bgp 51`, `router bgp 41` bzw. `router bgp 71` mit `remote-as 8412`. Das neue Provider-AS ist der **vorgeschlagene** Wert 65000 – nicht die aktuelle Underlay-AS.
- PE–CE-eBGP arbeitet im **IPv4-VRF-AF** am PE; es wird *nicht* mit dem VPNv4-iBGP-Peer verwechselt.
- BGP-Routen des CE müssen in VPNv4 importiert werden; die in anderen Kapiteln gezeigten `redistribute static`-Anweisungen sind **nur für einen optionalen statischen Vorabtest** vorgesehen und müssen beim eBGP-Design geprüft/angepasst werden.
- `KUNDE_B` nutzt Port 5 und auf beiden CEs eigene VRF-Lite-Routingtabellen. Auf Client-51 war der Port 5 laut aktuellem Screenshot shutdown; bei Client-101 ebenfalls.
- Die Service-SID je VRF mit uDT4-Verhalten und tatsächliche SRv6-Weiterleitung sind **zu verifizieren**. Ein BGP-Established allein reicht nicht.
- Bereits vorhandene SRv6-Locators und IS-IS-Konfiguration dürfen nicht durch ältere MPLS/EVPN-Konfigurationszeilen ersetzt werden.

## Prüfschritte vor dem BGP-Commit

**Provider-Edges:**

```cisco
show version
show running-config router bgp
show running-config interface Loopback0
show running-config interface GigabitEthernet0/0/0/4
show running-config interface GigabitEthernet0/0/0/5
show route ipv6
show segment-routing srv6 locator detail
```

**Clients (IOSv):**

```cisco
show version
show running-config | section router bgp
show running-config | section interface
show ip route
```

Danach die eBGP-Nachbarschaften **pro Kunden-VRF** schrittweise konfigurieren, VPNv4-Austausch prüfen und schließlich Forwarding und Isolation testen.

!!! warning "Status"
    Dieses Dokument ist die **Design-/Migrationsgrundlage** und noch keine auf IOS XRv 9000 25.1.1 im Lab validierte Endkonfiguration. Für Client-101 fehlt im historischen Konfigurationsfile ein BGP-Prozess; die dort vorgeschlagene AS 101 ist daher neu. Auf XRv-4/7 fehlen aktuelle PE-Ausgaben.
