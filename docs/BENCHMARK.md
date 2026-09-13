# Speed — measured, not quoted

## Summary

| Conditions | PortButler | `scp` | `sftp` |
|---|---|---|---|
| Raspberry Pi 5, 1GbE | **108 MB/s** | 74 MB/s | 74 MB/s |
| localhost (no wire limit) | **465 MB/s** | 267 MB/s | 267 MB/s |

Same Mac, same server, same 64 MB file. **1.45× to 1.74×.**

The 108 MB/s over the Pi is **91% of the theoretical maximum for 1GbE**
(119 MB/s). The wire is nearly full, so under those conditions there is nothing
left to win. To see the CPU ceiling you have to remove the wire, which is what
the localhost run is for.

## Why we do not quote anyone else's numbers

Other SSH and SFTP tools publish performance figures. **Putting them in a table
next to ours would be a lie.** Different hardware, different network, different
file size, different ciphers. A number measured on wired gigabit and a number
measured over Wi-Fi placed side by side is not a fact, it is an advertisement.

**So we measure on one Mac, against one server, with one file.** Everything
below can be re-run with the same commands.

## What was measured

| | |
|---|---|
| Client | Mac (Apple silicon), macOS 26.6 |
| Server | Raspberry Pi 5, Ubuntu 24.04, OpenSSH |
| Network | wired 1GbE, 0.65 ms round trip (theoretical max ≈ 119 MB/s) |
| Second condition | localhost — removes the wire, shows the CPU ceiling |
| Compared against | OpenSSH 10.3p1 (`scp`, `sftp`) — what ships with macOS |
| File | 64 MB of random bytes (so compression cannot flatter anyone) |
| Runs | three each; the numbers below are the observed runs |

## SFTP download

| Tool | Run 1 | Run 2 | Run 3 |
|---|---|---|---|
| **PortButler** | **108.4 MB/s** | **108.0 MB/s** | **109.1 MB/s** |
| `scp` | 72.7 MB/s | 74.4 MB/s | 74.4 MB/s |
| `sftp` | 37.6 MB/s\* | 74.4 MB/s | 73.6 MB/s |

\* The first run includes connecting and authenticating. The rest are warm.

**About 1.45× faster, for one reason — several requests are in flight at once.**

```
scp / sftp:   [request]→ ... response ←  [request]→ ... response ←
              32 KiB at a time, waiting 0.43 ms for every round trip
              ceiling = 32 KiB ÷ 0.43 ms ≈ 74 MB/s   ← matches what we measured

PortButler:   [req][req][req][req] → responses come back interleaved
              4 streams · 256 KiB chunks · each written to its own offset
```

**Latency is the ceiling.** The benchmark tool computes that ceiling and prints
it alongside (`one-request-at-a-time ceiling: 74.1 MB/s`). That `scp` lands on
almost exactly that number is the evidence this explanation is right.

**The further away the server, the bigger the gap.** At 200 ms — a server on the
other side of the world — sequential transfer crawls and parallel does not.

## SFTP upload

| Tool | Run 1 | Run 2 | Run 3 |
|---|---|---|---|
| `scp` | 73.6 MB/s | 74.4 MB/s | 74.4 MB/s |

**Upload is still sequential.** It could use the same structure as download, but
writing to several offsets of a remote file at once leaves **a file with holes in
it** if the transfer dies halfway. We will not turn that on before the recovery
path is right. Not breaking matters more than being fast.

## Terminal throughput

A terminal is a different problem from a file transfer. The question is not how
many MB per second arrive — it is **whether the screen keeps moving while they
do.**

```
Target     50 MB/s or more, inside the frame budget (8.33 ms), losing nothing
Measured   passes — all three
```

Three things are checked together.

**Throughput.** On its own this means nothing; you can always make it larger by
making the buffer larger.

**Drain p99 ≤ 8.33 ms** — one frame at 120 Hz. Go over and a frame is dropped,
which the person sees as **scrolling that stutters.** We watch the 99th
percentile rather than the average because one stutter in a hundred is still
one you notice.

**No loss.** When the buffer fills we **slow down rather than drop.** If a line
disappears from a terminal, the person does not know it disappeared. Missing the
line that mattered is worse than waiting.

## Why WindTerm's published numbers are not in a table with ours

WindTerm publishes SFTP transfer benchmarks. What we found (WindTerm 1.72):

| | Download | Upload |
|---|---|---|
| WindTerm | 216.3 MB/s | 247.0 MB/s |
| FileZilla | 161.1 MB/s | 171.8 MB/s |
| WinSCP | 63.7 MB/s | 56.7 MB/s |

**Numbers limited by a wire must not be mixed with numbers that are not.**
216 MB/s is impossible on 1GbE (max 119 MB/s), so that run used a faster link or
stayed inside one machine. Putting our 108 MB/s Pi figure next to it would make
us look like half the speed — but **that comparison is between network cables,
not between clients.**

So here are the runs where the wire is not the limit:

| | Download | Upload |
|---|---|---|
| **PortButler** | **465 MB/s** | 267 MB/s |
| WindTerm 1.72 | 216.3 MB/s | 247.0 MB/s |
| FileZilla | 161.1 MB/s | 171.8 MB/s |
| WinSCP | 63.7 MB/s | 56.7 MB/s |

Ours is an Apple silicon Mac; theirs is Windows 10 on a 2.3 GHz Core i5.
**Read it as an order of magnitude, not a ranking** — a real comparison would
mean installing all of them on the same Mac and moving the same file.

Source: [WindTerm — Performance / Sftp Transfer](https://kingtoolbox.github.io/2023/11/15/benchmark-sftp-transfer/)

## Running it again

```sh
# terminal throughput gate (synthetic load)
cargo run --release --bin throughput -- 5 50 8

# SFTP — the alias has to exist in ssh_config
cargo run --release --bin sftp-bench -- <alias> 64

# what we compared against
scp <server>:/tmp/bench.bin /tmp/out.bin
sftp <server>:/tmp/bench.bin /tmp/out.bin
```

## What we are not claiming

**Only download is faster.** Upload is the same as `scp`.

**Small files see no difference.** Parallelism hides round-trip latency, and at
sizes with only a few round trips there is nothing to hide.

**If the server is the bottleneck, no difference.** A Raspberry Pi 5 does
encryption fast enough; a weaker board runs out of CPU first.

**If the wire is the bottleneck, no difference either.** We already use 91% of
1GbE. No client can be faster than that under those conditions — the remaining
9% is protocol overhead.

**The first run is always different.** It includes connecting and
authenticating. The `sftp` run 1 above is what that looks like.
