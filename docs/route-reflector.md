# XRv-6 als VPNv4 Route Reflector

XRv-6 übernimmt die **zentrale BGP-Route-Reflector-Rolle** für die vier PEs **XRv-4, XRv-5, XRv-7 und XRv-10**. Das vorhandene IS-IS-/SRv6-Underlay bleibt bestehen.

## Rollen

| Router | BGP-Rolle | VPN |
|---|---|---|
| XRv-6 | Route Reflector (RR) | Reflektiert VPNv4 für A, B und C |
| XRv-5 | RR-Client / PE | KUNDE_A, KUNDE_B |
| XRv-10 | RR-Client / PE | KUNDE_A, KUNDE_B |
| XRv-4 | RR-Client / PE | KUNDE_C |
| XRv-7 | RR-Client / PE | KUNDE_C |

Alle Provider-BGP-Router verwenden gemäß **neuem Design** AS **65000**. Die historische AS **8412** bleibt eine Referenz, nicht die aktuelle Konfiguration.

## BGP und Datenpfad trennen

Die VPNv4-Control-Plane lautet:

```text
                   XRv-6
              MP-BGP VPNv4 RR
              /    /   \    \
          XRv-4 XRv-5 XRv-7 XRv-10
              RR-Clients (PE)
```

Der **Kundendatenverkehr wird nicht über XRv-6 umgeleitet**, nur weil XRv-6 Route Reflector ist. Der SRv6-Datenpfad wird durch das bestehende IPv6-Underlay und die im VPNv4 ausgetauschten SRv6-Service-SIDs bestimmt.

## Voraussetzungen

- `Loopback0` von XRv-6 ist laut bestehendem Schema `2001:db8:1:1::6/128`; **auf XRv-6 live überprüfen**.
- `Loopback0` aller vier PEs ist per IPv6 zu XRv-6 erreichbar.
- Auf dem RR wird **keine** Kunden-VRF und **keine** uDT4-Service-SID für diese VPNs benötigt, sofern XRv-6 ausschließlich Routen reflektiert.
- Die folgenden Befehle sind ein **IOS XR 25.1.1 Lab-Entwurf**, nicht auf dem Image getestete Endkonfiguration.

## XRv-6: Route Reflector konfigurieren

```cisco
configure
router bgp 65000
 bgp router-id 1.1.1.6
 address-family vpnv4 unicast
 !
 neighbor 2001:db8:1:1::4
  remote-as 65000
  update-source Loopback0
  address-family vpnv4 unicast
   route-reflector-client
  !
 !
 neighbor 2001:db8:1:1::5
  remote-as 65000
  update-source Loopback0
  address-family vpnv4 unicast
   route-reflector-client
  !
 !
 neighbor 2001:db8:1:1::7
  remote-as 65000
  update-source Loopback0
  address-family vpnv4 unicast
   route-reflector-client
  !
 !
 neighbor 2001:db8:1:1::10
  remote-as 65000
  update-source Loopback0
  address-family vpnv4 unicast
   route-reflector-client
  !
 !
!
commit
```

**Beachte:** Wenn bereits ein BGP-Prozess unter einer anderen AS auf XRv-6 existiert, **nicht** einfach einen zweiten `router bgp`-Prozess einspielen. Vorher mit `show running-config router bgp` die bestehende BGP-Konfiguration aufnehmen.

## Auf allen vier PEs: RR als einzigen VPNv4-iBGP-Peer einrichten

Auf **XRv-4, XRv-5, XRv-7 und XRv-10** (im jeweils vorhandenen `router bgp 65000`):

```cisco
configure
router bgp 65000
 neighbor 2001:db8:1:1::6
  remote-as 65000
  update-source Loopback0
  address-family vpnv4 unicast
  !
 !
!
commit
```

Die bisher dokumentierten **direkten VPNv4-Peerings XRv-5 ↔ XRv-10 und XRv-4 ↔ XRv-7 entfallen**. Die VRF-Definitionen, RD/RT-Werte und CE-eBGP-Nachbarschaften werden **nicht** auf XRv-6 verschoben.

!!! warning "VPNv4 über IPv6"
    Die Nachbarschaften nutzen IPv6-Loopbacks für VPNv4. Vor einem produktiven Nachbau auf XRv 9000 unbedingt per `show bgp ...` und `show segment-routing srv6 sid` überprüfen, ob VPNv4-NLRI, SRv6-Service-SID und Next-Hop so transportiert werden, dass die PEs direkt über SRv6 weiterleiten können. **Kein Next-Hop-Self am RR konfigurieren**, solange die SRv6-VPNv4-Funktion nicht geprüft ist.

## Verifikation

Auf **XRv-6**:

```cisco
show running-config router bgp 65000
show bgp vpnv4 unicast summary
show bgp vpnv4 unicast
```

Erwartet werden **vier etablierte** VPNv4-BGP-Nachbarschaften.

Auf **XRv-5**, **XRv-10**, **XRv-4**, **XRv-7**:

```cisco
show bgp vpnv4 unicast summary
show bgp vpnv4 unicast
show route vrf KUNDE_A
show route vrf KUNDE_B
show route vrf KUNDE_C
show segment-routing srv6 sid
```

Nur VRFs abfragen, die auf dem jeweiligen PE tatsächlich konfiguriert sind. Nachweise: Routen nur in der zugehörigen VRF, korrekte RTs, **ursprünglicher PE-Next-Hop** statt RR-Next-Hop und schließlich echter CE-zu-CE-Datenverkehr.

**Nächster sinnvoller Schritt:** aktuelle `show running-config router bgp`- und `show running-config interface Loopback0`-Ausgaben von XRv-6 prüfen, bevor der Entwurf eingespielt wird.
