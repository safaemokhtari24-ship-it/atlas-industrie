\# Decision note - architecture of the Atlas Industrie campus



\## 1. Context and requirements

Atlas Industrie runs two buildings: A (head office and IT, 6 access switches, IT servers) and B (production, 4 access switches, production supervision). The network carries IT, voice, guests, OT (automata and supervision) and a DMZ, split into 11 VLANs (see `data/vlans.yml`).



Stated requirements:

\- Production (building B) must continue to run if building A is isolated.

\- The OT zone must be separated from IT by a firewall, with a default-deny principle between IT and OT.

\- Target availability of the core: 99.99 %.

\- Addressing plan without overlap, IPv4 and IPv6 (ULA `fd64:6e93:e078::/48`), validated automatically.



\## 2. Technical targets retained

The retained architecture is the one of the chapter diagram (`docs/schemas/campus-logique.mmd`).

\- Trust zones, each with its own address block: infrastructure, IT, voice, guests, OT, DMZ, plus VLAN 99 (unused ports, no address, routes nothing).

\- Firewall pair FW-A / FW-B. FW-A is the Internet entry point and is linked to CC-A and to FW-B; FW-B is linked to CC-B. The firewalls enforce the zone policy. The DMZ (VLAN 60) hosts the services exposed to the outside and is reachable only through a firewall.

\- Default-deny between IT and OT: nothing is allowed from IT to OT or from OT to IT except flows explicitly listed (for example supervision). One rule on the block 10.42.64.0/20 covers all OT VLANs.

\- Two L3 cores, CC-A in building A and CC-B in building B, linked by two fibres on distinct routes.

\- 10 access switches (6 in A, 4 in B), 48-port PoE+, each attached to its local core with a second uplink to the other core.

\- IT servers in building A (CC-A), production supervision in building B behind the industrial access switches.



\## 3. The two architectures compared

Three scenarios were priced.



| | S1 three tiers | S2 distributed collapsed core | S3 single-site collapsed core |

|---|---|---|---|

| Cores | 2 modular L3 | 2 fixed 10/25G L3 (CC-A in A, CC-B in B) | 2 fixed 10/25G L3, both in A |

| Distribution | 4 L3 | none | none |

| Access | 10 | 10 | 10 |

| Firewalls | 2 | 2 | 2 |

| Devices | 18 | 14 | 14 |

| Production keeps running if building A is isolated | yes | yes | no |



S1 adds a distribution tier that a campus of this size does not need. S3 is simple and cheap but keeps both cores in building A.



\## 4. Five-year costing

Prices in MAD excluding tax (1 EUR = 10.8 MAD). Support: 10 % of CAPEX per year. Operations: 1.5 days per device per year at 5,400 MAD a day (8,100 MAD per device and year), firewalls included. Horizon: 5 years.



CAPEX detail:

\- S1: 2 x 162,000 + 4 x 54,000 + 10 x 27,000 + 216,000 + 129,600 = 1,155,600

\- S2: 2 x 86,400 + 270,000 + 216,000 + 108,000 = 766,800

\- S3: 2 x 86,400 + 270,000 + 216,000 + 86,400 = 745,200



| | S1 | S2 | S3 |

|---|---|---|---|

| Devices (with firewalls) | 18 | 14 | 14 |

| CAPEX | 1,155,600 | 766,800 | 745,200 |

| Support per year | 115,560 | 76,680 | 74,520 |

| Operations per year | 145,800 | 113,400 | 113,400 |

| Support over 5 years | 577,800 | 383,400 | 372,600 |

| Operations over 5 years | 729,000 | 567,000 | 567,000 |

| \*\*TCO 5 years\*\* | \*\*2,462,400\*\* | \*\*1,717,200\*\* | \*\*1,684,800\*\* |



TCO = CAPEX + 5 x (support + operations).



Gaps:

\- S2 against S1: 1,717,200 - 2,462,400 = \*\*-745,200 MAD\*\* (S2 is 30.3 % cheaper).

\- S2 against S3: 1,717,200 - 1,684,800 = \*\*+32,400 MAD\*\* (S2 is 1.9 % more expensive).



\## 5. Decision and justification

\*\*Retained: S2, distributed collapsed core.\*\*



Against S1: S2 meets the same requirements with 4 fewer devices and 745,200 MAD less over five years. The distribution tier brings no requirement we have, only cost and one more layer to operate.



Why not S3, the cheapest? Not because it is worse, but because it does not satisfy a stated requirement: production must continue if building A is isolated, and in S3 both cores are in A. S3 is therefore not a cheaper equivalent solution, it is a different solution. It serves to price the requirement: the requirement costs \*\*32,400 MAD over five years\*\*.



Is the extra cost worth it? At the assumed 108,000 MAD per hour of production outage, 32,400 MAD is 0.3 hour, that is \*\*18 minutes of avoided outage over five years\*\*. If building A is isolated for more than 18 minutes once in five years, S2 has paid for itself.



Security: the firewall pair sits between the cores and the outside, the DMZ is reachable only through a firewall, and default-deny between IT and OT is applied on the OT block, which the address hierarchy allows in a single rule.



\## 6. Risks and points of failure

\- \*\*The 99.99 % target is not reached.\*\* Three devices in series at 99.9 % give 0.999^3 = 99.70 %. Doubling the core and the firewall gives (1 - 0.001^2)^2 x 0.999 = 99.90 %: the non-redundant access switch sets the ceiling, below 99.99 %. To claim more, the access layer, the power supplies, the fibre routes and the failover mechanisms would have to be analysed.

\- \*\*Common causes are not in the figure.\*\* The parallel model assumes independent failures. Same software version on both cores, same cable chamber (see `docs/schemas/campus-physique.md`), same utility substation: these are not independent. The theoretical figure always overstates.

\- \*\*Internet and IT servers depend on building A.\*\* In the chapter diagram, Internet enters through FW-A and the IT servers sit on CC-A. If building A is isolated, production keeps running locally on CC-B, but building B loses Internet access and the IT servers. S2 protects production, not every service.

\- Each access switch is a single point of failure for its own ports, even with a second uplink.

\- Two fibres between CC-A and CC-B, but the assumed routes meet again in the cable chamber under the courtyard.

\- Failover of the firewall pair and of the virtual gateway (.1 in VLAN 10) is not tested at the design stage.



\## 7. Assumptions and limits

Assumptions:

\- Prices are the course orders of magnitude (1 EUR = 10.8 MAD), not quotations.

\- Cost of one hour of outage: 108,000 MAD.

\- Firewall pair priced identically in the three scenarios and counted in operations.

\- Support 10 % of CAPEX per year, flat; no inflation, no discounting, no residual value.

\- The split of the 4 access switches of building B between industrial (OT) and office use is assumed, the inventory does not specify it.

\- The cabling plan (rooms, north gallery, south facade, cable chamber, UPS A and UPS B, shared substation) is \*\*invented\*\* for the exercise and must be checked on site.



Excluded from the costing: power and cooling, civil works, network cards, licences beyond the 10 % support, training, WAN/Internet subscription, decommissioning.



Limits: the check script validates duplicate VLAN identifiers and IPv4/IPv6 overlaps only, not mandatory fields nor the consistency between VLAN, zone and subnet.



