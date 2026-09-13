# How it compares

**As of 2026-09-14.** Competitors change. What is written here is what their
public documentation said on that date, and where we could not confirm
something we say so.

## First, where they are better

A table built only from the columns we win is a table nobody believes.

**WindTerm is free and does more.** SSH, SFTP, Telnet, serial and tmux; port
forwarding; Zmodem and Xmodem; split panes, snippets and triggers. Windows,
macOS and Linux. **On feature count we lose.** And it costs nothing.

**Termius syncs.** An encrypted cloud vault keeps your sessions on your Mac,
iPhone and iPad. We do not do that — we move a file.

**Transmit and ForkLift have transfer queues and resume.** We do not. If a large
download dies halfway, ours starts over.

**FileZilla speaks FTP and FTPS.** We only do SFTP.

## What is different

| | PortButler | WindTerm | Termius | Prompt 3 | Transmit 5 |
|---|---|---|---|---|---|
| SSH terminal | ✅ | ✅ | ✅ | ✅ | ❌ |
| SFTP file management | ✅ | ✅ | ✅ | ❌ | ✅ |
| Serial (`/dev/cu.*`) | ✅ | ✅ | ✅ | ❌ | ❌ |
| All three in one window | ✅ | ✅ | ✅ | — | — |
| macOS native | ✅ | ❌ cross-platform | ✅ | ✅ | ✅ |
| Price | **$29 once** | free | ~$10/month | $49.99 once | $45 once |
| App Store sandbox | no (direct) | no | yes | — | no |

Serial is not ours alone — three of these have it. **"SSH and serial in one
place" is not a claim we can make by ourselves**, and saying so would be untrue.

What is unusual is the **combination at this price.** Panic's Prompt is $49.99
for SSH alone and Transmit is $45 for files alone; buying both is $95 and still
has no serial port.

## So why use this one

Three reasons. None of them is a feature count.

### 1. It behaves like a Mac app

WindTerm and FileZilla are built on cross-platform toolkits and look the same on
three operating systems — which means **they look like none of them.** Tabs,
sidebar material, dark mode, system fonts, `⌘K` and `⌘W` muscle memory. A tool
only fits the hand when those line up.

Termius is native but ships through the App Store, **so it lives in a sandbox**
and cannot read `~/.ssh` freely. To use the keys and `known_hosts` you already
have in Terminal, an app has to stay outside the store.

### 2. When it fails, it tells you what to check

As far as we know no other client does this. Most hand you the OS error.

```
Others:      Connection failed
             Connection refused (os error 61)

PortButler:  The machine is up and nothing is listening on that port.
             Check that the SSH server is running and the port is right
             (many devices use 2222 instead of 22).
```

Refused, no answer, name will not resolve, something answered but it is not SSH,
wrong path — each says something different. **In particular, "the TCP connection
worked but this is not SSH" does not send you off doubting the address**, which
is what makes people keep editing a hostname that was correct all along.

### 3. It will not get your account locked

Servers count failed logins. `MaxAuthTries` cuts you off, `pam_faillock` locks
the account, and `fail2ban` **blocks the IP, which throws you out of every
account on that machine.**

If your agent holds six keys, a client will offer them one after another, spend
the server's attempts, and **get cut off before you are ever asked for a
password.** The password you knew all along never gets its turn.

We offer at most four public keys and leave the rest for the password. We skip
methods the server does not accept. We warn before a retry that could lock you
out. And automatic reconnect does not retry an authentication failure.

For anyone running the same login across hundreds of identical devices, this is
less a feature than an accident that does not happen.

## Where it is sold

**Professional Mac apps mostly sell from their own site** — Sketch, Tower,
Transmit, Bartender, TablePlus, CleanShot X — because the App Store sandbox
blocks tools that need real system access. This is the normal shape of things,
not an odd one.

**Mac App Store** — we cannot go. The sandbox blocks writing
`~/.ssh/known_hosts`, and without that there is no host key verification.

**Setapp** — MacPaw's subscription bundle. It does **not** require sandboxing;
Developer ID signing, notarization and a universal binary are enough. It is the
one official channel open to us, and there is no exclusivity, so it can sit
alongside direct sales. The order matters though: stand up on direct sales
first, then apply.

## Speed

Downloads are about 1.45× faster than `scp` and `sftp` because several requests
are in flight at once. **Uploads are the same.** Method, conditions and the
things it does not help with are in [BENCHMARK.md](BENCHMARK.md).

We have not benchmarked the other tools' transfer speeds. Doing it properly
means installing them on the same Mac and moving the same file, and until then
we will not put numbers in a table.

## What we could not confirm

- Whether WindTerm writes `~/.ssh/known_hosts` directly (it appears to keep its
  own store, but we did not verify)
- SFTP transfer speed of any tool other than ours
- Termius pricing by region (we saw roughly $10/month)

## Sources

- [WindTerm (GitHub)](https://github.com/kingToolbox/WindTerm) — features, partly Apache-2.0, free
- [Termius pricing](https://termius.com/pricing) — subscription
- [Prompt 3 (Panic)](https://panic.com/prompt/) — $49.99, SSH only
- [Transmit 5 (Panic)](https://panic.com/transmit/) — $45, file transfer only
- [Core Shell](https://coreshell.app/) — $9.99, SSH only; no SFTP, no serial
