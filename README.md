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

## 3. Przebieg zadania:
### 1. Konfiguracja sieci Remote_A i Remote_B

### Urządzenia:
- **G1**, **G2**, **S_A**, **S_B**, **Remote_A**, **Remote_B**

### Cel:
- Sprawdzenie łączności z lokalnymi bramami
- Konfiguracja usługi **DHCP** na routerach **G1** i **G2** do automatycznego przydzielania adresów IP hostom w sieci.

### Komendy konfiguracyjne:

#### **Router G1**:
| Komponent                     | Konfiguracja                                                                                       |
|-------------------------------|----------------------------------------------------------------------------------------------------|
| **DHCP**                       | ```ip dhcp excluded-address 172.16.11.1``` <br> ```ip dhcp pool REMOTE_A``` <br> ```network 172.16.11.0 255.255.255.0``` <br> ```default-router 172.16.11.1``` |
| **Interfejs GigabitEthernet0/0/0** | ```ip address 172.16.11.1 255.255.255.0``` <br> |
| **Interfejs Serial0/1/0**      | ```ip address 195.29.151.1 255.255.255.252``` <br> |


#### **Router G2**:
| Komponent                     | Konfiguracja                                                                                       |
|-------------------------------|----------------------------------------------------------------------------------------------------|
| **DHCP**                       | ```ip dhcp excluded-address 172.16.33.1``` <br> ```ip dhcp pool REMOTE_B``` <br> ```network 172.16.33.0 255.255.255.0``` <br> ```default-router 172.16.33.1``` |
| **Interfejs GigabitEthernet0/0/0** | ```ip address 172.16.33.1 255.255.255.0``` <br> |
| **Interfejs Serial0/1/0**      | ```ip address 195.29.151.5 255.255.255.252``` <br> |

