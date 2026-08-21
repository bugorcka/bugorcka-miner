# bugorcka

**High-performance CUDA GPU miner for BTX** (MatMul v4, algo `btx_v2`).
Windows / Linux / HiveOS. Closed source, binary releases only.

> Built from scratch and tuned at SASS level — beats every public BTX miner
> we could measure against, on Turing, Ampere and Blackwell alike.

## Features

- **BTX / MatMul v4** (`btx_v2`) — the current live algorithm (Epoch A, v4 chain)
- NVIDIA GPUs from **Pascal (GTX 10xx) up to Blackwell (RTX 50xx)**, CMP/mining cards included
- Pool mining (TCP / SSL-TLS stratum), **own stratum-bridge dialect**, and **solo mining**
  straight against your coin daemon over JSON-RPC
- Live **TUI dashboard** (hashrate sparkline, per-GPU temps/fans/power, pool status)
  or plain streaming log for files and HiveOS
- **JSON stats API** (`-api-port`) for HiveOS and external monitors
- Per-GPU selection (`-d 0,1,2`), **intensity duty-cycling** (`-i 1..21`) to cap heat/power
- Self-contained binaries — no CUDA Toolkit needed on the rig, only the NVIDIA driver

## Dev fee

**3%** of mining time (default). One cycle is 100 minutes, so the percent equals
minutes per cycle. Every switch to the fee and back is announced in the log.
`-df <n>` can raise it if you want to support development; values ≤ 3 mean 3.

## Downloads

Grab the latest from **[Releases](../../releases/latest)**:

| Asset | Platform |
|---|---|
| `bugorcka-vX.Y.Z_win64.zip` | Windows 10/11 x64 (example `start.bat` inside) |
| `bugorcka-vX.Y.Z_linux_ubuntu22.tar.gz` | Ubuntu 22.04+ / glibc 2.35+ distros |
| `bugorcka-vX.Y.Z_linux_ubuntu20.tar.gz` | Ubuntu 18.04/20.04+, older distros (static OpenSSL) |
| `bugorcka-vX.Y.Z_hiveos_ub20.tar.gz` | HiveOS stock images (recommended) |
| `bugorcka-vX.Y.Z_hiveos_ub22.tar.gz` | HiveOS newer Ubuntu 22 based images |

Requirements: NVIDIA driver new enough for CUDA 12 runtime (R525+; R570+ recommended,
required for RTX 50xx).

## Quick start

Pool:

```
bugorcka -o stratum+ssl://btx.ninjaraider.com:44921 -u btx1qyourwallet -w rig1
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

Flight Sheet → Miner → **Custom**, then:

- **Miner name:** `bugorcka`
- **Installation URL:** link to the `bugorcka-vX.Y.Z_hiveos_ub20.tar.gz` asset from Releases
- **Wallet and worker template:** `%WAL%.%WORKER_NAME%`
- **Pool URL:** your pool / bridge `host:port`
- **Extra config arguments:** any CLI flags, single line (e.g. `-i 18`)

Hashrate and per-GPU stats show up natively in the Hive dashboard
(the package wires the JSON stats API to the Hive agent).

## Command line

```
bugorcka -h

Pool:
  -o <url>                  Pool address, e.g. stratum+ssl://btx.ninjaraider.com:44921
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
  -df, --devfee <n>         Developer fee, percent of mining time (default: 3).

Other:
  -h, --help                Show help and exit.
  -a, --algo <name>         Mining algorithm (btx_v2, the default).
```

## Notes

- Antivirus software commonly flags **any** GPU miner as a PUA — the binaries here
  are exactly what the Releases page publishes, nothing else. Add an exclusion if needed.
- Multi-GPU rigs: one instance drives all cards; separate nonce ranges per card,
  no duplicate shares.
- Questions, bug reports, feature requests → [Issues](../../issues).
