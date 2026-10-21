# Atlas Industrie - network architecture repository
Design of the Atlas Industrie network, the fictitious company of the
"Advanced Network Architectures" course. One folder per object type;
the addressing plan is authoritative in `data/vlans.yml`.
Local check: `python3 scripts/check_vlans.py` from the repository root.
IPv6 ULA prefix of the project: fd64:6e93:e078::/48
## What the check does and does not do
It detects duplicate VLAN identifiers and IPv4/IPv6 overlaps.
It does NOT validate mandatory fields, nor the consistency between
VLAN, zone and subnet.