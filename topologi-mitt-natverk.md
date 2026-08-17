# Topologi mitt natverk

Visuell presentation av natverkstopologin for drift, felsokning och dokumentation.

```mermaid
flowchart LR
    classDef wan fill:#e0f2fe,stroke:#075985,color:#082f49
    classDef edge fill:#dcfce7,stroke:#15803d,color:#14532d
    classDef infra fill:#fef3c7,stroke:#b45309,color:#78350f
    classDef app fill:#fee2e2,stroke:#b91c1c,color:#7f1d1d

    Internet[Internet]:::wan --> DuckDNS[DuckDNS\nrudbergloggar / rud4berg / rud4bergimmich]:::wan
    DuckDNS --> UniFi[UniFi Router\nPort forward TCP 80, 443]:::edge

    subgraph LAN[Lokalt natverk 192.168.1.0/24]
        Proxmox[Proxmox host]:::infra
        LXC[LXC 105\n192.168.1.65]:::infra
        Nginx[Nginx Reverse Proxy\n80 and 443]:::app
        Flask[Flask and Gunicorn\n127.0.0.1:8080]:::app
        HA[Home Assistant\n192.168.1.166:8123]:::app
        ESP32[ESP32 Greenhouse\n192.168.1.125]:::app
        Immich[Immich\n192.168.1.24:2283]:::app

        Proxmox --> LXC --> Nginx --> Flask
        Nginx --> HA
        Nginx --> ESP32
        Nginx --> Immich
    end

    UniFi --> Proxmox
```

## Exponerade domaner och mal

- `rudbergloggar.duckdns.org` -> Nginx i LXC (`192.168.1.65`)
- `rud4berg.duckdns.org` -> proxas vidare till Home Assistant (`192.168.1.166:8123`)
- `rud4bergimmich.duckdns.org` -> proxas vidare till Immich (`192.168.1.24:2283`)

## Trafikflode

1. Klient fragar DuckDNS om doman.
2. Trafik gar till publik IP och skickas via UniFi port forwarding till `192.168.1.65`.
3. Nginx valjer backend baserat pa host och path.
4. Flask hanterar loggsidor, medan ovriga requests proxas till respektive LAN-tjanst.
