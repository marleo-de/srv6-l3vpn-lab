# Topologie und Adressierung

## Physische PE-/CE-Anschlüsse

Die roten Portnummern auf Seite 3 des SRv6-Leitfadens kennzeichnen `GigabitEthernet0/0/0/X` auf XRv. Die Client-Seite nutzt `GigabitEthernet0/0` und `0/1`.

| PE | PE-Interface | CE | CE-Interface | VPN |
|---|---|---|---|---|
| **XRv-5** | `Gi0/0/0/4` – `10.5.51.5/24` | **Client-51** | `Gi0/0` – `10.5.51.51/24` | KUNDE_A |
| **XRv-5** | `Gi0/0/0/5` | **Client-51** | `Gi0/1` | KUNDE_B |
| XRv-10 | `Gi0/0/0/4` | Client-101 | `Gi0/0` | KUNDE_A |
| XRv-10 | `Gi0/0/0/5` | Client-101 | `Gi0/1` | KUNDE_B |
| XRv-4 | `Gi0/0/0/4` | Client-41 | `Gi0/0` | KUNDE_C |
| XRv-7 | `Gi0/0/0/4` | Client-71 | `Gi0/0` | KUNDE_C |

XRv-1 ↔ Client-11 bleibt physisch bestehen, wird aber für A/B **nicht** verwendet. XRv-2 ↔ Client-21 und weitere Links bleiben für spätere Erweiterungen verfügbar. XRv-5 und XRv-10 sind sowohl **PEs als auch Teilnehmer am IS-IS-/SRv6-Underlay**. **XRv-6 ist zentraler VPNv4 Route Reflector** für XRv-4/5/7/10 und bleibt selbst Underlay-Router. [BGP-Topologie](route-reflector.md).

## Bestätigte Ist-Konfiguration: Client-51

Der aktuelle Screenshot zeigt:

| Interface | Tatsächlicher Wert | Status aus Config |
|---|---|---|
| `Loopback104` | `51.1.0.1/16` | `description advertise to PE` |
| `Loopback106` | `2001:0:51::1/48` | IPv6-Testadresse |
| `Gi0/0` | **`10.5.51.51/24`** | `description XRv-5 Gi4`, kein Shutdown |
| `Gi0/1` | keine IPv4-Adresse | `description XRv-5 Gi5`, **shutdown** |

**Jetzt bestätigt:** `XRv-5 Gi0/0/0/4 = 10.5.51.5/24` mit Beschreibung `Client-51 Gi0/0`; `Gi0/0/0/5` ist `shutdown` und hat keine IPv4-Adresse. Darüber hinaus ist `Gi0/0/0/3 = 10.5.6.5/24` Richtung XRv-6 belegt.

## Bestätigte Ist-Konfiguration: XRv-10 und Client-101

| Gerät / Interface | IPv4 | Weitere Information |
|---|---|---|
| XRv-10 `Gi0/0/0/4` | `10.10.101.10/24` | Link Client-101 `Gi0/0` |
| Client-101 `Gi0/0` | `10.10.101.101/24` | Kein Shutdown im gezeigten Block |
| Client-101 `Loopback104` | `101.1.0.1/16` | IPv4-Testpräfix |
| Client-101 `Loopback106` | `2001:0:101::1/48` | IPv6-Testpräfix |
| XRv-10 `Gi0/0/0/5` | keine | `shutdown` |
| Client-101 `Gi0/1` | keine | `shutdown` |

## VRF-/BGP-Plan (neu)

| VRF | PE-Paar | RD links / rechts | Route Target | Testnetze |
|---|---|---|---|---|
| KUNDE_A | XRv-5 ↔ XRv-10 | `65000:51` / `65000:101` | `65000:100` | **`51.1.0.0/16`**, **`101.1.0.0/16`** |
| KUNDE_B | XRv-5 ↔ XRv-10 | `65000:52` / `65000:102` | `65000:200` | neue Test-Loopbacks nötig |
| KUNDE_C | XRv-4 ↔ XRv-7 | `65000:41` / `65000:71` | `65000:300` | noch nicht anhand der Geräte belegt |

### Vorschlag für KUNDE_B (noch **nicht** vorhanden)

| Link | PE-IP | CE-IP | CE-Test-Loopback |
|---|---|---|---|
| XRv-5 ↔ Client-51, Port 5 | `172.16.52.1/30` | `172.16.52.2/30` | `192.168.52.1/32` |
| XRv-10 ↔ Client-101, Port 5 | `172.16.102.1/30` | `172.16.102.2/30` | `192.168.102.1/32` |

Für A/B benötigen die gemeinsam verwendeten CEs **getrennte Routingtabellen (VRF-Lite)**. Bei einer VRF-Zuweisung auf IOSv zuvor IP-Adressen sichern: die IP-Adresse kann dabei entfernt werden.

## Underlay-Loopbacks und Locator-Schema

| PE | IPv6-Loopback gemäß Leitfaden | uSID-Locator gemäß Leitfaden |
|---|---|---|
| XRv-5 | `2001:db8:1:1::5/128` | `5f00:0:5::/48` |
| XRv-10 | `2001:db8:1:1::10/128` | `5f00:0:10::/48` |
| XRv-4 | `2001:db8:1:1::4/128` | `5f00:0:4::/48` |
| XRv-7 | `2001:db8:1:1::7/128` | `5f00:0:7::/48` |

Die Werte stammen aus dem Nummerierungsschema des ursprünglichen Leitfadens und sind **auf den betreffenden PEs zu prüfen**.
