# Topologi mitt natverk

Visuell presentation av nätverkstopologin baserad på den aktuella infrastrukturen, inklusive VLAN, switchar, access points, firewall och IP-segment.

```mermaid
flowchart LR
    classDef isp fill:#dbeafe,stroke:#1d4ed8,color:#0f172a,stroke-width:2px
    classDef switch fill:#dbeafe,stroke:#1d4ed8,color:#0f172a,stroke-width:2px
    classDef ap fill:#dbeafe,stroke:#1d4ed8,color:#0f172a,stroke-width:2px
    classDef server fill:#fef3c7,stroke:#b45309,color:#78350f,stroke-width:2px
    classDef app fill:#fee2e2,stroke:#b91c1c,color:#7f1d1d,stroke-width:2px
    classDef client fill:#f3f4f6,stroke:#374151,color:#111827,stroke-width:2px

    ISP[ISP]:::isp --> USG[USG 3P]:::switch

    USG --> MainSwitch[Main Switch]:::switch
    MainSwitch --> MainSwitch1[Main Switch 1]:::switch
    MainSwitch --> MainSwitch2[Main Switch 2]:::switch
    MainSwitch --> IoTSwitch[IoT Switch]:::switch
    MainSwitch1 --> Media[Media Vardagsrum]:::client

    MainSwitch2 --> APGarage[AP Garage]:::ap
    IoTSwitch --> APHall[AP Hall]:::ap

    USG -. "NAT / Port forward 80, 443" .-> Nginx

    subgraph VLAN10[VLAN 10 - Server / Management
    192.168.1.0/24]
        Proxmox[Proxmox host
        192.168.1.10]:::server
        LXC[LXC 105
        192.168.1.65]:::server
        Nginx[Nginx Reverse Proxy
        80/443]:::app
        Flask[Flask + Gunicorn
        127.0.0.1:8080]:::app
        HA[Home Assistant
        192.168.1.166:8123]:::app
        Immich[Immich
        192.168.1.24:2283]:::app
        ESP32[ESP32 Greenhouse
        192.168.1.125]:::app
    end

    subgraph VLAN20[VLAN 20 - Clients
    192.168.2.0/24]
        Laptop[Laptop / PC]:::client
        Phone[Smartphone / Tablet]:::client
        TV[TV / Media]:::client
    end

    subgraph VLAN30[VLAN 30 - IoT
    192.168.3.0/24]
        IoT1[IoT-sensorer]:::client
        IoT2[Smart devices]:::client
    end

    subgraph VLAN40[VLAN 40 - Guest
    192.168.4.0/24]
        Guest[Guest Wi‑Fi]:::client
    end

    MainSwitch --> Proxmox
    Proxmox --> LXC
    LXC --> Nginx
    Nginx --> Flask
    Nginx --> HA
    Nginx --> Immich
    Nginx --> ESP32

    APGarage --> Laptop
    APGarage --> Phone
    MainSwitch1 --> TV
    APHall --> Guest
    IoTSwitch --> IoT1
    IoTSwitch --> IoT2
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
