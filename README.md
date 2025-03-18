# Projekt Sieci Rozleglej

## Opis projektu

Projekt zakłada konfigurację sieci rozległej (WAN) na bazie topologii, która została zrealizowana w programie Cisco Packet Tracer. Celem projektu było stworzenie złożonej infrastruktury sieciowej, która obejmuje zarówno konfigurację podstawowych usług sieciowych, jak i bardziej zaawansowanych technologii, takich jak VPN GRE, NAT, HSRP, BGP, EIGRP, OSPF, PPP, Frame Relay.

### Zadania w projekcie:
1. **Konfiguracja sieci multivlan (HQ LAN)** z implementacją HSRP.
2. **Konfiguracja routingu IGP i BGP**.
3. **Implementacja NAT** w segmentach sieci prywatnych.
4. **Implementacja PPP i Frame Relay**.
5. **Konfiguracja VPN opartą o GRE**.
6. **Testowanie łączności** między różnymi segmentami sieci.

## Topologia Sieci

![image](https://github.com/user-attachments/assets/42711284-ac5a-48bc-b4ca-21320cc7b875)

## Szczegóły konfiguracji

### 1. Sieci:
- **HQ LAN**: 209.165.200.0/24 (z podziałem na dwa vlany o rozmiarze 128 hostów każdy).
- **Adresy Access HQ**: 209.165.201.0/29.
- **Adresy Access1 i Access2**: 195.29.151.0/30 i 195.29.151.4/30.
- **Sieć Transit**: 81.26.16.0/24.
- **Sieci prywatne**: Remote_A (172.16.11.0/24), Remote_B (172.16.33.0/24).
- **VPN**: 10.10.10.0/24 z trzema tunelami GRE: G1-G3, G3-G4, G4-G2.

### 2. Implementacja technologii:
- **HSRP (Hot Standby Router Protocol)**: Zapewnienie redundancji bram sieciowych w HQ LAN.
- **BGP (Border Gateway Protocol)**: Implementacja eBGP z wykorzystaniem ASN 3925 i 24848.
- **NAT (Network Address Translation)**: Użycie NAT overload do translacji adresów prywatnych na publiczne.
- **PPP (Point-to-Point Protocol) oraz Frame Relay**: Konfiguracja na łączach pomiędzy urządzeniami.
- **GRE (Generic Routing Encapsulation)**: Tunelowanie między różnymi segmentami sieci poprzez trzy tunele GRE.
- **OSPF (Open Shortest Path First)**: Implementacja OSPF w sieci VPN.

### 3. Przebieg zadania:
1. **Skonfiguruj sieci Remote_A i Remote_B**:
   - Urządzenia G1, G2, S_A, S_B, Remote_A, Remote_B.
   - Sprawdzenie łączności z lokalnymi bramami.

2. **Skonfiguruj sieci Access1 i Access2**:
   - Urządzenia ISP1 i ISP2.
   - Sprawdzenie łączności G1-ISP1 oraz G2-ISP2.

3. **Implementacja NAT overload na G1 i G2**:
   - Konwersja adresów prywatnych z sieci Remote_A na adres publiczny WAN.
   - Upewnienie się, że Routing jest poprawnie skonfigurowany i łączność działa.

4. **Skonfiguruj łącza pomiędzy ISP1, ISP2 i ISP3**:
   - Konfiguracja łączy PPP/HDLC na łączu ISP1-ISP3 oraz ISP2-ISP3.
   - Skonfiguruj segment **Label Switching** z użyciem urządzenia **FR_SW**.

5. **Skonfiguruj Routing IGP w segmencie Transit**:
   - Zastosowanie EIGRP i odpowiednia dystrybucja tras.

6. **Skonfiguruj HQ LAN**:
   - Konfiguracja VLAN-ów, subinterfejsów, HSRP dla redundancji bram.

7. **Skonfiguruj eBGP**:
   - Wykorzystanie ASN 3925 i 24848, rozgłoszenie odpowiednich sieci w HQ oraz Transit.

8. **Testowanie łączności**:
   - Upewnienie się, że łączność między Remote_A, Remote_B oraz serwerem w HQ działa poprawnie.

9. **Skonfiguruj tunel GRE**:
   - Skonfiguruj 3 tunele: G1-G3, G3-G4, G4-G2 i zweryfikuj połączenie.

10. **Implementacja OSPF w sieci VPN**:
    - Skonfiguruj OSPF w ramach sieci VPN, rozgłaszając odpowiednie sieci lokalne i tunelowe.

## Testy

Po skonfigurowaniu sieci, przeprowadzono testy łączności, aby upewnić się, że:
- Hosty w Remote_A oraz Remote_B mają łączność z HQ LAN.
- Tunel GRE pomiędzy urządzeniami działa poprawnie.
- Routing IGP oraz BGP są poprawnie skonfigurowane i rozgłaszają odpowiednie sieci.
- Implementacja NAT umożliwia komunikację z siecią zewnętrzną.
