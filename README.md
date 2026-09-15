<p align="center">
  <img src="logo.png" alt="bugorcka" width="320">
</p>

# bugorcka

**High-performance multi-algorithm CUDA GPU miner.**
Windows / Linux / HiveOS. Closed source, binary releases only.

> Built from scratch and tuned at SASS level — beats every public miner we
> could measure against, on Turing, Ampere, Ada and Blackwell alike.

## Algorithms

| Algorithm | Coin | Consensus | Status |
|---|---|---|---|
| `btxv4` (MatMul v4) | BTX | GPU PoW | live |
| `numen` (Proof of Scan) | NUMN | GPU PoW | live |

More algorithms land here as new coins get added — this list grows, it
doesn't get replaced.

## Features

- **Multi-algorithm**: BTX (`btxv4`) and NUMEN (`numen`) mine from the same
  binary, selected with `-a`
- NVIDIA GPUs: **Turing and newer** — GeForce RTX 20xx / 30xx / 40xx / 50xx.
  Verified on real hardware across both algorithms: RTX 2060 SUPER, 2070
  SUPER, 3070, 3080, 3080 Ti, 4060 Ti, 5070 Ti, 5080.
  Pascal (GTX 10xx) and Volta (V100) are not supported. Datacenter cards
  (A100 / H100 / B100) are coming in a future release
- Pool mining (TCP / SSL-TLS stratum) — each algorithm auto-detects its own
  pool dialect, including our **own stratum-bridge dialect** for BTX
- Live **TUI dashboard** (hashrate sparkline, per-GPU temps/fans/power, pool
  status) or plain streaming log for files and HiveOS
- **JSON stats API** (`-api-port`) for HiveOS and external monitors
- Per-GPU selection (`-d 0,1,2`), **intensity duty-cycling** (`-i 1..21`) to cap heat/power
- **GPU overclocking** — per-card core/memory clock lock/offset, power limit (absolute or `%`), and fan control
- **Automatic thermal pause** (`-tstop`/`-tstart`) — mining pauses when a card gets too hot, resumes once it cools
- Self-contained binaries — no CUDA Toolkit needed on the rig, only the NVIDIA driver

## Dev fee

Each algorithm has its own default rate, set independently as new algorithms land.

| Algorithm | Default | Raise it |
|---|---|---|
| `btxv4` | **2%** | `-df <n>` — values ≤ 2 mean 2, higher raises it |
| `numen` | **5%** | `-df <n>` — values ≤ 5 mean 5, higher raises it |

## Performance

Live hashrate, measured on real rigs (not `-benchmark`):

**BTX (`btxv4`)**

| Architecture | Card | Power | Hashrate |
|---|---|---|---|
| Turing | RTX 2080 | 120 W | 0.30 H/s |
| Ampere | RTX 3080 | 230 W | 0.60 H/s |
| Ada | RTX 4060 Ti | 120 W | 0.41 H/s |
| Blackwell | RTX 5080 | 250 W | 1.05 H/s |

**NUMEN (`numen`)**

| Architecture | Card | Power | Hashrate |
|---|---|---|---|
| Turing | RTX 2080 | 100 W | 300 KH/s |
| Ampere | RTX 3080 | 140 W | 660 KH/s |
| Ada | RTX 4060 Ti | 100 W | 550 KH/s |
| Blackwell | RTX 5070 Ti | 123 W | 1036 KH/s |
| Blackwell | RTX 5080 | 150 W | 1100 KH/s |

More cards land here as they get measured.

## Downloads

Grab the latest from **[Releases](../../releases/latest)**:

| Asset | Platform |
|---|---|
| `bugorcka-vX.Y.Z_..._win64.zip` | Windows 10/11 x64 (example `start.bat` inside) |
| `bugorcka-vX.Y.Z_..._linux_ubuntu22.tar.gz` | Ubuntu 22.04+ / glibc 2.35+ distros |
| `bugorcka-vX.Y.Z_..._hiveos_ub22.tar.gz` | HiveOS (Ubuntu 22 based images) |

Every asset carries every algorithm — pick the coin with `-a`, no separate
downloads needed.

Requirements: a recent NVIDIA driver (for RTX 50xx use the newest available).
No CUDA Toolkit needed on the rig.

## Supported pools

**BTX**
- **LuckyPool** — https://btx.luckypool.io/
  - EU endpoint: **`btx-eu.lproute.com:8666`** — works in both **SSL/TLS** and **plain TCP** mode.
- Your own **stratum-bridge** (default `bridge` dialect), or any other BTX stratum pool.

**NUMEN**
- **ninjaraider** — TCP `numen.ninjaraider.com:44960`, SSL/TLS `:44961`.

## Quick start

BTX, pool (LuckyPool, SSL/TLS):

```
bugorcka -a btxv4 -o stratum+ssl://btx-eu.lproute.com:8666 -u btx1qyourwallet -w rig1
```

NUMEN, pool (ninjaraider, plain TCP):

```
bugorcka -a numen -o numen.ninjaraider.com:44960 -u nu7yourSS58address -w rig1
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

Flight Sheet → Miner → **Custom**, then fill in:

- **Miner name:** `bugorcka`
- **Installation URL** — the `_hiveos_ub22.tar.gz` asset from
  [Releases](../../releases/latest), e.g.:

  ```
  https://github.com/bugorcka/bugorcka-miner/releases/download/v0.1.4/bugorcka-v0.1.4_hiveos_ub22.tar.gz
  ```
- **Pool URL:** `btx-eu.lproute.com:8666` for BTX (add `stratum+ssl://` for
  TLS), or `numen.ninjaraider.com:44960` for NUMEN
