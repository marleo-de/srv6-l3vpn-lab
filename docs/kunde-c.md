# Phase 5 – Dritte VRF KUNDE_C: Client-41 und Client-71

!!! info "BGP-Ziel statt statischer Dauerlösung"
    Die alten Dateien enthalten eBGP mit Client-AS 41 und 71 und historischem Provider-AS 8412. Das neue Lab soll eBGP ergänzen; die statischen Konfigurationen hier sind vorläufige Testvarianten. [BGP-Plan](bgp-plan.md).

Mit dieser Erweiterung erhalten **Client-41** und **Client-71** ein eigenes IPv4-L3VPN `KUNDE_C`, getrennt von `KUNDE_A` und `KUNDE_B`.

!!! warning "Noch nicht bestätigte Adressen"
    Auf Seite 3 des ursprünglichen SRv6-Labs sind XRv-4 ↔ Client-41 und XRv-7 ↔ Client-71 als physische Verbindungen erkennbar. **Die tatsächlich konfigurierten CE-IPs, Loopback-Adressen sowie Interface-Zustände von XRv-4/7 und Client-41/71 liegen bislang nicht vor.** Die unten aufgeführten Adressen sind daher **nur ein konsistenter neuer Vorschlag**. Nicht blind übernehmen. Falls dort bereits `10.x`-Netze und `41.1.0.0/16` oder `71.1.0.0/16` verwendet werden, den Ist-Zustand bevorzugen.

## 1. Rollen und Planungswerte

| Merkmal | Standort Client-41 | Standort Client-71 |
|---|---|---|
| Provider Edge | **XRv-4** | **XRv-7** |
| PE-Port (Topologie) | `Gi0/0/0/4` | `Gi0/0/0/4` |
| CE-Port (Topologie) | `Gi0/0` | `Gi0/0` |
| VRF | `KUNDE_C` | `KUNDE_C` |
| RD (Vorschlag) | `65000:41` | `65000:71` |
| Route Target | `65000:300` | `65000:300` |
| PE IPv4 (Vorschlag) | `172.16.41.1/30` | `172.16.71.1/30` |
| CE IPv4 (Vorschlag) | `172.16.41.2/30` | `172.16.71.2/30` |
| CE-Testloopback (Vorschlag) | `192.168.41.1/32` | `192.168.71.1/32` |
| IPv6 BGP-Loopback laut Lab-Schema | `2001:db8:1:1::4` | `2001:db8:1:1::7` |
| SRv6-uSID-Locator laut Lab-Schema | `5f00:0:4::/48` | `5f00:0:7::/48` |

Die Loopback-/Locatorwerte entsprechen dem Nummerierungsschema des ursprünglichen Lab-Leitfadens; **vor Konfiguration auf den betroffenen Routern verifizieren**. Das Underlay bleibt unverändert.

## 2. Vorabprüfungen auf XRv-4 und XRv-7

```cisco
show version
show running-config interface GigabitEthernet0/0/0/4
show running-config interface Loopback0
show segment-routing srv6 locator detail
show isis neighbors
show route ipv6
show running-config router bgp 65000
```

Auf Client-41 und Client-71 (`IOSv` als **Annahme**, nicht per Screenshot bestätigt):

```cisco
show version
show running-config interface GigabitEthernet0/0
show ip interface brief
show running-config | section interface Loopback
show ip route
```

## 3. VRF auf XRv-4 und XRv-7

Identische VRF-Definition auf beiden PEs:

```cisco
configure
vrf KUNDE_C
 address-family ipv4 unicast
  import route-target
   65000:300
  !
  export route-target
   65000:300
  !
 !
!
commit
```

## 4. PE-CE-Anschlüsse (nur bei Übernahme des vorgeschlagenen IP-Plans)

**XRv-4**:

```cisco
configure
interface GigabitEthernet0/0/0/4
 vrf KUNDE_C
 ipv4 address 172.16.41.1 255.255.255.252
 no shutdown
!
commit
```

**XRv-7**:

```cisco
configure
interface GigabitEthernet0/0/0/4
 vrf KUNDE_C
 ipv4 address 172.16.71.1 255.255.255.252
 no shutdown
!
commit
```

