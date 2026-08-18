# Topologi mitt natverk

Visuell presentation av nätverkstopologin baserad på den aktuella infrastrukturen, inklusive VLAN, switchar, access points, firewall och IP-segment.

```mermaid
%%{init: {'theme': 'base', 'themeVariables': {
  'background': '#050b14',
  'primaryColor': '#0b1220',
  'primaryTextColor': '#e2e8f0',
  'primaryBorderColor': '#1d4ed8',
  'lineColor': '#1d4ed8',
  'secondaryColor': '#0b1220',
  'tertiaryColor': '#0b1220',
  'fontFamily': 'Segoe UI, Arial, sans-serif',
  'fontSize': '14px'
}}}%%
flowchart LR
    classDef isp fill:#0b1220,stroke:#60a5fa,color:#e2e8f0,stroke-width:2px
    classDef switch fill:#0b1220,stroke:#1d4ed8,color:#e2e8f0,stroke-width:2px
    classDef ap fill:#0b1220,stroke:#60a5fa,color:#e2e8f0,stroke-width:2px
    classDef client fill:#111827,stroke:#94a3b8,color:#e2e8f0,stroke-width:2px
    classDef iot fill:#111827,stroke:#7dd3fc,color:#e2e8f0,stroke-width:2px
    classDef server fill:#111827,stroke:#f59e0b,color:#fef3c7,stroke-width:2px

    linkStyle default stroke:#1d4ed8,stroke-width:2px,color:#dbeafe

    ISP[ISP]:::isp --> USG[USG 3P]:::switch
    USG --> MainSwitch[Main Switch]:::switch

    MainSwitch --> MainSwitch1[Main Switch 1]:::switch
    MainSwitch --> MainSwitch2[Main Switch 2]:::switch
    MainSwitch --> IoTSwitch[IoT Switch]:::switch

    MainSwitch1 --> Media[Media Vardagsrum]:::client
    MainSwitch2 --> APGarage[AP Garage]:::ap
    IoTSwitch --> APHall[AP Hall]:::ap

    APGarage --> Laptop[Laptop / PC]:::client
    APGarage --> Phone[Smartphone / Tablet]:::client
    APHall --> Guest[Guest Wi‑Fi]:::client
    IoTSwitch --> IoT1[IoT-sensorer]:::iot
    IoTSwitch --> IoT2[Smart devices]:::iot
    MainSwitch1 --> TV[TV / Media]:::client

    USG -. "NAT / Port forward 80, 443" .-> Nginx[Nginx Reverse Proxy]:::server
    Nginx --> HA[Home Assistant]:::server
    Nginx --> Immich[Immich]:::server
    Nginx --> ESP32[ESP32 Greenhouse]:::server

    style MainSwitch fill:#0b1220,stroke:#1d4ed8,color:#e2e8f0,stroke-width:3px
    style APGarage fill:#0b1220,stroke:#60a5fa,color:#e2e8f0,stroke-width:2px
    style APHall fill:#0b1220,stroke:#60a5fa,color:#e2e8f0,stroke-width:2px
```

## Exponerade tjänster och domäner

- `rudbergloggar.duckdns.org` -> Nginx i LXC (`192.168.1.65`)
- `rud4berg.duckdns.org` -> vidare till Home Assistant (`192.168.1.166:8123`)
- `rud4bergimmich.duckdns.org` -> vidare till Immich (`192.168.1.24:2283`)

## IP-plan och logiska segment

- VLAN 10 - Server / Management: `192.168.1.0/24`
- VLAN 20 - Clients: `192.168.2.0/24`
- VLAN 30 - IoT / sensorer: `192.168.3.0/24`
- VLAN 40 - Guest: `192.168.4.0/24`

## Trafikflöde

1. Klienten ansluter via Wi‑Fi eller kabel.
2. Trafiken går via access point och switch till USG / firewall.
3. USG tillämpar NAT, VLAN-policy och port forwarding.
4. Extern trafik till port 80/443 dirigeras till Nginx i LXC.
5. Nginx beslutar backend baserat på värdnamn och endpoint.
6. Flask hanterar loggsidor, medan andra tjänster proxas vidare till HA, Immich eller ESP32.

## Notering

Detta diagram är baserat på den visade topologin med central switch, sekundära switchar, AP:er och logiska VLAN-segment för att spegla den faktiska nätverkslayouten i drift.