- **Wallet and worker template:** `%WAL%.%WORKER_NAME%`
- **Extra config arguments:** `-a btxv4` or `-a numen` (append any other
  flags on the same line, e.g. `-a numen -i 18`)

Or import the ready flight sheet as JSON (set your own wallet, worker and the
release **Installation URL** — this example is BTX, swap `-a btxv4` for
`-a numen` and the pool URL for a NUMEN sheet):

```json
{"name":"LP_BTX_bugorcka","isFavorite":true,"items":[{"coin":"BTX","pool_ssl":false,"dpool_ssl":false,"miner":"custom","miner_alt":"bugorcka","miner_config":{"url":"stratum+tcp://btx-eu.lproute.com:8666","miner":"bugorcka","template":"%WAL%.%WORKER_NAME%","install_url":"https://github.com/bugorcka/bugorcka-miner/releases/download/v0.1.4/bugorcka-v0.1.4_hiveos_ub22.tar.gz","user_config":"-a btxv4 "},"pool_geo":[]}]}
```

Set `"pool_ssl":true` and `"url":"stratum+ssl://btx-eu.lproute.com:8666"` for TLS.

Hashrate and per-GPU stats show up natively in the Hive dashboard
(the package wires the JSON stats API to the Hive agent).

## Command line

```
bugorcka -h

Pool:
  -o <url>                  Pool address, e.g. stratum+ssl://btx-eu.lproute.com:8666
                             (host:port with no scheme also works, plain TCP). Any scheme
                             containing ssl/tls enables TLS.
  -u <wallet>               Your payout address for the selected algorithm's coin
                             (btx1... for BTX, nu... SS58 for NUMEN).
  -w, --worker <name>       Worker name, sent as "WALLET.name" (default: rig1).
      --pool-protocol <d>   Pool dialect (BTX only, auto-detected otherwise): bridge
                             (default), btxpool, stratum.
  -p <pass>                 Pool password (default: x -- most pools ignore it).

GPUs and load:
  -d, --devices <csv>       CUDA device indices to mine on, e.g. 0,1,2 (default: all).
  -i, --intensity <1-21>    GPU load: 21 = full speed (default). Lower values idle the
                             card between episodes -- 10 is about half load, 2 about 10%.
  -benchmark                Measure each selected GPU's hashrate and exit.
  -seconds <n>              Benchmark duration in seconds (default: 8).
      --list-devices        List every CUDA GPU (index / PCI / name / VRAM / UUID) and exit --
                             indices match -d and the OC/fan CSV positions below.

GPU clocks / power (NVML, needs root/admin; nothing changes unless you pass a flag).
Each value is one number for every card, or a CSV where position = GPU index (X = reset):
  -lc, --lock-core <v|csv>     Lock core clock to a fixed MHz.
  -lm, --lock-memory <v|csv>   Lock memory clock.
  -oc, --offset-core <v|csv>   Core clock offset (can be negative).
  -om, --offset-memory <v|csv> Memory clock offset (can be negative).
  -pl, --power-limit <v|csv>   Power limit in watts (e.g. 230) or percent of the card's
                             stock limit (e.g. 80%).
  -fan, --fan <v|csv>       Fan speed 0-100% (X = back to the driver's auto curve).
  -rlc, --reset-lock-clock  Reset clocks/offsets/power/fan on all -d cards.

Thermal protection (off by default):
  -tstop, --gpu-temp-stop <C>  Pause a card once it hits this temperature.
  -tstart, --gpu-temp-start <C> Resume once it cools to this temperature (default: stop-5).

Output:
  -fl, --fulllog            Plain streaming log instead of the live dashboard/TUI.
  -l, --log <name>          Also append the run log to this file (default: logminer).
  -nc, --nocolor            Disable colored output.
  -ll, --log-level <lvl>    Log verbosity: error | warn | info | debug (default: info).
  -api-port <port>          Serve JSON mining stats on http://127.0.0.1:<port>/.

Developer fee:
  -df, --devfee <n>         Developer fee, percent of mining time (default varies by
                             algorithm, see the Dev fee table above).

Other:
  -h, --help                Show help and exit.
  -a, --algo <name>         Mining algorithm: btxv4 (default) or numen.
```

Solo/RPC mining (`-address`, own daemon) is BTX-only — NUMEN runs on a
Substrate chain and is pool-only for now.

## Notes

- **First start on a new card is slower — this is expected, once per card.** The miner
  auto-tunes its GEMM configuration and caches the result (`~/.cache/bugorcka` on Linux,
  `%LOCALAPPDATA%\bugorcka` on Windows); the cache invalidates itself on miner version,
  GPU or clock changes. All builds ship native RTX 50xx code, so this is just the GEMM
  tune — about a minute per card. Hashrate is low during this warm-up and normal from
  the next start on.
- Antivirus software commonly flags **any** GPU miner as a PUA — the binaries here
  are exactly what the Releases page publishes, nothing else. Add an exclusion if needed.
- Multi-GPU rigs: one instance drives all cards; separate nonce ranges per card,
  no duplicate shares.
- Questions, bug reports, feature requests → [Issues](../../issues).

## License

(c) 2026 Bugorcka. All rights reserved. Proprietary software — see [LICENSE](LICENSE).
Third-party components and their licenses are listed in
[THIRD-PARTY-NOTICES.txt](THIRD-PARTY-NOTICES.txt).