### Weryfikacja:
1. Sprawdzenie, czy urządzenia końcowe **Remote_A** i **Remote_B** uzyskują adresy IP w odpowiednich zakresach.
![dhcp](https://github.com/user-attachments/assets/31589ba1-6020-483a-9369-4585bb99a924)
![dhcp2](https://github.com/user-attachments/assets/bb2aad85-7329-41c6-8e35-264eaef8e4cf)

3. Użycie polecenia **ping** do sprawdzenia łączności z bramą lokalną.
![pingGate](https://github.com/user-attachments/assets/41430942-6d9c-4614-b5a3-543c8742f46d)

---

## 2. Konfiguracja sieci Access1 i Access2

### Urządzenia:
- **ISP1**, **ISP2**

### Cel:
- Sprawdzenie łączności **G1-ISP1** oraz **G2-ISP2**.

#### Adresacja łączy:

| Łącze         | Adresacja        |
|---------------|------------------|
| **G1-ISP1**   | **195.29.151.0/30** |
| **G2-ISP2**   | **195.29.151.4/30** |

### Weryfikacja:
1. Upewnienie się, że **G1** ma łączność z **ISP1**, a **G2** z **ISP2** (sprawdzenie z użyciem **ping**).

---

## 3. Implementacja NAT overload na G1 i G2

### Cel:
- Konwersja adresów prywatnych w sieci **Remote_A** i **Remote_B** na adresy publiczne.

#### Konfiguracja na **G1**:
```bash
ip nat inside source list 1 interface Serial0/1/0 overload
access-list 1 permit 172.16.11.0 0.0.0.255
interface Serial0/1/0
 ip nat outside
interface GigabitEthernet0/0/0
 ip nat inside
```

#### Konfiguracja na **G2**:
```bash
ip nat pool GLOB_ADDR 194.126.254.1 194.126.254.1 netmask 255.255.255.252
ip nat inside source list 100 pool GLOB_ADDR overload
access-list 100 deny ip 172.16.33.0 0.0.0.255 172.16.11.0 0.0.0.255
interface Serial0/1/0
 ip nat outside
interface GigabitEthernet0/0/0
 ip nat inside
```

#### Weryfikacja
   - Sprawdzenie łączności pomiędzy Remote_A-ISP1 oraz Remote_B-ISP2 za pomocą ping.


## 4. Konfiguracja łącza pomiędzy ISP1, ISP2 i ISP3

### 4.1 Adresacja łączy szeregowych

| Łącze         | Adresacja        |
|--------------|------------------|
| **ISP1-ISP3** | **209.165.201.0/30** |
| **ISP2-ISP3** | **209.165.201.4/30** |

### 4.2 Konfiguracja segmentu Label Switching  
Zastosowanie urządzenia **FR_SW** do implementacji **MPLS Label Switching**.

---

## 5. Konfiguracja Routing IGP w segmencie Transit

### 5.1 Konfiguracja EIGRP  
Zastosowanie **EIGRP** do dynamicznej dystrybucji tras.

#### ISP1:
```bash
router eigrp 1
 redistribute static
 network 81.26.16.0 0.0.0.3
 network 81.26.16.8 0.0.0.7
 network 195.29.151.0 0.0.0.3
```

#### ISP2:
```bash
router eigrp 1
 network 81.26.16.4 0.0.0.3
 network 81.26.16.8 0.0.0.7
 network 195.29.151.4 0.0.0.3
 network 194.126.254.0 0.0.0.3
```

#### ISP3:
```bash
router eigrp 1
 network 81.26.16.0 0.0.0.3
 network 81.26.16.4 0.0.0.3
 network 209.165.201.0 0.0.0.3
```

#### ISP4:
```bash
router eigrp 1
 passive-interface Serial0/1/1
 network 81.26.16.8 0.0.0.7
 network 209.165.201.4 0.0.0.3
```

## 6. Konfiguracja HQ LAN.
### 6.1 Konfiguracja subinterfejsów i HSRP.

#### G3
```bash
interface GigabitEthernet0/0/0.11
 encapsulation dot1Q 11
 ip address 209.165.200.1 255.255.255.128
 standby 11 ip 209.165.200.127
 standby 11 priority 110
 standby 11 preempt

interface GigabitEthernet0/0/0.22
 encapsulation dot1Q 22
 ip address 209.165.200.130 255.255.255.128
 standby 22 ip 209.165.200.254
```

#### G4
```bash
interface GigabitEthernet0/0/0.11
 encapsulation dot1Q 11
 ip address 209.165.200.2 255.255.255.128
 standby 11 ip 209.165.200.127

interface GigabitEthernet0/0/0.22
 encapsulation dot1Q 22
 ip address 209.165.200.129 255.255.255.128
 standby 22 ip 209.165.200.254
 standby 22 priority 110
 standby 22 preempt
```

### 6.2 Konfiguracja przełącznika S34.
```bash
spanning-tree mode rapid-pvst
spanning-tree vlan 11,22 priority 24576

int range f0/1-9
 switchport access vlan 11
 switchport mode access
 switchport port-security
 switchport port-security mac-address sticky
 spanning-tree portfast
 spanning-tree bpduguard enable

int range f0/10-20
 switchport access vlan 22
 switchport mode access
 switchport port-security
 switchport port-security mac-address sticky
 spanning-tree portfast
 spanning-tree bpduguard enable
```

## 7. Konfiguracja eBGP

### 7.1 Adresacja łączy BGP

| Łącze         | Adresacja          |
|--------------|------------------|
| **ISP3-G3**  | **209.165.201.0/30** |
| **ISP4-G4**  | **209.165.201.4/30** |

### 7.2 ASN i połączenia

| Router  | ASN  | Połączenie               |
|---------|------|--------------------------|
| **ISP3** | 24848 | **209.165.201.1 ↔ G3**  |
| **G3**   | 3925  | **209.165.201.2 ↔ ISP3** |
| **ISP4** | 24848 | **209.165.201.5 ↔ G4**  |
| **G4**   | 3925  | **209.165.201.6 ↔ ISP4** |

### 7.3 Konfiguracja routerów BGP

#### **ISP3**:
```bash
router bgp 24848
 bgp router-id 3.3.3.3
 neighbor 209.165.201.2 remote-as 3925
```

#### **G3**:
```bash
router bgp 3925
 bgp router-id 33.33.33.33
 neighbor 209.165.201.1 remote-as 24848
 network 209.165.200.0 mask 255.255.255.128
```

#### **ISP4**:
```bash
router bgp 24848
 bgp router-id 4.4.4.4
 neighbor 209.165.201.6 remote-as 3925
```

#### **G4**:
```bash
router bgp 3925
 bgp router-id 44.44.44.44
 neighbor 209.165.201.5 remote-as 24848
 network 209.165.200.128 mask 255.255.255.128
```

## 8. Konfiguracja GRE VPN

### 8.1 Adresacja tuneli GRE

| Tunel   | Źródło         | Cel            | Adresacja       |
|---------|--------------|--------------|---------------|
| **TUN1** | **G1: 195.29.151.1** | **G3: 209.165.201.2** | **10.10.10.1/30** |
| **TUN2** | **G3: 209.165.201.2** | **G4: 209.165.201.6** | **10.10.10.5/30** |
| **TUN3** | **G4: 209.165.201.6** | **G2: 195.29.151.5** | **10.10.10.9/30** |

### 8.2 Konfiguracja interfejsów GRE

#### **G1**:
```bash
interface Tunnel1
 ip address 10.10.10.1 255.255.255.252
 tunnel source 195.29.151.1
 tunnel destination 209.165.201.2
```

#### **G3**:
```bash
interface Tunnel1
 ip address 10.10.10.2 255.255.255.252
 tunnel source 209.165.201.2
 tunnel destination 195.29.151.1

interface Tunnel2
 ip address 10.10.10.5 255.255.255.252
 tunnel source 209.165.201.2
 tunnel destination 209.165.201.6
```

#### **G4**:
```bash
interface Tunnel2
 ip address 10.10.10.6 255.255.255.252
 tunnel source 209.165.201.6
 tunnel destination 209.165.201.2

interface Tunnel3
 ip address 10.10.10.9 255.255.255.252
 tunnel source 209.165.201.6
 tunnel destination 195.29.151.5
```

#### **G2**:
```bash
interface Tunnel3
 ip address 10.10.10.10 255.255.255.252
 tunnel source 195.29.151.5
 tunnel destination 209.165.201.6
```

## 9. Konfiguracja OSPF w sieci GRE

### 9.1 Identyfikatory routerów i sieci OSPF

| Router | Router-ID   | Sieci OSPF                |
|--------|------------|---------------------------|
| **G1** | 13.13.13.13 | **10.10.10.0/30, 172.16.11.0/24** |
| **G2** | 24.24.24.24 | **10.10.10.8/30, 172.16.33.0/24** |
| **G3** | 31.31.31.31 | **10.10.10.0/30, 10.10.10.4/30** |
| **G4** | 42.42.42.42 | **10.10.10.4/30, 10.10.10.8/30** |

### 9.2 Konfiguracja OSPF

#### **G1**:
```bash
router ospf 1
 router-id 13.13.13.13
 network 10.10.10.0 0.0.0.3 area 0
 network 172.16.11.0 0.0.0.255 area 0
```

#### **G2**:
```bash
router ospf 1
 router-id 24.24.24.24
 network 10.10.10.8 0.0.0.3 area 0
 network 172.16.33.0 0.0.0.255 area 0
```

#### **G3**:
```bash
router ospf 1
 router-id 31.31.31.31
 passive-interface GigabitEthernet0/0/0.11
 passive-interface GigabitEthernet0/0/0.22
 network 10.10.10.0 0.0.0.3 area 0
 network 10.10.10.4 0.0.0.3 area 0
```

#### **G4**:
```bash
router ospf 1
 router-id 42.42.42.42
 passive-interface GigabitEthernet0/0/0.11
 passive-interface GigabitEthernet0/0/0.22
 network 10.10.10.4 0.0.0.3 area 0
 network 10.10.10.8 0.0.0.3 area 0
```



