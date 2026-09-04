<p align="center">
  <img src="logo.png" alt="bugorcka" width="320">
</p>

# bugorcka

**High-performance CUDA GPU miner for BTX** (MatMul v4, algo `btxv4`).
Windows / Linux / HiveOS. Closed source, binary releases only.

> Built from scratch and tuned at SASS level — beats every public BTX miner
> we could measure against, on Turing, Ampere and Blackwell alike.

## Features

- **BTX / MatMul v4** (`btxv4`) — the current live algorithm (Epoch A, v4 chain)
- NVIDIA GPUs: **Turing and newer** — GeForce RTX 20xx / 30xx / 40xx / 50xx.
  Verified on real hardware: RTX 2060 SUPER, 3080 Ti, 5070 Ti, 5080.
  Pascal (GTX 10xx) and Volta (V100) are not supported — the algorithm needs the
  int8 tensor instructions introduced with Turing. Datacenter cards
  (A100 / H100 / B100) are coming in the next release
- Pool mining (TCP / SSL-TLS stratum), **own stratum-bridge dialect**, and **solo mining**
  straight against your coin daemon over JSON-RPC
- Live **TUI dashboard** (hashrate sparkline, per-GPU temps/fans/power, pool status)
  or plain streaming log for files and HiveOS
- **JSON stats API** (`-api-port`) for HiveOS and external monitors
- Per-GPU selection (`-d 0,1,2`), **intensity duty-cycling** (`-i 1..21`) to cap heat/power
- Self-contained binaries — no CUDA Toolkit needed on the rig, only the NVIDIA driver

## Dev fee

Each algorithm has its own default rate, set independently as new algorithms
land. One cycle is 100 minutes, so the percent equals minutes per cycle; every
switch to the fee and back is announced in the log.

| Algorithm | Default | Raise it |
|---|---|---|
| `btxv4` | **2%** | `-df <n>` — values ≤ 2 mean 2, higher raises it |

## Performance

Live hashrate, measured on real rigs (not `-benchmark`):

| Architecture | Card | Power | Hashrate |
|---|---|---|---|
| Turing | RTX 2080 | 140 W | 0.27 H/s |
| Ampere | RTX 3080 Ti | 230 W | 0.50 H/s |
| Blackwell | RTX 5080 | 240 W | 1.00 H/s |

More cards land here as they get measured.

## Downloads

Grab the latest from **[Releases](../../releases/latest)**:

| Asset | Platform |
|---|---|
| `bugorcka-vX.Y.Z_..._win64.zip` | Windows 10/11 x64 (example `start.bat` inside) |
| `bugorcka-vX.Y.Z_..._linux_ubuntu22.tar.gz` | Ubuntu 22.04+ / glibc 2.35+ distros |
| `bugorcka-vX.Y.Z_..._hiveos_ub22.tar.gz` | HiveOS (Ubuntu 22 based images) |

Requirements: a recent NVIDIA driver (for RTX 50xx use the newest available).
No CUDA Toolkit needed on the rig.

## Supported pools

- **LuckyPool** — https://btx.luckypool.io/
  - EU endpoint: **`btx-eu.lproute.com:8666`** — works in both **SSL/TLS** and **plain TCP** mode.
- Your own **stratum-bridge** (default `bridge` dialect), or any other BTX stratum pool.
- **Solo** straight against your coin daemon over JSON-RPC.

## Quick start

Pool (LuckyPool, SSL/TLS):

```
bugorcka -o stratum+ssl://btx-eu.lproute.com:8666 -u btx1qyourwallet -w rig1
```

Same pool over plain TCP:

```
bugorcka -o stratum+tcp://btx-eu.lproute.com:8666 -u btx1qyourwallet -w rig1
```

Pick specific GPUs, half load, log to file:

```
bugorcka -o pool-host:port -u btx1qyourwallet -d 0,1,2 -i 12 -l rig1
```

Benchmark (no pool needed):

```
bugorcka -benchmark -seconds 20
```

Solo against your own node:

```
bugorcka -address btx1qyourwallet -rpcconnect 127.0.0.1
```

## HiveOS setup

Flight Sheet → Miner → **Custom**, then fill in (LuckyPool example):

- **Miner name:** `bugorcka`
- **Installation URL** — the `_hiveos_ub22.tar.gz` asset from
  [Releases](../../releases/latest), e.g.:

  ```
  https://github.com/bugorcka/bugorcka-miner/releases/download/vX.Y.Z/bugorcka-vX.Y.Z_hiveos_ub22.tar.gz
  ```
- **Pool URL:** `btx-eu.lproute.com:8666` (add `stratum+ssl://` for TLS)
- **Wallet and worker template:** `%WAL%.%WORKER_NAME%`
- **Extra config arguments:** `-a btxv4` (append any other flags, single line, e.g. `-a btxv4 -i 18`)

Or import the ready flight sheet as JSON (set your own wallet, worker and the
release **Installation URL**):

```json
{"name":"LP_BTX_bugorcka","isFavorite":true,"items":[{"coin":"BTX","pool_ssl":false,"dpool_ssl":false,"miner":"custom","miner_alt":"bugorcka","miner_config":{"url":"stratum+tcp://btx-eu.lproute.com:8666","miner":"bugorcka","template":"%WAL%.%WORKER_NAME%","install_url":"URL","user_config":"-a btxv4 "},"pool_geo":[]}]}
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
  -u <wallet>               Your BTX payout address (btx1...).
  -w, --worker <name>       Worker name, sent as "WALLET.name" (default: rig1).
      --pool-protocol <d>   Pool dialect: bridge (default), btxpool, stratum.
  -p <pass>                 Pool password (default: x -- BTX pools ignore it).

Solo (JSON-RPC to your own coin daemon):
  -address <addr>           Payout address for the coinbase reward.
  -chain <main|test|regtest> Network to mine on (default: main).
  -rpcconnect <host>        Daemon RPC host (default: 127.0.0.1).
  -rpcport <port>           Daemon RPC port (default: the algorithm's own port).
  -rpcuser / -rpcpassword   RPC credentials (default: read the daemon's .cookie file).
  -rpccookiefile <path>     Path to the daemon's .cookie file.

GPUs and load:
  -d, --devices <csv>       CUDA device indices to mine on, e.g. 0,1,2 (default: all).
  -i, --intensity <1-21>    GPU load: 21 = full speed (default). Lower values idle the
                             card between episodes -- 10 is about half load, 2 about 10%.
  -benchmark                Measure each selected GPU's hashrate and exit.
  -seconds <n>              Benchmark duration in seconds (default: 8).

Output:
  -fl, --fulllog            Plain streaming log instead of the live dashboard/TUI.
  -l, --log <name>          Also append the run log to this file (default: logminer).
  -nc, --nocolor            Disable colored output.
  -api-port <port>          Serve JSON mining stats on http://127.0.0.1:<port>/.

Developer fee:
  -df, --devfee <n>         Developer fee, percent of mining time (default: 2, algorithm-specific).

Other:
  -h, --help                Show help and exit.
  -a, --algo <name>         Mining algorithm (btxv4, the default).
```

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
