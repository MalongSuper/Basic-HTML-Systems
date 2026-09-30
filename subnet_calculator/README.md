# Subnet Calculator

A single-file, dependency-free HTML tool (`Subnet_Calculator.html`) that takes a parent IPv4 network and a list of host requirements, then produces:

- an **IP address report** (type, class, mask, network, broadcast, usable range, and so on),
- a **VLSM subnetting plan** (one subnet per requirement, largest first),
- an animated **network topology diagram** (router, subnets, computers) that can be saved as a PNG.

Open the file in any modern browser. No build step, server, or network access is needed.

---

## 1. Workflow

1. **Enter the parent network.** Type an IPv4 address and a CIDR (for example `192.168.10.0` and `24`).
2. **Add subnets.** Use **+ Add subnet** and enter the number of hosts each subnet must support. Use **Remove** to delete a row. At least one subnet is required; the limit is 32.
3. **Click Calculate.** The script then does the following:
   1. Parses and validates the address and CIDR.
   2. Reads each host count and rejects anything that isn't a whole number of at least 1.
   3. Sorts the requirements **largest first** so blocks stay aligned.
   4. Masks the address down to the parent's network address.
   5. Converts each host count into a block size (a power of two), then into a CIDR prefix.
   6. Allocates the blocks one after another from the start of the parent network, checking alignment and that they fit.
   7. Builds the report table, the plan table, and the topology diagram.
4. **Read the results.** Tables and diagram animate in. Export either table as CSV/JSON, or save the diagram with **Save topology as PNG**.

```
form inputs → parse/validate → sort by size → block size → CIDR → sequential allocation → tables + diagram
```

---

## 2. How to use it (and always get the correct subnet)

### Input rules

| Rule | Why it matters |
| --- | --- |
| Enter the **parent** network's CIDR, not the subnets' | `192.168.10.0/24` is the big block being split. The subnet prefixes are calculated for you. |
| Any address inside the network works | The address is masked down to its network. `192.168.10.77/24` is treated as `192.168.10.0/24`. |
| Count **every address a subnet needs** | Include the router/gateway interface (+1 per subnet), servers, printers and planned growth. The topology draws a router on every subnet, so each one needs a gateway address. |
| Make sure the parent is big enough | The sum of all block sizes must not exceed the parent's size (see below). |

### How block sizes work

Each subnet reserves 2 extra addresses (network and broadcast). The calculator picks the smallest power of two that fits `hosts + 2`, with a minimum block of 4 (a `/30`).

| Hosts needed | Block size | CIDR | Usable hosts |
| --- | --- | --- | --- |
| 1–2 | 4 | /30 | 2 |
| 3–6 | 8 | /29 | 6 |
| 7–14 | 16 | /28 | 14 |
| 15–30 | 32 | /27 | 30 |
| 31–62 | 64 | /26 | 62 |
| 63–126 | 128 | /25 | 126 |
| 127–254 | 256 | /24 | 254 |

Note that a requirement like 63 hosts jumps to a /25, because a /26 only holds 62.

### Checklist before trusting a result

1. **Capacity:** add up the block sizes. The total must be ≤ the parent size (for example `/24` = 256).
2. **Usable ≥ requested:** in the plan table, *Usable hosts* must be at least *Requested hosts* on every row.
3. **Alignment:** every *Network* address must be a multiple of its block size (a /26 must start at .0, .64, .128 or .192).
4. **No overlap:** each *Network* should start right after the previous *Broadcast*.
5. **Naming:** subnets are named `Subnet 1…N` by **form row order**, but the plan is listed **largest first**, so the table order can differ from the form order.

### Worked example

Input: `192.168.10.0/24`, hosts `60, 30, 14, 6`.

| Subnet | Requested | Block | CIDR | Network | First usable | Last usable | Broadcast |
| --- | --- | --- | --- | --- | --- | --- | --- |
| Subnet 1 | 60 | 64 | /26 | 192.168.10.0 | .1 | .62 | .63 |
| Subnet 2 | 30 | 32 | /27 | 192.168.10.64 | .65 | .94 | .95 |
| Subnet 3 | 14 | 16 | /28 | 192.168.10.96 | .97 | .110 | .111 |
| Subnet 4 | 6 | 8 | /29 | 192.168.10.112 | .113 | .118 | .119 |

120 of 256 addresses are used; `192.168.10.120` onward is free.

### Error messages

