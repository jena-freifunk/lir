# Freifunk Jena — IPv4-Adressraum

Community-Netz: **10.17.0.0/16** (65536 Adressen)  
IPv6-ULA: `fd59:b1b7:1d1f::/64` (Host-Anteil aus MAC)

## Aufteilung

| Bereich | Rolle | Präfix | Adressen |
|---------|-------|--------|----------:|
| 10.17.0.1 – 10.17.15.254 | Speziell / statisch (hier beanspruchen) | /20 | 4094 |
| 10.17.16.2 – 10.17.111.254 | DHCP-Clients | — | 24573 |
| 10.17.112.1 – 10.17.127.254 | LibreMesh-Knoten (automatisch) | /20 | 4094 |
| 10.17.128.1 – 10.17.255.254 | Für die Zukunft reserviert | /17 | 32766 |

```
10.17.0.0/16
├── 10.17.0.0/20      speziell / statisch     10.17.0.1   – 10.17.15.254
├── DHCP              Clients                 10.17.16.2  – 10.17.111.254
├── 10.17.112.0/20    LibreMesh-Knoten        10.17.112.1 – 10.17.127.254
└── 10.17.128.0/17    für die Zukunft reserv. 10.17.128.1 – 10.17.255.254
```

LibreMesh-Optionen:

- `main_ipv4_address '10.17.112.0/16/20'`
- `anygw_dhcp_start '4098'` (10.17.16.2)
- `anygw_dhcp_limit '24573'` (bis 10.17.111.254)

Statische Hosts **nicht** in den DHCP-, Knoten- oder Zukunftsbereichen eintragen.

## Wie LibreMesh das auf DHCP abbildet

`main_ipv4_address` baut den DHCP-Pool **nicht**. Es setzt nur:

1. Community-Subnetz = erstes Präfix → `10.17.0.0/16` (anygw-VIP ≈ `10.17.0.1`, Netzmaske)
2. automatische Knoten-IPs = zweites Präfix → `10.17.112.0/20`

DHCP kommt ausschließlich aus `anygw_dhcp_start` / `anygw_dhcp_limit`.  
`lime-proto-anygw` schreibt sie in die OpenWrt-Datei `/etc/config/dhcp` auf dem Interface `anygw` (dnsmasq):

| UCI-Option | Bedeutung |
|------------|-----------|
| `start` | Host-Offset ab Netzwerkadresse `10.17.0.0` |
| `limit` | Anzahl der Leases (`0` = bis Ende des `/16`) |

Unser Pool:

```
first = 10.17.0.0 + 4098              = 10.17.16.2
last  = 10.17.0.0 + 4098 + 24573 - 1  = 10.17.111.254
```

Ohne explizites `start`/`limit` gelten die Defaults `start=2`, `limit=0` → DHCP würde fast das gesamte `/16` abdecken und sich mit Speziell + Knoten überschneiden. Diese beiden Optionen immer mit der Aufteilung oben synchron halten.

## Speziell / statisch — vorgesehener Bereich

Nur **10.17.0.1 – 10.17.15.254** verwenden.

## Anspruchstabelle (statisch)

Eine freie Adresse im Spezialbereich beanspruchen, bevor sie genutzt wird.  
Diese Datei im Paket bearbeiten und einen PR öffnen.

Regeln:

1. Eine Zeile pro IPv4-Adresse (oder zusammenhängender Block unter *Notizen*).
2. *Beansprucht von* = Person oder Gruppe + Kontakt (Nick / Mail / Matrix).
3. Zeile freigeben, wenn der Dienst weg ist.
4. Niemals außerhalb von **10.17.0.1 – 10.17.15.254** beanspruchen.

| IPv4 | Beansprucht von | Zweck | Seit | Notizen |
|------|-----------------|-------|------|---------|
| 10.17.0.1 | LibreMesh | AnyGW | 2026-09-17 | |
| 10.17.13.0/25 | Martin | Dienste & Richtfunk | 2026-09-17 | |
| … | | | | Bei Bedarf Zeilen ergänzen |
