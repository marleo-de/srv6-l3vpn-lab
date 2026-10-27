# Weiterführende Tests

Nach erfolgreicher Grundkonfiguration:

1. **KUNDE_A** mit [Client-51 ↔ Client-101](l3vpn.md) prüfen.
2. **KUNDE_B** auf denselben beiden PEs mit [separaten CE-VRF-Lite-Kontexten](kunde-b.md) aufbauen; Route Targets und Testpräfixe dürfen sich nicht vermischen.
3. **KUNDE_C** zwischen [Client-41 und Client-71](kunde-c.md) in Betrieb nehmen, nachdem deren Ist-Adressen bestätigt sind.
4. **Überlappende IP-Netze** in verschiedenen Mandanten verwenden und Isolation testen.
5. **PE–CE eBGP** als Zielkonfiguration umsetzen, siehe [BGP-Plan](bgp-plan.md); statische Routen nur als Zwischentest.
6. **Route Reflector** statt direkter PE-iBGP-Sessions testen.
7. **Flex-Algo 128** und **TI-LFA** während aktiver Kundendatenströme untersuchen.

Vor jeder nächsten Phase die vorherige auf XRv 9000 tatsächlich im Forwarding verifizieren.
