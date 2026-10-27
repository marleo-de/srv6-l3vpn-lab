# VRF KUNDE_A – Client-51 ↔ Client-101

**PE-Paar:** XRv-5 ↔ XRv-10; **VPNv4-Control-Plane über RR XRv-6**. **Route Target:** `65000:100`. **Provider-AS:** `65000`.

!!! warning "Vor der Konfiguration prüfen"
    XRv-5 `Gi0/0/0/4 = 10.5.51.5/24`, Client-51 `Gi0/0 = 10.5.51.51/24` und Loopback104 `51.1.0.1/16` sind nun per Screenshot bestätigt. Die VRF-Zuweisung kann die bisherige Interface-IP entfernen; die Adresse im selben Änderungsschritt wieder setzen. SRv6-uDT4-Funktion auf XRv 9000 bleibt separat zu prüfen.

## 1. VRF auf XRv-5 und XRv-10

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

## 2. Kundenseitige PE-Interfaces

**XRv-5 – bestehende IP-Adresse zuerst ermitteln; diesen Block nicht unverändert einfügen:**

```cisco
show running-config interface GigabitEthernet0/0/0/4
```

```cisco
configure
interface GigabitEthernet0/0/0/4
 vrf KUNDE_A
 ipv4 address <BESTEHENDE_XRv5_Gi4_IP> 255.255.255.0
 no shutdown
!
commit
```

**XRv-10 – bereits bestätigte IP:**

```cisco
configure
interface GigabitEthernet0/0/0/4
 vrf KUNDE_A
 ipv4 address 10.10.101.10 255.255.255.0
 no shutdown
!
commit
```

!!! danger "VRF-Zuweisung kann Adressen entfernen"
    IOS XR kann die IPv4-Adresse beim Wechsel von globaler Tabelle in VRF entfernen. Die bestehende Konfiguration vorher sichern. Das Unterlay (IS-IS) nicht an die CE-Interfaces hängen.

## 3. Vorläufiger statischer Basistest – **nicht** das Zielrouting

Die folgenden Routen sind nur optional für einen isolierten SRv6-Basistest. Für die eigentliche Lab-Erweiterung ist **eBGP PE–CE** vorgesehen. Nicht gleichzeitig blind mit dynamisch gelernten Präfixen konfigurieren.

## 3.1 Statische PE-Routen zu den Testnetzen

**XRv-5 – Client-51 ist über `10.5.51.51` erreichbar:**

```cisco
configure
router static
 vrf KUNDE_A
  address-family ipv4 unicast
   51.1.0.0/16 10.5.51.51
  !
 !
!
commit
```

**XRv-10 – Client-101:**

```cisco
configure
router static
 vrf KUNDE_A
  address-family ipv4 unicast
   101.1.0.0/16 10.10.101.101
  !
 !
!
commit
```

Für das geplante eBGP-Routing siehe [BGP-Design](bgp-plan.md).

## 4. MP-BGP VPNv4 via XRv-6 und SRv6-Service-SIDs

**Voraussetzung:** XRv-6 ist als [VPNv4 Route Reflector](route-reflector.md) eingerichtet. Die Peers zeigen auf XRv-6 und **nicht direkt auf den gegenüberliegenden PE**.

**XRv-5:**

```cisco
configure
router bgp 65000
 bgp router-id 1.1.1.5
 address-family vpnv4 unicast
 !
 neighbor 2001:db8:1:1::6
  remote-as 65000
  update-source Loopback0
  address-family vpnv4 unicast
  !
 !
 vrf KUNDE_A
  rd 65000:51
  address-family ipv4 unicast
   redistribute static
   ! Nur für statischen Vorabtest; für eBGP später BGP-CE-Routen exportieren
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
 bgp router-id 1.1.1.10
 address-family vpnv4 unicast
 !
 neighbor 2001:db8:1:1::6
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

Die dokumentierte Implementierung nutzt `alloc mode per-vrf` für eine servicebezogene SRv6-SID (uDT4). **Die XRv-9000-Datenebene ist dafür noch nicht verifiziert.**

## 5. CE-Konfiguration: IOSv und VRF-Lite

Da **dieselben CEs auch KUNDE_B bedienen**, müssen die Routingtabellen auf Client-51 und Client-101 getrennt werden. Die bestehenden Loopback104 müssen in die **CE-VRF KUNDE_A** verschoben werden. Sonst liegen die Kundennetze weiterhin in der globalen Tabelle.

Die CE-VRF-Lite-Schritte samt Risiken und Routingtrennung sind unter [KUNDE_B](kunde-b.md) beschrieben. CE-Befehle sind klassische IOS-Befehle, nicht IOS XR.

## 6. Verifikation

```cisco
show vrf KUNDE_A
show route vrf KUNDE_A
show bgp vpnv4 unicast summary
show bgp vpnv4 unicast
show segment-routing srv6 sid
```

Erwartung: auf XRv-5 `51.1.0.0/16` als lokale statische Route und `101.1.0.0/16` als VPNv4/BGP-Route; auf XRv-10 umgekehrt. Zusätzlich eine funktionsfähige SRv6-Service-SID und **beidseitige** CE-Konnektivität. Für die zu testende Rückroute auf den CEs siehe [Verifikation](verifikation.md).
