# Phase 4 – Erweiterungen

Nach erfolgreichem Basis-L3VPN:

1. **[Zweite VRF `KUNDE_B`](kunde-b.md):** ist jetzt als eigenständige Lab-Phase dokumentiert. Danach bewusst überlappende IPv4-Kundennetze nutzen und Isolation prüfen.
2. **[KUNDE_C: Client-41 und Client-71](kunde-c.md):** dritte getrennte VRF mit XRv-4 und XRv-7; Ist-Adressen auslesen.
3. **PE–CE eBGP:** statische Routen durch dynamischen Austausch ersetzen; Redistribution kontrollieren.
4. **Mehr PEs:** zusätzliche Standorte an vorhandenen Client-Routern nutzen.
5. **Route Reflector:** einen Core-Router ergänzend als BGP-RR für VPNv4 einsetzen; er muss dafür nicht selbst CE-VRFs führen.
6. **Flex-Algo 128:** gezielte Bindung von Service-SIDs an den bereits vorhandenen Delay-Locator prüfen; das ist ein separater Test der Plattformunterstützung.
7. **TI-LFA:** während eines kontinuierlichen Kundendatenstroms einen Core-Link ausfallen lassen, Paketverlust messen und mit IS-IS-Fast-Reroute-Status abgleichen.

Jede Erweiterung erst durchführen, wenn die vorangegangene Phase verifiziert ist.
