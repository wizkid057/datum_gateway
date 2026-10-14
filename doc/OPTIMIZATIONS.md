# Optimizing DATUM Gateway

Start with a working, verified setup from the [installation guide](INSTALL.md). These optimizations help reduce bandwidth usage, deliver new work promptly, and keep more of your hashrate working on the current Bitcoin block.

The defaults are intended to be sane starting points, but they are not always the best settings for every mining farm. Tune them for your hardware, hashrate, network capacity, and preferred share statistics, and verify the results under your site's normal load.

- [Sizing share difficulty](#sizing-share-difficulty)
- [Large farms and multiple Gateways](#large-farms-and-multiple-gateways)
- [Open file limits](#open-file-limits)
- [Local LAN bandwidth and stale work](#local-lan-bandwidth-and-stale-work)
- [How DATUM distributes work](#how-datum-distributes-work)
- [Sites with limited bandwidth](#sites-with-limited-bandwidth)
- [Bitcoin node peer selection](#bitcoin-node-peer-selection)

## Sizing share difficulty

The pool sets a minimum share difficulty. Set `stratum.vardiff_min` **higher as your aggregate hashrate grows** to avoid excessive submissions. Choose an aggregate share rate you are comfortable with for share-sampling variance and the responsiveness of your hashrate statistics. More shares give smoother statistics over shorter periods; fewer shares reduce bandwidth usage on both the ASIC-to-Gateway and Gateway-to-pool connections, but take longer to average out.

**A few dozen shares per second across the entire Gateway is more than sufficient for almost any mining farm.** There is generally no need to exceed **100 shares per second**, and no need to aim that high. These are operating guidelines, not protocol limits. Count all ASICs together when choosing your target. If you want a site-wide budget across multiple Gateways, divide that budget among them.

For hashrate `H` in hashes per second and share difficulty `D`:

```text
Expected shares/second ≈ H / (D × 2^32)
Required difficulty   ≈ H / (target shares/second × 4,294,967,296)
```

Use the total hashrate of all miners on the Gateway, including remote miners. `1 PH/s = 10^15 H/s`; `1 EH/s = 10^18 H/s`. Choose a floor at least as high as the calculated difficulty, the pool's minimum, and the existing configured floor (default `16384`), then **round up to a power of two**. DATUM rounds non-power-of-two `vardiff_min` values **down**, so do the upward rounding yourself.

Examples targeting 30 shares/second across the Gateway, assuming the pool does not require a higher floor. Substitute your preferred rate in the formula above:

| Total hashrate | Calculated minimum difficulty | Set `stratum.vardiff_min` | Expected shares/second at that floor |
| --- | ---: | ---: | ---: |
| 10 PH/s | 77,611 | 131,072 | 17.8 |
| 50 PH/s | 388,052 | 524,288 | 22.2 |
| 100 PH/s | 776,103 | 1,048,576 | 22.2 |
| 1 EH/s | 7,761,022 | 8,388,608 | 27.8 |

For example, a 100 PH/s installation targeting 30 shares/second calculates `10^17 / (30 × 2^32) ≈ 776,102.15`. Round up to the next power of two, `1048576`, and add it to the existing `stratum` section:

```json
{
  "stratum": {
    "vardiff_min": 1048576
  }
}
```

Restart the Gateway after editing its configuration. Shares arrive randomly, so this controls the expected average, not a hard per-second maximum. Rounding up gives a rate below the chosen target: the 100 PH/s example averages about 22.2 shares/second at that floor. Recalculate when adding hashrate, and check the observed rate and statistics over several minutes before deciding whether to adjust it.

Variable difficulty may raise individual miners above this floor, reducing the actual submission rate further. `stratum.vardiff_target_shares_min` defaults to **8 shares per minute per connection**, not per Gateway. At that target, 1,000 connections would submit about 133 shares/second if no difficulty floor constrained them. A suitably sized `vardiff_min` keeps aggregate traffic down even when the per-miner target would otherwise call for more shares. The pool's floor still applies if it is higher than yours.

Higher difficulty produces fewer shares representing more work each; it does not reduce your physical hashrate or chance of finding a block. Individual miners' share counts and hashrate estimates will take longer to settle, especially for slower hardware sharing a Gateway with a large fleet.

At a high difficulty floor, a slow ASIC may take hours to submit its first accepted share. By default, `stratum.idle_timeout_no_shares` disconnects clients that have not submitted any accepted shares within **7,200 seconds (two hours)** of connecting. If long waits are expected, consider increasing this timeout; `0` disables it. Allow for random variation in share arrival times.

## Large farms and multiple Gateways

The default configuration allows **1,024 simultaneous Stratum clients**. Size the connection limits for your ASIC count before connecting a larger farm:

| Setting | Default | Purpose |
| --- | ---: | --- |
| `stratum.max_clients` | 1024 | Total client limit; additional connections are rejected. |
| `stratum.max_threads` | 8 | Maximum number of Stratum server threads. |
| `stratum.max_clients_per_thread` | 128 | Maximum clients handled by each thread. |

`max_clients` must be no greater than `max_threads × max_clients_per_thread`; the Gateway rejects configurations that exceed that capacity. Raising only `max_clients` is therefore insufficient with the default thread settings. The process also needs [an open file limit above the client count](#open-file-limits). The current build limits are 64 threads and 4,096 clients per thread; these are implementation limits, not recommended operating targets.

For example, to allow 2,048 clients with eight threads, merge this into your existing `stratum` section:

```json
{
  "stratum": {
    "max_clients": 2048,
    "max_threads": 8,
    "max_clients_per_thread": 256
  }
}
```

Restart the Gateway after changing these settings. Allow room for reconnects and spare capacity, and choose a thread count appropriate for the machine's CPU and workload. Increasing limits does not add processing power, memory, or network bandwidth. Review the [memory requirements](INSTALL.md#requirements), monitor CPU and RAM usage, and check new-block delivery under the full farm's load. Adjust [share difficulty](#sizing-share-difficulty) for the total hashrate as you add miners.

If you split the farm across several Gateways, assign a distinct `mining.coinbase_unique_id` to each instance using the same coinbase tags. Use values from **1 to 65535**; the default is `4242`. For example, set `1001` on the first Gateway and `1002` on the second. The ID is included in the coinbase transaction to help keep work distinct between instances, including during full solo mining or solo fallback. Do not clone an instance's configuration to another Gateway without updating this ID.

Divide your site-wide share-rate budget among the Gateways, and verify each one's pool connection, reward address, and clients separately. Each instance needs working RPC access and new-block notifications. If using HTTP notifications, arrange for the Bitcoin node to notify every Gateway that depends on it.

### Open file limits

Each connected miner uses a socket, which counts as an open file descriptor. The operating system's **per-process soft open file limit** (`RLIMIT_NOFILE`) must be higher than `stratum.max_clients`, with headroom for listening sockets, pool and RPC connections, dashboard connections, logs, and other descriptors. All Gateway threads share this limit; adding threads does not multiply it. A limit of `1024` is therefore insufficient for `1024` miners.

The Gateway warns at startup when `max_clients` exceeds the soft or hard limit, but does not raise either automatically. Running out of descriptors can produce `accept failed: Too many open files` before reaching the configured client limit. Check both the Gateway connection settings above and the process limits when increasing capacity.

For a systemd service, inspect the configured limits and create an override:

```bash
sudo systemctl show datum-gateway -p LimitNOFILE -p LimitNOFILESoft
sudo systemctl edit datum-gateway
```

Add this to the override, choosing a value above your planned maximum client count:

```ini
[Service]
LimitNOFILE=65535
```

A single `LimitNOFILE` value sets both soft and hard limits. Apply it, then inspect the running process's `Max open files` line on Linux:

```bash
sudo systemctl daemon-reload
sudo systemctl restart datum-gateway
gateway_pid=$(systemctl show datum-gateway -p MainPID --value)
sudo cat "/proc/${gateway_pid}/limits"
```

Use `datum_gateway` instead of `datum-gateway` for the [source-build service example](INSTALL.md#systemd-service-for-source-builds), which already sets `LimitNOFILE=65535`.

For a foreground source build, check the shell's soft and hard limits with `ulimit -Sn` and `ulimit -Hn`. If the hard limit permits it, run `ulimit -Sn 65535` in the same shell before launching the Gateway. Raising the hard limit requires administrator configuration. Shell limits do not change an existing systemd service's limits. Raising file limits also does not raise `stratum.max_clients` or its thread capacity; configure both for the farm's size.

## Local LAN bandwidth and stale work

Keep Bitcoin Core, DATUM Gateway, and your ASICs on the same LAN, with enough available bandwidth to get new work to every miner quickly. Even LAN transmission time matters at a large farm: each client needs its own work message, so the Gateway's network interface and shared switch uplinks must carry the aggregate traffic.

Bitcoin's target block interval is 600 seconds. For small delays, the average proportion of hashing spent on the previous block is approximately:

```text
Stale work fraction ≈ average delay in seconds / 600
Stale work percent  ≈ average delay in seconds / 600 × 100
```

A **100 ms average delay** contributes about `0.1 / 600 × 100 = 0.0167%` stale work, or roughly **0.017%**, before other sources of delay. This is a baseline contribution, not a prediction of the exact rejected-share percentage reported by a pool. For the farm as a whole, weight each miner's delay by its hashrate. If 100 ms is the time to reach the *last* miner, miners updated earlier have smaller delays, so the farm's average contribution will be lower.

Budget approximately **1 KB per client for the urgent new-block work**. Full work can be as much as **roughly 32 KB per client**, depending on the coinbase size selected for the ASIC through fingerprinting, plus message and network overhead. Full work follows the small empty-block job, so its size has less direct impact on the initial stale-work window.

To estimate the time needed to transmit a work update over a shared link:

```text
Transmission time in seconds ≈ clients × bytes per client × 8 / available bits per second
```

For 1,000 clients receiving 1 KB each, about 1 MB must leave the Gateway. Using decimal units, that takes at least **8 ms at 1 Gbit/s**, or **80 ms at 100 Mbit/s**, before overhead, competing traffic, software processing, or miner response time. Ten times as many clients need ten times as much data. These are transmission times for the whole batch, not the average delay experienced by each miner.

Size the Gateway NIC and shared uplinks for these bursts, with spare capacity. Check the actual negotiated link speeds, switch congestion, packet loss, and competing traffic. A fast port on each ASIC does not compensate for a saturated Gateway uplink. Keep bulk transfers away from the path used to deliver mining work, and measure performance during new-block transitions as well as ordinary operation.

**Encryption applies to the DATUM connection between the Gateway and the pool.** Ordinary Stratum v1 between ASICs and the Gateway is unencrypted. If remote ASICs reach your Gateway over the internet, carry that connection through a private VPN. A VPN protects the remote connection but does not remove its latency; running the Gateway, node, and miners together on the site's LAN still provides the greatest benefit.

## How DATUM distributes work

DATUM Gateway automatically uses the **Stratum v1 empty-block speedup**. When switching to a new Bitcoin block, it first sends a small job containing only a coinbase transaction and no ordinary transactions, with `clean_jobs` set so miners discard old work. This gets ASICs hashing on the new block quickly. Full work containing transactions follows as it becomes available; empty work is a brief transition, not a configuration to mine empty blocks permanently.

Routine work updates on the same block use `clean_jobs: false`. The Gateway spreads these nonurgent updates across approximately the work-update period, smoothing bandwidth usage across clients. The initial full-work follow-up after empty work is prioritized; it is not held back for the whole period.

`bitcoind.work_update_seconds` controls the normal update period: the default is **40 seconds**, with a supported range of 5–120 seconds. Increasing it reduces the frequency of routine full-work messages, at the cost of refreshing transaction selection less often. New-block notifications still trigger urgent work immediately; they do not wait for the normal update period. Keep [new-block notifications](INSTALL.md#node-configuration) working correctly to get this benefit.

When editing the configuration file, keep `datum.protocol_global_timeout` at least **5 seconds greater** than `bitcoind.work_update_seconds`; otherwise the Gateway refuses to start. The timeout defaults to 60 seconds, so a 60-second work-update interval requires a timeout of at least 65 seconds.

ASIC fingerprinting is enabled by default through `stratum.fingerprint_miners`. It selects an appropriate coinbase size for recognized hardware, so full-work bandwidth varies with the ASICs and firmware in use. Include the full-work traffic in your capacity planning even though the small new-block job is the most urgent message.

## Sites with limited bandwidth

For mining sites using LTE, Starlink, or another connection with limited or variable bandwidth, prioritize local work delivery and leave capacity for timely Bitcoin block relay:

- **Run the Gateway, Bitcoin node, and ASICs on the same LAN at the mining site.** ASIC work distribution then stays local, while the internet connection carries Bitcoin node traffic and the Gateway's DATUM connection to the pool.
- **Target around one share per second or less for the site's aggregate hashrate.** Use the [difficulty calculation](#sizing-share-difficulty) with a target of `1` or lower, and divide the site's budget among Gateways if needed. This reduces upload bandwidth and congestion; allow more time for share-based statistics to settle.
- **Use a normal work-update period of 40–60 seconds.** The default 40 seconds is suitable; 60 seconds further reduces routine work-update traffic and requires `datum.protocol_global_timeout` of at least 65 seconds. This also reduces local distribution traffic while retaining urgent new-block updates.
- **Set a small, nonzero `maxuploadtarget` in Bitcoin Core's `bitcoin.conf`.** Choose a daily upload budget appropriate for the connection to reduce unnecessary historical-block serving. This is an upload target over 24 hours, not a hard bandwidth cap: recent-block relay and other necessary traffic continue, and it does not limit downloads. A value of `0` means unlimited. See [Bitcoin Core's traffic guidance](https://github.com/bitcoin/bitcoin/blob/master/doc/reduce-traffic.md) for the setting's behavior.
- **Choose several reliable peers with low latency from that site's connection.** Measure from the LTE or satellite connection itself; a peer that performs well from another location may not be the best choice here. See [peer selection](#bitcoin-node-peer-selection).

For example, at **1 PH/s** and a target of **1 share/second**, the calculated difficulty is about `232,831`. Rounding up to `262144` gives roughly **0.89 shares/second**. This configuration excerpt combines that floor with a 60-second normal work-update period and the required 65-second protocol timeout; merge it into your existing Gateway configuration, preserving any longer timeout, and restart the Gateway:

```json
{
  "stratum": {
    "vardiff_min": 262144
  },
  "bitcoind": {
    "work_update_seconds": 60
  },
  "datum": {
    "protocol_global_timeout": 65
  }
}
```

Recalculate the floor for your site's actual hashrate. Preserve the node's transaction relay so it can build useful templates and reconstruct compact blocks efficiently. Monitor the connection under load: upload saturation can delay incoming block announcements and pool acknowledgements as well as outgoing shares. Keeping spare capacity and fast local delivery helps maximize useful work and minimize stale shares.

## Bitcoin node peer selection

Your node's peers affect how quickly new blocks reach you and how quickly blocks you find reach the rest of the Bitcoin network. Better block propagation gets your miners onto current work sooner and helps your own blocks compete successfully.

It usually makes sense to hand-pick several reliable, high-performance peers from publicly available node information websites and add them using `addnode` entries in `bitcoin.conf`. Bitcoin Core will attempt to keep those connections open. Keep automatic peer discovery enabled alongside these selected peers, and prefer a mix of independent operators and network paths.

Evaluate peers from the mining site's actual connection. Look for low latency, stable connections, and prompt block delivery; low ping alone does not guarantee fast block relay. Reassess peers over time, especially after routing or ISP changes. The goal is consistently fast incoming and outgoing blocks, not simply a large connection count.

