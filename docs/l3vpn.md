# Phase 2 – IPv4-L3VPN über SRv6

!!! warning "Lab-Konfiguration – nicht blind einspielen"
    Die CE-Interface-Nummern sind aus Seite 3 der PDF abgeleitet; die vorgeschlagenen IPv4-Adressen dagegen sind neu und müssen vor `commit` abgeglichen werden. Cisco dokumentiert die VPNv4/uDT4-Konfiguration für andere IOS-XR-Hardwarefamilien; XRv 9000 muss in diesem Lab praktisch geprüft werden.

## 1. Kundenseitige VRF (beide PEs)

Auf **XRv-1** und **XRv-10**:

```cisco
configure
vrf KUNDE_A
 address-family ipv4 unicast
  import route-target
   65000:100
  !
  export route-target
   65000:100
  !
 !
!
commit
```

Prüfen:

```cisco
show vrf KUNDE_A
show running-config vrf KUNDE_A
```

## 2. Kundenseitige Interfaces

**XRv-1** – Port 4 aus der PDF, vorab Status prüfen:

```cisco
configure
interface GigabitEthernet0/0/0/4
 vrf KUNDE_A
 ipv4 address 172.16.11.1 255.255.255.252
 no shutdown
!
commit
```

**XRv-10** – Port 4 aus der Topologiegrafik, vorab Status prüfen:

```cisco
configure
interface GigabitEthernet0/0/0/4
 vrf KUNDE_A
 ipv4 address 172.16.101.1 255.255.255.252
 no shutdown
!
commit
```

Bei einer bereits vorhandenen Interface-Konfiguration erst Kollisionen mit IP-Adressen, CDP und VRFs prüfen. IOS XR kann für die VRF-Zuweisung auf bestehenden Interfaces vorheriges Entfernen der IPv4-Adresse verlangen.

## 3. CE-Adressen und statische Routen

Vorschlag für Client-11: `172.16.11.2/30` mit LAN `192.168.11.0/24`; Client-101: `172.16.101.2/30` mit LAN `192.168.101.0/24`. CE-Konfiguration ist abhängig vom Gerätetyp und wird **nicht** als angeblich vorhandene Konfiguration dargestellt.

**XRv-1**:

```cisco
configure
router static
 vrf KUNDE_A
  address-family ipv4 unicast
   192.168.11.0/24 172.16.11.2
  !
 !
!
commit
```

**XRv-10**:

```cisco
configure
router static
 vrf KUNDE_A
  address-family ipv4 unicast
   192.168.101.0/24 172.16.101.2
  !
 !
!
commit
```

## 4. MP-BGP VPNv4 und uDT4 (per VRF)

Die VPNv4-Nachbarschaft nutzt **IPv6-Loopbacks als Transport**. Die Route Targets unter `vrf` steuern den Import/Export, die RDs in BGP unterscheiden die VPN-Routen.

**XRv-1**:

```cisco
configure
router bgp 65000
 bgp router-id 1.1.1.1
 address-family vpnv4 unicast
 !
 neighbor 2001:db8:1:1::10
  remote-as 65000
  update-source Loopback0
  address-family vpnv4 unicast
  !
 !
 vrf KUNDE_A
  rd 65000:11
  address-family ipv4 unicast
   redistribute static
   segment-routing srv6
    locator LABlocator
    alloc mode per-vrf
   !
  !
 !
!
commit
```

**XRv-10**:

```cisco
configure
router bgp 65000
 bgp router-id 1.1.1.10
 address-family vpnv4 unicast
 !
 neighbor 2001:db8:1:1::1
  remote-as 65000
  update-source Loopback0
  address-family vpnv4 unicast
  !
 !
 vrf KUNDE_A
  rd 65000:101
  address-family ipv4 unicast
   redistribute static
   segment-routing srv6
    locator LABlocator
    alloc mode per-vrf
   !
  !
 !
!
commit
```

!!! info "Warum pro VRF?"
    `alloc mode per-vrf` wählt in der dokumentierten Cisco-Implementierung für IPv4-L3VPN das uDT4-Verhalten: eine servicegebundene SID je VRF statt einer SID pro Präfix. Keine manuelle SID fest vorgeben.

## 5. Freigabe vor dem nächsten Schritt

```cisco
show bgp vpnv4 unicast summary
show bgp vpnv4 unicast
show route vrf KUNDE_A
show segment-routing srv6 sid
```

Wenn BGP zwar VPNv4-Routen austauscht, aber keine uDT4-Service-SID installiert wird, **nicht** die Datenebene als erfolgreich bewerten. Weiter bei [Verifikation](verifikation.md).

Nach erfolgreicher Prüfung folgt die [zweite VRF KUNDE_B](kunde-b.md) auf Interface `Gi0/0/0/5`.

## Quelle für die Syntax

[Cisco NCS 5500 IOS XR 25.x, Abschnitt IPv4-L3VPN über SRv6 Micro-SIDs](https://www.cisco.com/c/en/us/td/docs/iosxr/ncs5500/segment-routing/25xx/configuration/guide/b-segment-routing-cg-ncs5500-25xx.pdf). Die konkrete Plattformunterstützung von XRv 9000 bleibt offen.
