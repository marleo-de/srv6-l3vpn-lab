# Phase 4 – Zweite VRF KUNDE_B

Nach erfolgreichem [Basistest KUNDE_A](l3vpn.md) fügen wir auf **XRv-1** und **XRv-10** eine zweite, unabhängig geroutete VPN-Instanz hinzu.

!!! warning "Vorab: Client-11 und Client-101 sind jeweils gemeinsam angeschlossene CE-Router"
    An jedem CE enden nun zwei Leitungen (Port 4 für KUNDE_A, Port 5 für KUNDE_B). **Beide Kunden dürfen auf dem CE nicht in derselben globalen Routingtabelle landen.** Für echte Isolation müssen die Client-Router VRF-Lite oder getrennte logische Router-Kontexte unterstützen. Falls das nicht möglich ist, für KUNDE_B jeweils einen separaten CE-Router einsetzen. Nur eine zweite VRF auf dem PE reicht für die vollständige Ende-zu-Ende-Isolation nicht.

!!! note "Bestätigte CE-Ausgangslage"
    Auf **Client-11** zeigt die vorliegende Konfiguration `Gi0/0 = 10.1.11.11/24`, `Gi0/1` noch **ohne IPv4-Adresse**, `Loopback104 = 11.1.0.1/16` und `Loopback106 = 2001:0:11::1/48`. Für KUNDE_B kann der zweite physische CE-Port genutzt werden. **Die bestehende KUNDE_A-Adressierung bitte nicht durch die ursprünglichen Beispiele ersetzen.** Eine VRF-Zuordnung auf dem IOSv-CE kann vorhandene IPv4-Adressen entfernen und muss deshalb geplant erfolgen.

## 1. Neuer Adressplan

Die CE-Portnummern ergeben sich aus **Seite 3** der vorhandenen Lab-PDF. Die IPv4-Adressen sind neue Beispiele; vor der Nutzung auf Konflikte prüfen.

| Einstellung | XRv-1 / Client-11 | XRv-10 / Client-101 |
|---|---|---|
| VRF | `KUNDE_B` | `KUNDE_B` |
| PE-Interface | `Gi0/0/0/5` | `Gi0/0/0/5` |
| CE-Interface | `G0/1` | `G0/1` |
| PE-IP | `172.16.12.1/30` | `172.16.102.1/30` |
| CE-IP | `172.16.12.2/30` | `172.16.102.2/30` |
| CE-Test-LAN | `192.168.12.0/24` | `192.168.102.0/24` |
| RD | `65000:12` | `65000:102` |
| Import-/Export-RT | `65000:200` | `65000:200` |

Der BGP-AS bleibt **65000**; die vorhandene iBGP-VPNv4-Nachbarschaft zwischen beiden PEs wird **mitbenutzt**. Es wird keine zweite BGP-Session gebraucht.

## 2. VRF KUNDE_B – auf beiden PEs

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

Das Route Target `65000:200` darf nicht mit dem von `KUNDE_A` (`65000:100`) verwechselt werden.

## 3. Kundenseitige Interfaces

**XRv-1**:

```cisco
configure
interface GigabitEthernet0/0/0/5
 vrf KUNDE_B
 ipv4 address 172.16.12.1 255.255.255.252
 no shutdown
!
commit
```

**XRv-10**:

```cisco
configure
interface GigabitEthernet0/0/0/5
 vrf KUNDE_B
 ipv4 address 172.16.102.1 255.255.255.252
 no shutdown
!
commit
```

!!! note "Bestehende Interface-Konfiguration"
    Falls die Ports bereits Adressen besitzen, vor der VRF-Änderung `show running-config interface GigabitEthernet0/0/0/5` prüfen. IOS XR kann das Entfernen einer bestehenden IPv4-Adresse vor dem VRF-Wechsel erfordern.

## 4. Statische Routen zu den Kundennetzen

**XRv-1**:

```cisco
configure
router static
 vrf KUNDE_B
  address-family ipv4 unicast
   192.168.12.0/24 172.16.12.2
  !
 !
!
commit
```