!!! danger "Bestandskonfiguration hat Vorrang"
    Bei bereits adressierten Interfaces kann das `vrf`-Kommando bestehende IPv4-Adressen entfernen. Zuerst Ist-Konfiguration sichern, bestehende IPs zuordnen und nur bei gewollter Umadressierung die Beispiel-Adressen verwenden.

## 5. Statisches PE-CE-Routing (Beispiel)

Wenn die vorgeschlagenen Loopback-`/32`-Adressen **auf den CEs tatsächlich konfiguriert** werden:

**XRv-4**:

```cisco
configure
router static
 vrf KUNDE_C
  address-family ipv4 unicast
   192.168.41.1/32 172.16.41.2
  !
 !
!
commit
```

**XRv-7**:

```cisco
configure
router static
 vrf KUNDE_C
  address-family ipv4 unicast
   192.168.71.1/32 172.16.71.2
  !
 !
!
commit
```

## 6. MP-BGP VPNv4 – Peer XRv-4 ↔ XRv-7

Die beiden PEs benötigen für dieses Zwei-Standort-VPN nur eine direkte iBGP-VPNv4-Session. **Weitere PE-Peers sind für diese neue VRF nicht erforderlich**, solange kein Route Reflector oder andere Dienste hinzukommen.

**XRv-4**:

```cisco
configure
router bgp 65000
 bgp router-id 1.1.1.4
 address-family vpnv4 unicast
 !
 neighbor 2001:db8:1:1::7
  remote-as 65000
  update-source Loopback0
  address-family vpnv4 unicast
  !
 !
 vrf KUNDE_C
  rd 65000:41
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

**XRv-7**:

```cisco
configure
router bgp 65000
 bgp router-id 1.1.1.7
 address-family vpnv4 unicast
 !
 neighbor 2001:db8:1:1::4
  remote-as 65000
  update-source Loopback0
  address-family vpnv4 unicast
  !
 !
 vrf KUNDE_C
  rd 65000:71
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

!!! warning "Virtuelle Datenebene"
    Ein erfolgreicher BGP-Routenaustausch bestätigt noch nicht, dass XRv 9000 die **uDT4**-Weiterleitung unterstützt. Den tatsächlich programmierten SRv6-Service-Endpunkt und CE-zu-CE-Verkehr prüfen. Die gezeigte Syntax ist weiterhin ein *plattformabhängiger Lab-Entwurf*.

## 7. CE-Konfiguration (IOSv-Beispiel, nur für unbenutzte Ports/Loopbacks)

Für einen **einzelnen** Kunden pro CE kann das CE-Interface global geroutet bleiben. Soll der CE mehrere Kunden gleichzeitig führen, VRF-Lite verwenden, wie im Kapitel [KUNDE_B](kunde-b.md) beschrieben.

**Client-41**:

```cisco
configure terminal
interface GigabitEthernet0/0
 ip address 172.16.41.2 255.255.255.252
 no shutdown
exit
interface Loopback141
 ip address 192.168.41.1 255.255.255.255
exit
ip route 192.168.71.1 255.255.255.255 172.16.41.1
end
write memory
```

**Client-71**:

```cisco
configure terminal
interface GigabitEthernet0/0
 ip address 172.16.71.2 255.255.255.252
 no shutdown
exit
interface Loopback171
 ip address 192.168.71.1 255.255.255.255
exit
ip route 192.168.41.1 255.255.255.255 172.16.71.1
end
write memory
```

Die Loopback-Nummern 141/171 sind **neu gewählt**, um möglicherweise vorhandene `Loopback104` nicht zu überschreiben. Vorher prüfen, ob die Interface-IDs noch frei sind.

## 8. Verifikation

Auf den beiden PEs:

```cisco
show vrf KUNDE_C
show route vrf KUNDE_C
show bgp vpnv4 unicast summary
show bgp vpnv4 unicast
show segment-routing srv6 sid
```

Auf Client-41:

```cisco
ping 172.16.41.1
ping 192.168.71.1 source Loopback141
traceroute 192.168.71.1 source Loopback141
```

Auf Client-71 entsprechend mit `192.168.41.1` und `Loopback171`.

**Isolation:** `KUNDE_C` importiert nur `65000:300`; ohne Route Leaking dürfen die Routen von `KUNDE_A` (`65000:100`) und `KUNDE_B` (`65000:200`) nicht erscheinen.
