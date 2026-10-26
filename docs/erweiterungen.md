# Phase 4 – Erweiterungen

Nach erfolgreichem Basis-L3VPN:

1. **Zweite VRF `KUNDE_B`:** neue Route Targets (z. B. 65000:200), zwei weitere angeschlossene Kundenstandorte. Bewusst überlappende IPv4-Kundennetze nutzen und Isolation prüfen.
2. **PE–CE eBGP:** statische Routen durch dynamischen Austausch ersetzen; Redistribution kontrollieren.
3. **Mehr PEs:** zusätzliche Standorte an vorhandenen Client-Routern nutzen.
4. **Route Reflector:** einen Core-Router ergänzend als BGP-RR für VPNv4 einsetzen; er muss dafür nicht selbst CE-VRFs führen.
5. **Flex-Algo 128:** gezielte Bindung von Service-SIDs an den bereits vorhandenen Delay-Locator prüfen; das ist ein separater Test der Plattformunterstützung.
6. **TI-LFA:** während eines kontinuierlichen Kundendatenstroms einen Core-Link ausfallen lassen, Paketverlust messen und mit IS-IS-Fast-Reroute-Status abgleichen.

Jede Erweiterung erst durchführen, wenn die vorangegangene Phase verifiziert ist.