| Message | Meaning and fix |
| --- | --- |
| `Subnet N is not aligned on a /X boundary` | The next free address isn't a valid start for that block. This only happens if a block is placed out of order or the parent network starts at an unusual point. Check the parent address and CIDR. |
| `Subnet allocations exceed the parent network` | The blocks add up to more than the parent holds. Use a shorter CIDR (bigger parent) or reduce host counts. |
| `Each subnet needs at least 1 host` | A host field is empty, zero, negative or not a whole number. |
| `Each octet must be between 0 and 255` / `Enter an IPv4 address as n1.n2.n3.n4` | The address is malformed. |
| `CIDR must be between 0 and 32` | The prefix is out of range. |

### Known limitations

- IPv4 only.
- Allocation is contiguous from the start of the parent network; it never leaves gaps or packs around existing subnets.
- *Private IP* means the RFC 1918 ranges only (`10/8`, `172.16/12`, `192.168/16`). Loopback and link-local addresses show as Public.
- For `/31` and `/32` parents the report's usable-host formula (`2^(32-CIDR) - 2`) gives 0 and -1; these are special cases not modelled here.
- *Subnet bits* and *Usable subnets* in the report are classful (CIDR minus the default class prefix), so they show a note for Class D/E or supernets.

---

## 3. Key methods

All logic lives in the `<script>` block of `Subnet_Calculator.html`.

### Parsing and conversion

| Method | Purpose |
| --- | --- |
| `parseOctets(text)` | Splits `n1.n2.n3.n4` into four validated numbers (0–255). |
| `parseCidr(text)` | Accepts `24` or `/24`; validates 0–32. |
| `ipToInt(n1, n2, n3, n4)` | Converts four octets to an unsigned 32-bit integer. |
| `intToIp(value)` | Converts a 32-bit integer back to dotted decimal. |
| `dottedToBinary(dotted)` | Dotted decimal to space-separated 8-bit binary groups. |
| `prefixMask(c)` | Returns the 32-bit mask integer for a prefix length. |
| `networkBase(n1, n2, n3, n4, c)` | Address AND mask: the parent's network address (unsigned). |

### Address model (report table)

| Method / class | Purpose |
| --- | --- |
| `IPAddress` | Holds an address and CIDR; getters for `Host`, `Mask`, `Subnet` (network ID) and `Broadcast` in binary and decimal. |
| `ExtendedAddress` | Extends `IPAddress` with `IsPrivate` and `IPType` (Private/Public). |
| `networkAddress(ip)` | Returns the class: A (1–127), B (128–191), C (192–223), D (224–239) or E (240–255). |
| `defaultPrefix(ip)` | Default classful prefix: 8, 16 or 24 (null for D/E). |
| `hostBits(c)`, `subnetBits(c, ip)` | Host bits (`32 - CIDR`) and subnet bits (`CIDR - default prefix`). |
| `usableHosts(c)` | `2^(32-c) - 2`. |
| `countUsableSubnets(c, ip)` | Classful subnet count; throws for supernets or Class D/E. |
| `firstLastFromNetwork(net, bcast)` | First and last usable addresses (network + 1, broadcast - 1). |

### VLSM engine (plan table)

| Method | Purpose |
| --- | --- |
| `getRequiredHosts(hosts)` | Converts each host count into a block size: the smallest power of two that fits `hosts + 2`, minimum 4. |
| `getCidr(requiredHosts)` | Block size to prefix length (`32 - log2(block)`). |
| `subnettingPlan(base, prefix, hosts)` | Allocates blocks sequentially from the base address; checks alignment and fit; returns network, broadcast, first/last usable and usable count for each subnet. |
| `compute()` | Orchestrator: reads the form, validates, **sorts largest first**, builds the report rows, plan rows and plan objects. |

### Interface

| Method | Purpose |
| --- | --- |
| `addSubnet(hosts, animate)` / `removeSubnet(row)` | Add or animate-remove a subnet row. |
| `refreshRows()` / `activeRows()` | Renumber labels; disable Remove on the last row and Add at 32 rows. |
| `initSubnets()` | Seeds the default rows (60, 30, 14, 6). |
| `fillTable(table, rows)` | Renders rows with a staggered fade-in. |
| `drawDiagram(parentNet, plan)` | Builds the topology SVG: router, subnet boxes, computers, animated links. |
| `computerIcon(cx, y, delay)` | SVG computer icon. Icons are capped at 4 per subnet (`MAX_HOST_ICONS`), with a "+N more hosts" label. |
| `savePng()` | Clones the SVG, bakes in static styles, renders at 2× on a canvas and downloads `network-topology.png`. |
| `toCsv()` / `download()` | CSV and JSON export helpers. |
| `showError()` / `clearError()` / `escapeHtml()` | Error banner and safe text output. |