**XRv-10**:

```cisco
configure
router static
 vrf KUNDE_B
  address-family ipv4 unicast
   192.168.102.0/24 172.16.102.2
  !
 !
!
commit
```

Die CE-Testnetze müssen am jeweiligen Client tatsächlich existieren (z. B. als Loopback mit einer Testadresse und entsprechend angepasstem /32-Präfix) – ansonsten wird der Ping ins entfernte LAN trotz funktionierendem SRv6 nicht beantwortet.

## 5. KUNDE_B in vorhandenes BGP aufnehmen

**Auf XRv-1** im vorhandenen `router bgp 65000` ergänzen:

```cisco
configure
router bgp 65000
 vrf KUNDE_B
  rd 65000:12
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

**Auf XRv-10**:

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

Damit soll jede VRF eine eigene SRv6-Service-SID mit uDT4-Verhalten erhalten. **Die tatsächliche SID-Installation und Datenebenenfunktion auf XRv 9000 mit IOS XR 25.1.1 sind im Lab nachzuweisen**, nicht vorauszusetzen.

## 6. CE-Seite – zwingend getrennte Routingtabellen

Auf **Client-11**:

- `G0/0` in CE-VRF `KUNDE_A`, IPv4 `172.16.11.2/30`, Default/Route über `172.16.11.1`
- `G0/1` in CE-VRF `KUNDE_B`, IPv4 `172.16.12.2/30`, Default/Route über `172.16.12.1`

Auf **Client-101**:

- `G0/0` in CE-VRF `KUNDE_A`, IPv4 `172.16.101.2/30`, Default/Route über `172.16.101.1`
- `G0/1` in CE-VRF `KUNDE_B`, IPv4 `172.16.102.2/30`, Default/Route über `172.16.102.1`

**Client-11 läuft mit Cisco IOSv 15.9(3)M4 (`vios-adventerprisek9-m`).** Für die CE-Konfiguration gilt klassische Cisco-IOS-Syntax (`configure terminal`, `ip vrf` bzw. je nach Parser `vrf definition`, `ip vrf forwarding` bzw. `vrf forwarding`, und `write memory`). Die genaue VRF-Lite-Befehlssyntax und Unterstützung bitte auf dem IOSv-Gerät mit `vrf ?` / `ip vrf ?` prüfen, bevor die CE-Konfiguration übernommen wird. `Client-101` wird als IOSv angenommen und sollte mit `show version` gegengeprüft werden.

## 7. Verifikation

Auf **beiden PEs**:

```cisco
show vrf KUNDE_A
show vrf KUNDE_B
show route vrf KUNDE_A
show route vrf KUNDE_B
show bgp vpnv4 unicast summary
show bgp vpnv4 unicast
show segment-routing srv6 sid
```

| Test | Erwartetes Ergebnis |
|---|---|
| KUNDE_B von Client-11 nach Client-101 | `192.168.102.0/24` erreichbar, Gegenrichtung ebenfalls |
| KUNDE_A weiterhin erreichbar | Vorhandene Konnektivität bleibt intakt |
| KUNDE_A → KUNDE_B | **Keine Erreichbarkeit**, solange keine explizite Routenfreigabe/Leak-Konfiguration vorliegt |
| BGP-Routen | A-Routen RT `65000:100`, B-Routen RT `65000:200` |
| SRv6 | Pro VRF eigene Service-SID; bei fehlender uDT4-Datenebene Test nicht bestanden |

### Optional: überlappende Adressen

Wenn beide Mandanten unabhängig funktionieren, können sie absichtlich **dieselben IPv4-Präfixe** verwenden. Das erfordert getrennte Routingtabellen auf PE **und** CE. Dazu zuerst die Testpräfixe auf beiden Kundenseiten nachvollziehbar anpassen, damit identische Adressen nicht versehentlich innerhalb *derselben* VRF doppelt vergeben werden.

Zurück zu [Verifikation und Fehlersuche](verifikation.md).
