```
   ■■■■◣                            [▬▬]
   ■ \          /\/\/\/\/\        [▬▬▬]  #####  #    #  #####  ####  #####   ##### #    #  ####
      \       /            \       //    #    # #    # #      #    # #    # #      #   #  #    #
       \O===<     DIG IN     >===O//     #####  #    # #   ## #    # #####  #      ####   #    #
        \   |  \          /  |   //      #    # #    # #    # #    # #  #   #      #   #  ######
            |   X        X   |           #    # #    # #    # #    # #   #  #      #    # #    #
            |   .\/.\/.\/.   |           #####   ####   ####   ####  #    #  ##### #    # #    #
             \______________/
              \/  \/  \/  \/
```

# bugorcka

[![Latest release](https://img.shields.io/github/v/release/bugorcka/bugorcka-miner)](https://github.com/bugorcka/bugorcka-miner/releases)
[![Downloads](https://img.shields.io/github/downloads/bugorcka/bugorcka-miner/total?label=downloads&logo=github)](https://github.com/bugorcka/bugorcka-miner/releases)
[![License](https://img.shields.io/badge/license-proprietary-lightgrey)](LICENSE)
[![Issues](https://img.shields.io/github/issues/bugorcka/bugorcka-miner)](../../issues)
[![Discord](https://img.shields.io/badge/Discord-join%20server-5865F2?logo=discord&logoColor=white)](https://discord.gg/BMZ3fQgyT)

Multi-algorithm CUDA GPU miner. Windows / Linux / HiveOS. Closed source, binary
releases only. The banner above is not a logo - it is `bugorcka`'s actual
startup output, unedited.

This repository hosts the official release packages. Grab the latest build from the [**Releases**](../../releases/latest) page.

## On this page

- [Algorithms, pools and dev fee](#algorithms-pools-and-dev-fee)
- [Features](#features)
- [Performance](#performance)
- [Downloads](#downloads)
- [Supported GPUs](#supported-gpus)
- [Quick start](#quick-start)
- [HiveOS setup](#hiveos-setup)
- [Command line](#command-line)
- [Troubleshooting](#troubleshooting)
- [Notes](#notes)
- [License](#license)

## Algorithms, pools and dev fee

One binary carries every algorithm - pick the coin with `-a`. Each algorithm has
its own dev fee, set independently as new algorithms land. The rate in effect is
shown in the startup banner and the dashboard. `-df` can only raise the `btxv4` fee;
the other algorithms use a fixed rate and ignore it.

| Algorithm (`-a`) | Coin | Consensus | Pools | Dev fee | Status |
|---|---|---|---|---|---|
| `btxv4` (MatMul v4) | BTX | GPU PoW | [LuckyPool](https://btx.luckypool.io/) `btx-eu.lproute.com:8666` (SSL/TLS and plain TCP) | **2%**, raise with `-df <n>` (values <= 2 mean 2) | live |
| `numen` (Proof of Scan) | NUMN | GPU PoW | ninjaraider `numen.ninjaraider.com` - TCP `:44960`, SSL/TLS `:44961` · rplant `stratum-eu.rplant.xyz:7189` (TCP, SSL/TLS `:17189`; regions eu, ru, asia, na) · nitopool `stratum.nitopool.online:3028` · diffpool `http://stratum.diffpool.xyz:8010` | **5%** (fixed) | live |
| `sha256d` (Compute Substrate) | CSD | GPU PoW | LuckyPool `csd-<region>.lproute.com:8760` (SSL/TLS) | **1%** (fixed) | live |

More algorithms land here as new coins get added - this list grows, it
doesn't get replaced.

## Features

- **Multi-algorithm**: BTX (`btxv4`), NUMEN (`numen`) and CSD (`sha256d`) mine from the
  same binary, selected with `-a`
- **Pool list with automatic switching** for NUMEN: `-o ninjaraider,rplant-eu,nitopool` - main pool first, the rest are backups
  (known pools also by short name)
- Pool mining (TCP / SSL-TLS stratum) - each algorithm auto-detects its own
  pool dialect, including our **own stratum-bridge dialect** for BTX
- Live **TUI dashboard** (hashrate sparkline, per-GPU temps/fans/power, pool
  status) or plain streaming log for files and HiveOS
- **JSON stats API** (`-api-port`) for HiveOS and external monitors
- Per-GPU selection (`-d 0,1,2`), **intensity duty-cycling** (`-i 1..21`) to cap heat/power
- **GPU overclocking** - per-card core/memory clock lock/offset, power limit (absolute or `%`), and fan control
- **Automatic thermal pause** (`-tstop`/`-tstart`) - mining pauses when a card gets too hot, resumes once it cools
- Self-contained binaries - no CUDA Toolkit needed on the rig, only the NVIDIA driver

## Performance

Live hashrate, measured on real rigs (not `-benchmark`):

**BTX (`btxv4`)**

| Architecture | Card | Power | Hashrate |
|---|---|---|---|
| Turing | RTX 2060 SUPER | 100 W | 0.20 H/s |
| Turing | RTX 2080 | 120 W | 0.28 H/s |
| Ampere | RTX 3080 | 200 W | 0.52 H/s |
| Ampere | RTX 3080 Ti | 230 W | 0.60 H/s |
| Ada | RTX 4060 Ti | 120 W | 0.41 H/s |
| Blackwell | RTX 5070 Ti | 220 W | 0.89 H/s |
| Blackwell | RTX 5080 | 250 W | 1.05 H/s |

**NUMEN (`numen`)**

| Architecture | Card | Power | Hashrate |
|---|---|---|---|
| Turing | RTX 2060 SUPER | 80 W | 272 KH/s |
| Ampere | RTX 3070 | 105 W | 650 KH/s |
| Ada | RTX 4060 Ti | 107 W | 779 KH/s |
| Blackwell | RTX 5070 Ti | 208 W | 1589 KH/s |

**CSD (`sha256d`)**

The dev fee is not subtracted from these figures (the 1% fee takes that share of mining time).

| Architecture | Card | Power | Hashrate | Efficiency |
|---|---|---|---|---|
| Turing | RTX 2060 SUPER | 98 W | 1.68 GH/s | 17.2 MH/s per W |
| Ampere | RTX 3070 | 105 W | 2.27 GH/s | 21.7 MH/s per W |
| Ampere | RTX 3080 | 175 W | 3.35 GH/s | 19.2 MH/s per W |
| Ada | RTX 4060 Ti | 102 W | 2.79 GH/s | 27.4 MH/s per W |
| Blackwell | RTX 5070 Ti | 179 W | 6.15 GH/s | 34.5 MH/s per W |

More cards land here as they get measured.

## Downloads

Grab the latest from **[Releases](../../releases/latest)**:

| Asset | Platform |
|---|---|
| `bugorcka-vX.Y.Z_..._win64.zip` | Windows 10/11 x64 (example `start.bat` inside) |
| `bugorcka-vX.Y.Z_..._linux_ubuntu22.tar.gz` | Ubuntu 22.04+ / glibc 2.35+ distros |
| `bugorcka-vX.Y.Z_..._hiveos_ub22.tar.gz` | HiveOS (Ubuntu 22 based images) |

Every asset carries every algorithm - pick the coin with `-a`, no separate
downloads needed.

Requirements: a recent NVIDIA driver (for RTX 50xx use the newest available).
No CUDA Toolkit needed on the rig.

## Supported GPUs

NVIDIA, **Turing and newer**.

| Compute cap | Architecture | Line |
|---|---|---|
| sm_75 | Turing | RTX 20xx |
| sm_86 | Ampere | RTX 30xx |
| sm_89 | Ada Lovelace | RTX 40xx |
| sm_120 | Blackwell | RTX 50xx |

This is a kernel-level requirement, not a check against your card's name -
desktop, laptop and workstation cards of these architectures all work the
same way. Verified on real hardware: RTX 2060 SUPER, 2070 SUPER, 2080, 3070,
3080, 3080 Ti, 4060 Ti, 5070 Ti, 5080.

**Not supported:** Pascal (GTX 10xx) and Volta (V100) - the miner needs
instructions neither architecture has.

## Quick start

BTX, pool (LuckyPool, SSL/TLS):

```
bugorcka -a btxv4 -o stratum+ssl://btx-eu.lproute.com:8666 -u btx1qyourwallet -w rig1
```

NUMEN, pool (ninjaraider, plain TCP):

```
bugorcka -a numen -o numen.ninjaraider.com:44960 -u nu7yourSS58address -w rig1
```

NUMEN, pool (rplant, plain TCP):

```
bugorcka -a numen -o stratum+tcp://stratum-eu.rplant.xyz:7189 -u nu7yourSS58address -w rig1
```

NUMEN, pool (nitopool, plain TCP):

```
bugorcka -a numen -o stratum+tcp://stratum.nitopool.online:3028 -u nu7yourSS58address -w rig1
```

NUMEN, diffpool (HTTP RPC, no stratum):

```
bugorcka -a numen -o http://stratum.diffpool.xyz:8010 -u nu7yourSS58address -w rig1
```

NUMEN, several pools with automatic switching (short names):

```
bugorcka -a numen -o rplant-eu,rplant-ru,nitopool -u nu7yourSS58address -w rig1
```

CSD, pool (LuckyPool, SSL/TLS):

```
bugorcka -a sha256d -o stratum+ssl://csd-eu.lproute.com:8760 -u yourCSDaddress -w rig1
```

Pick specific GPUs, half load, log to file:

```
bugorcka -a btxv4 -o pool-host:port -u btx1qyourwallet -d 0,1,2 -i 12 -l rig1
```

Benchmark (no pool needed):

```
bugorcka -a numen -benchmark -seconds 20
```

## HiveOS setup

Flight Sheet -> Miner -> **Custom**, then fill in:

- **Miner name:** `bugorcka`
- **Installation URL** - the `_hiveos_ub22.tar.gz` asset from
  [Releases](../../releases/latest), e.g.:

  ```
  https://github.com/bugorcka/bugorcka-miner/releases/download/v0.3.0/bugorcka-v0.3.0_hiveos_ub22.tar.gz
  ```
- **Pool URL:** `btx-eu.lproute.com:8666` for BTX (add `stratum+ssl://` for
  TLS), `numen.ninjaraider.com:44960` for NUMEN, or `csd-eu.lproute.com:8760` (with `stratum+ssl://`) for CSD
- **Wallet and worker template:** `%WAL%.%WORKER_NAME%`
- **Extra config arguments:** `-a btxv4`, `-a numen` or `-a sha256d`
  (append any other flags on the same line, e.g. `-a numen -i 18`)

Or import the ready flight sheet as JSON (set your own wallet, worker and the
release **Installation URL** - this example is BTX, swap `-a btxv4` for
`-a numen` and the pool URL for a NUMEN sheet):

```json
{"name":"LP_BTX_bugorcka","isFavorite":true,"items":[{"coin":"BTX","pool_ssl":false,"dpool_ssl":false,"miner":"custom","miner_alt":"bugorcka","miner_config":{"url":"stratum+tcp://btx-eu.lproute.com:8666","miner":"bugorcka","template":"%WAL%.%WORKER_NAME%","install_url":"https://github.com/bugorcka/bugorcka-miner/releases/download/v0.3.0/bugorcka-v0.3.0_hiveos_ub22.tar.gz","user_config":"-a btxv4 "},"pool_geo":[]}]}
```

Set `"pool_ssl":true` and `"url":"stratum+ssl://btx-eu.lproute.com:8666"` for TLS.

Hashrate and per-GPU stats show up natively in the Hive dashboard
(the package wires the JSON stats API to the Hive agent).

## Command line

Flags you'll reach for most:

| Flag | What it does |
|---|---|
| `-a, --algo <name>` | Mining algorithm (required): `btxv4`, `numen` or `sha256d` |
| `--profile <name>` | CSD (`sha256d`) only: `speed` = maximum hashrate, `eff` = maximum hashrate per watt |
| `-o <url[,url2,...]>` | Pool address, e.g. `stratum+ssl://btx-eu.lproute.com:8666` (plain `host:port` works too). NUMEN: a comma-separated list = main pool then backups, known pools by short name (`ninjaraider`, `rplant[-eu\|-ru\|-asia\|-na]`, `nitopool`, `diffpool`, add `-ssl` for TLS) |
| `-u <wallet>` | Payout address for the selected algorithm's coin |
| `-w, --worker <name>` | Worker name, sent as `WALLET.name` (default: `rig1`) |
| `-d, --devices <csv>` | GPU indices to mine on, e.g. `0,1,2` (default: all) |
| `-i, --intensity <1-21>` | GPU load: 21 = full speed, lower idles between episodes |
| `-pl, --power-limit <v\|csv>` | Power limit in watts or `%` of the card's stock limit |
| `-tstop, --gpu-temp-stop <C>` | Pause a card once it hits this temperature |
| `-api-port <port>` | Serve JSON mining stats on `http://127.0.0.1:<port>/` |
| `-benchmark` | Measure hashrate on the selected GPUs and exit, no pool needed |

Full flag list below, or `bugorcka -h`.

```
bugorcka -h


Usage:
  Pool:  bugorcka -o <pool-url> -u <wallet> [-w <worker>] [options]
  Solo:  bugorcka -address <payout-address> [-rpcconnect <host>] [options]
  Bench: bugorcka -benchmark [-d <csv>] [-seconds <n>]

Pool:
  -o <url[,url2,...]>       Pool address, e.g. stratum+ssl://btx.ninjaraider.com:44921
                             numen only: a comma-separated list = main pool first, then backups
                             (automatic switching); known pools also by short name:
                             ninjaraider, rplant[-eu|-ru|-asia|-na], nitopool, diffpool, add -ssl for TLS.
                             (host:port with no scheme also works, plain TCP). Any scheme
                             containing ssl/tls enables TLS.
  -u <wallet>               Your BTX payout address (btx1...).
  -w, --worker <name>       Worker name, sent as "WALLET.name" (default: rig1).
      --pool-protocol <d>   Pool dialect. Default: auto -- detected from the pool address
                            (e.g. *.lproute.com:8666 -> public BTX Profile-1 Stratum;
                            *.rplant.* -> rplant, numen/poscan on port 7189;
                            http://stratum.diffpool.xyz:8010 -> diffpool, numen only).
                            Override: bridge, btxpool, stratum, luckypool, btx-stratum-v1,
                            rplant, diffpool.
  -p <pass>                 Pool password (default: x -- BTX pools ignore it).

Solo (JSON-RPC to your own coin daemon):
  -address <addr>           Payout address for the coinbase reward. Its format and the
                             daemon it talks to depend on the selected -a/--algo.
  -chain <main|test|regtest> Network to mine on (default: main).
  -testnet                  Shorthand for -chain test.
  -regtest                  Shorthand for -chain regtest.
  -rpcconnect <host>        Daemon RPC host (default: 127.0.0.1).
  -rpcport <port>           Daemon RPC port (default: the algorithm's own port for -chain).
  -rpcuser <user>           RPC username (default: read the daemon's .cookie file).
  -rpcpassword <pass>       RPC password (default: read the daemon's .cookie file).
  -rpccookiefile <path>     Path to the daemon's .cookie file.

GPUs and load:
  -d, --devices <csv>       CUDA device indices to mine on, e.g. 0,1,2 (default: all detected).
                             Indices are 0-based over the cards CUDA can see, so on HiveOS
                             they follow whatever CUDA_VISIBLE_DEVICES already left visible.
  -i, --intensity <1-21>    GPU load: 21 = full speed, no pauses (default). Lower values idle
                             the card between episodes -- 10 is about half load, 2 about 10%.
                             Caps heat/power or leaves the card usable. Above 21 is capped.
  -benchmark                Measure each selected GPU's hashrate and exit (does not touch a pool,
                             and ignores -i: throttling lives in the mining loop, not here).
  -seconds <n>              Benchmark duration in seconds (default: 8).
      --list-devices        List every CUDA GPU (index / PCI / name / VRAM / UUID) and exit.
                             The index is what -d and the per-card OC CSV positions refer to.

GPU clocks / power (NVML, needs root/admin; applied once at start; nothing is changed
unless you pass a flag). Each value is EITHER one number = same for every card, OR a
per-card CSV where position = GPU index (0,1,2,...): 0 = leave that card alone, X = reset
that card, else the value. Example on an 8-GPU rig:
    -lc 0,0,0,1800,1300,0,0,1500   -> GPU3=1800MHz, GPU4=1300MHz, GPU7=1500MHz, rest untouched
  -lc, --lock-core <v|csv>  Lock core clock to a fixed MHz (X = release the lock).
  -lm, --lock-memory <v|csv> Lock memory clock (X = release). Common: 810, 5001, 7001, 14000.
  -oc, --offset-core <v|csv> Core clock offset along the V/F curve (may be negative = downclock).
  -om, --offset-memory <v|csv> Memory clock offset (may be negative).
  -pl, --power-limit <v|csv> Cap board power, in watts (e.g. 230) or percent of the card's
                             stock limit (e.g. 80%). Upper limit only, clamped to the card's
                             own min/max range -- can lower draw but not exceed the card.
  -fan, --fan <v|csv>       Set fan to a fixed percent 0-100 (X = back to the driver's auto
                             curve). Applies to every fan on the card. Needs root/admin.
  -rlc, --reset-lock-clock  Reset everything (locks + offsets + power limit + fan) on all -d cards.

Thermal protection (watchdog; off unless -tstop is given):
  -tstop, --gpu-temp-stop <C>  Pause a card once it reaches this temperature; it idles (no
                             mining, so it cools) until it drops to the -tstart threshold.
  -tstart, --gpu-temp-start <C> Resume a paused card at/below this temperature (default: stop-5).

Output:
  -fl, --fulllog            Plain streaming log instead of the live dashboard/TUI. Use this
                             when the output goes to a file or to HiveOS's log viewer.
  -l, --log [name]          Also append the run log to a file (OFF unless this flag is given).
                             With no name -> 'logminer'; -l rig1 -> 'rig1'. Written to the
                             working directory, timestamps kept, colors stripped.
  -nc, --nocolor            Disable colored output (plain black & white).
  -ll, --log-level <lvl>    Log verbosity: error | warn | info | debug (default: info).
                             Lower levels are quieter; debug adds diagnostics.
  -api-port <port>          Serve JSON mining stats on http://127.0.0.1:<port>/ (default:
                             disabled; used by HiveOS/external monitors, see build_hive/).

Developer fee:
  -df, --devfee <n>         Developer fee, percent of mining time. The default is set per
                             algorithm; the startup banner shows the rate in effect. For btxv4
                             values below its base rate are raised to it, higher values raise
                             it (e.g. 5 means 5%). Other algorithms use a fixed rate and ignore it.

Other:
  -h, --help                Show this help and exit.
  -a, --algo <name>         Mining algorithm, REQUIRED: btxv4, numen, or sha256d.
      --profile <name>      Algo profile (sha256d: speed = max hashrate, eff = max hashrate per watt).

Examples:
  bugorcka -o stratum+tcp://btx.ninjaraider.com:44921 -u btx1qexampleaddress -w rig1
  bugorcka -o stratum+ssl://btx.ninjaraider.com:44921 -u btx1qexampleaddress -api-port 21373 -fl
  bugorcka -o my-bridge-host:3333 -u btx1qexampleaddress -w rig1 -d 0,1,2
  bugorcka -o my-bridge-host:3333 -u btx1qexampleaddress -i 12 -l rig1
  bugorcka -benchmark -seconds 20
  bugorcka -address <payout-address> -rpcuser me -rpcpassword secret

Note: solo mining needs the selected algorithm to implement a daemon/RPC backend. None of the
currently compiled algorithms (btxv4, numen, sha256d) do -- solo flags are parsed but refused at
startup; use pool mode.

Available algorithms: btxv4, numen, sha256d
```

Solo/RPC mining (`-address`, own daemon) is not available in this release - NUMEN runs on a
Substrate chain and CSD is pool-only for now. Use pool mode.

## Troubleshooting

- **Hashrate is low right after starting on a new card** - expected, once per
  card. See [Notes](#notes) below.
- **Antivirus flags the binary as a PUA** - common for any GPU miner. The
  binaries here are exactly what the Releases page publishes; add an
  exclusion if needed.
- **`-tstop`/`-tstart` never seems to trigger** - both take degrees Celsius,
  not a percent, and are off by default until you pass `-tstop`.
- **HiveOS dashboard shows no hashrate** - the packaged HiveOS build wires
  `-api-port` to the Hive agent automatically; on a manual/custom flight
  sheet, make sure `-api-port <port>` is present in the config line.
- **Miner exits immediately with an unsupported-GPU error** - check
  [Supported GPUs](#supported-gpus): Pascal (GTX 10xx) and Volta (V100) are
  not supported.
- Questions, bug reports, feature requests -> [Issues](../../issues).
- Live support and general discussion -> [Discord](https://discord.gg/BMZ3fQgyT).

## Notes

- **First start on a new card is slower - this is expected, once per card.** The miner
  auto-tunes its GEMM configuration and caches the result (`~/.cache/bugorcka` on Linux,
  `%LOCALAPPDATA%\bugorcka` on Windows); the cache invalidates itself on miner version,
  GPU or clock changes. All builds ship native RTX 50xx code, so this is just the GEMM
  tune - about a minute per card. Hashrate is low during this warm-up and normal from
  the next start on.
- Antivirus software commonly flags **any** GPU miner as a PUA - the binaries here
  are exactly what the Releases page publishes, nothing else. Add an exclusion if needed.
- Multi-GPU rigs: one instance drives all cards; separate nonce ranges per card,
  no duplicate shares.

## License

(c) 2026 Bugorcka. All rights reserved. Proprietary software - see [LICENSE](LICENSE).
Third-party components and their licenses are listed in
[THIRD-PARTY-NOTICES.txt](THIRD-PARTY-NOTICES.txt).

