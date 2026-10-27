# VRF KUNDE_B – Client-51 ↔ Client-101

**Zweite, unabhängige VRF** auf denselben Provider-Edges **XRv-5 und XRv-10**. Die CE-Links auf Port 5 werden separat verwendet; `KUNDE_A` bleibt auf Port 4.

!!! warning "Ist-Zustand und Trennung"
    Client-51 `Gi0/1` ist **shutdown** und hat keine IP; Client-101 `Gi0/1` ist ebenfalls **shutdown**. XRv-10 `Gi0/0/0/5` ist shutdown; **XRv-5 Gi0/0/0/5 ist laut Screenshot shutdown und ohne IPv4-Adresse**. Die Adressen für KUNDE_B sind Vorschläge. Da Client-51 und Client-101 je zwei VPNs bedienen, sind auf ihnen CE-VRF-Lite-Kontexte erforderlich.

## 1. IP-Plan (neu vorgeschlagen)

| Einstellung | XRv-5 / Client-51 | XRv-10 / Client-101 |
|---|---|---|
| PE-Port | `Gi0/0/0/5` | `Gi0/0/0/5` |
| CE-Port | `Gi0/1` | `Gi0/1` |
| PE-IP | `172.16.52.1/30` | `172.16.102.1/30` |
| CE-IP | `172.16.52.2/30` | `172.16.102.2/30` |
| CE-Testloopback | `192.168.52.1/32` | `192.168.102.1/32` |
| VRF / RT | `KUNDE_B` / `65000:200` | `KUNDE_B` / `65000:200` |
| RD | `65000:52` | `65000:102` |

## 2. VRF-Definition (auf beiden PEs)

```cisco
configure
vrf KUNDE_B
 address-family ipv4 unicast
  import route-target
   65000:200
  !
  export route-target
   65000:200
  !
 !
!
commit
```

## 3. PE-Interface (nach Überprüfung der Ist-Konfiguration)

**XRv-5:**

```cisco
configure
interface GigabitEthernet0/0/0/5
 vrf KUNDE_B
 ipv4 address 172.16.52.1 255.255.255.252
 no shutdown
!
commit
```

**XRv-10:**

```cisco
configure
interface GigabitEthernet0/0/0/5
 vrf KUNDE_B
 ipv4 address 172.16.102.1 255.255.255.252
 no shutdown
!
commit
```

## 4. Optionale statische Test-Routen (später durch PE–CE eBGP ersetzen)

**XRv-5:**

```cisco
configure
router static
 vrf KUNDE_B
  address-family ipv4 unicast
   192.168.52.1/32 172.16.52.2
  !
 !
!
commit
```

**XRv-10:**

```cisco
configure
router static
 vrf KUNDE_B
  address-family ipv4 unicast
   192.168.102.1/32 172.16.102.2
  !
 !
!
commit
```

## 5. BGP: zweite VRF im bestehenden AS 65000

Ziel ist PE–CE eBGP für beide VRFs; die historische Konfiguration ist nur Referenz. Siehe [BGP-Design](bgp-plan.md).

Die VPNv4-BGP-Nachbarschaften XRv-5 ↔ **XRv-6 (RR)** und XRv-10 ↔ **XRv-6 (RR)** bestehen aus KUNDE_A bereits. **Keine zweite Session für KUNDE_B** anlegen. [RR-Konfiguration](route-reflector.md).

**XRv-5:**

```cisco
configure
router bgp 65000
 vrf KUNDE_B
  rd 65000:52
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

**XRv-10:**

```cisco
configure
router bgp 65000
 vrf KUNDE_B
  rd 65000:102
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

## 6. CE-Routingtabellen trennen (IOSv 15.x, Syntax im Lab prüfen)

!!! danger "Bestehende IPv4-Adressen und Loopbacks"
    Auf IOSv entfernt `ip vrf forwarding` üblicherweise eine vorhandene Interface-IP-Adresse; sie muss danach innerhalb der VRF neu konfiguriert werden. Bestehende Loopback104 für KUNDE_A umhängen, nicht eine zweite Loopback104 anlegen. Vor `write memory` die Erreichbarkeit kontrollieren.

**Client-51 – CE VRF-Lite (beide Kunden):**

```cisco
configure terminal
ip vrf KUNDE_A
 rd 65000:51
exit
ip vrf KUNDE_B
 rd 65000:52
exit
interface GigabitEthernet0/0
 ip vrf forwarding KUNDE_A
 ip address 10.5.51.51 255.255.255.0
 no shutdown
exit
interface Loopback104
 ip vrf forwarding KUNDE_A
 ip address 51.1.0.1 255.255.0.0
exit
interface GigabitEthernet0/1
 ip vrf forwarding KUNDE_B
 ip address 172.16.52.2 255.255.255.252
 no shutdown
exit
interface Loopback152
 ip vrf forwarding KUNDE_B
 ip address 192.168.52.1 255.255.255.255
exit
ip route vrf KUNDE_A 101.1.0.0 255.255.0.0 10.5.51.5
ip route vrf KUNDE_B 192.168.102.1 255.255.255.255 172.16.52.1
end
```

**Client-101 – CE VRF-Lite (beide Kunden):**

```cisco
configure terminal
ip vrf KUNDE_A
 rd 65000:101
exit
ip vrf KUNDE_B
 rd 65000:102
exit
interface GigabitEthernet0/0
 ip vrf forwarding KUNDE_A
 ip address 10.10.101.101 255.255.255.0
 no shutdown
exit
interface Loopback104
 ip vrf forwarding KUNDE_A
 ip address 101.1.0.1 255.255.0.0
exit
interface GigabitEthernet0/1
 ip vrf forwarding KUNDE_B
 ip address 172.16.102.2 255.255.255.252
 no shutdown
exit
interface Loopback152
 ip vrf forwarding KUNDE_B
 ip address 192.168.102.1 255.255.255.255
exit
ip route vrf KUNDE_A 51.1.0.0 255.255.0.0 10.10.101.10
ip route vrf KUNDE_B 192.168.52.1 255.255.255.255 172.16.102.1
end
```

**Wichtig:** `ip vrf` / `ip vrf forwarding` ist klassische IOS-Syntax, nicht IOS XR. Die RDs auf den CEs dienen der VRF-Lite-Definition, nicht dem MP-BGP-VPNv4-Austausch. Die PE-IP `10.5.51.5` wurde aus der laufenden XRv-5-Konfiguration bestätigt.

## 7. Prüfung

Auf beiden PEs:

```cisco
show vrf KUNDE_A
show vrf KUNDE_B
show route vrf KUNDE_A
show route vrf KUNDE_B
show bgp vpnv4 unicast summary
show segment-routing srv6 sid
```

Auf beiden IOSv-CEs:

```cisco
show ip vrf
show ip route vrf KUNDE_A
show ip route vrf KUNDE_B
ping vrf KUNDE_A 101.1.0.1 source Loopback104
ping vrf KUNDE_B 192.168.102.1 source Loopback152
```

Das Ping-Beispiel ist aus Sicht von **Client-51** formuliert; auf Client-101 Testziele umkehren. Die VPN-Routen von A, B und C dürfen nicht ohne explizite Route-Leaking-Konfiguration vermischt werden.
