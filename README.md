<div align="center">

<img src="assets/banner.png" alt="TAPE Cybersecurity" width="100%">

<br>

<img src="https://readme-typing-svg.demolab.com?font=JetBrains+Mono&size=20&duration=3200&pause=1200&color=8FD9B6&center=true&vCenter=true&width=620&lines=Nmap+from+zero.;Every+command+from+the+video.;Ready+to+copy." alt="Nmap from zero.">

<br><br>

<a href="https://www.youtube.com/watch?v=WZ4BAQYuzg8"><img src="https://img.shields.io/badge/YouTube-WATCH_10:47-0d1117?style=for-the-badge&logo=youtube&logoColor=8FD9B6&labelColor=0d1117"></a>
<img src="https://img.shields.io/badge/LEVEL-BEGINNER-0d1117?style=for-the-badge&labelColor=0d1117&color=1f2a35">
<img src="https://img.shields.io/badge/PRICE-FREE-0d1117?style=for-the-badge&labelColor=0d1117&color=1f2a35">
<a href="https://t.me/TapeXSec"><img src="https://img.shields.io/badge/Telegram-TapeXSec-0d1117?style=for-the-badge&logo=telegram&logoColor=8FD9B6&labelColor=0d1117"></a>

</div>

---

# Nmap Basics: Find Every Open Door

📺 **Video:** [Nmap Lesson 1 | Basics for Beginners](https://www.youtube.com/watch?v=WZ4BAQYuzg8)

> Every attack starts with one question: **what is running on this machine?**
> In this lesson you'll learn to answer it with Nmap.

## Contents
1. [What is a port?](#1-what-is-a-port)
2. [Installing Nmap](#2-installing-nmap)
3. [Your first scan](#3-your-first-scan)
4. [Port states](#4-port-states)
5. [How Nmap knocks: the TCP handshake](#5-how-nmap-knocks-the-tcp-handshake)
6. [Choosing ports](#6-choosing-ports)
7. [Time savers: -T4, -Pn, subnets](#7-time-savers)
8. [Service versions (-sV)](#8-service-versions--sv)
9. [Scripts, OS detection and -A](#9-scripts-os-detection-and--a)
10. [Saving results](#10-saving-results)
11. [📋 Cheat sheet](#-cheat-sheet)

---

## 1. What is a port?

Think of a server as an **office building**:

| Building | Network |
|---|---|
| Street address | **IP address** |
| Numbered doors | **Ports** (65,535 of them) |
| Someone working behind a door | **Service** (a website, SSH, a database…) |

An **open port** means a service is running, and a running service is something we can test.

**Ports worth knowing by heart:**

| Port | Service | What it does |
|---|---|---|
| 21 | FTP | File transfer |
| 22 | SSH | Remote control |
| 80 / 443 | HTTP / HTTPS | Websites (443 = encrypted) |
| 3306 | MySQL | Database |
| 3389 | RDP | Windows Remote Desktop |

---

## 2. Installing Nmap

**Nmap** = **N**etwork **Map**per. Free, open source, around since 1997.

```bash
# Kali Linux: already installed

# Ubuntu / Debian
sudo apt install nmap

# Arch
sudo pacman -S nmap

# Windows: download the installer from https://nmap.org
```

Check that it works:

```bash
nmap --version
```
```
Nmap version 7.993 ( https://nmap.org )
```

---

## 3. Your first scan

```bash
nmap scanme.nmap.org
```
```
Starting Nmap 7.993 ( https://nmap.org )
Nmap scan report for scanme.nmap.org (45.33.32.156)
Host is up (0.20s latency).
Not shown: 995 closed tcp ports (conn-refused)
PORT     STATE    SERVICE
22/tcp   open     ssh
25/tcp   filtered smtp
80/tcp   open     http
646/tcp  filtered ldp
9929/tcp open     nping-echo

Nmap done: 1 IP address (1 host up) scanned in 7.75 seconds
```

**Line by line:**

| Line | Meaning |
|---|---|
| `Starting Nmap 7.993` | Which Nmap version ran the scan |
| `Nmap scan report for …` | Target name and its IP address |
| `Host is up` | The server answered, it's online |
| `(0.20s latency)` | How long the answer took to come back |
| `Not shown: 995 closed` | By default Nmap checks the **top 1,000** ports, not all 65,535. Closed ones are hidden |
| `PORT` | Door number / protocol (TCP) |
| `STATE` | Is the door open? |
| `SERVICE` | ⚠️ **Only a guess**, based on the port number alone. Use `-sV` for the real answer |

---

## 4. Port states

| State | What happened | Meaning |
|---|---|---|
| 🟢 **open** | Nmap knocked, a program answered | Someone is home. This is what we're looking for |
| 🔴 **closed** | The host answered "nobody here" | The machine is alive, but nothing listens on this port |
| 🟡 **filtered** | No answer at all | Usually a **firewall** is in the way |

In our scan, ports **25** (email) and **646** were filtered. Port 25 is very often blocked to fight spam.
Filtered ports are worth remembering: someone decided to hide them.

---

## 5. How Nmap knocks: the TCP handshake

A normal TCP connection is a **3-step handshake**:

```
YOU                         SERVER
 | ---- SYN ---------------> |   "Hi, can we talk?"
 | <--- SYN-ACK ------------ |   "Sure, I'm listening."  → port is OPEN
 | ---- ACK ---------------> |   "Great, let's go."      → connected
```

If the port is closed, the server answers `SYN` with `RST` ("go away").

**SYN scan (`-sS`)**, the default with sudo, never finishes the handshake:

```
 | ---- SYN ---------------> |
 | <--- SYN-ACK ------------ |   → we already know it's open
 | ---- RST ---------------> |   → hang up
```

| | `-sS` SYN scan | `-sT` connect scan |
|---|---|---|
| Needs sudo | ✅ yes | ❌ no |
| Handshake | half (never completed) | full |
| Speed | faster | slower |
| Traces on target | fewer | more |
| Closed ports shown as | `(reset)` | `(conn-refused)` |

> "Stealth" doesn't mean invisible: modern IDS and firewalls detect SYN scans easily.

---

## 6. Choosing ports

```bash
nmap -p 22,80 scanme.nmap.org            # only these ports
nmap -p 1-1000 scanme.nmap.org           # a range
nmap --top-ports 100 scanme.nmap.org     # 100 most common ports
nmap -p- scanme.nmap.org                 # all 65,535 ports (slow)
```

> 💡 Admins love hiding services on unusual high ports. On a real target, always run `-p-` at least once.

---

## 7. Time savers

**Speed: `-T0` … `-T5`**

| Template | Use |
|---|---|
| `-T0`, `-T1` | Very slow, made to sneak past detection |
| `-T3` | Default |
| `-T4` | ✅ The usual choice for labs and practice |
| `-T5` | Insane: fast but may miss results |

**Skip host discovery: `-Pn`**

Before scanning ports, Nmap checks whether the host is alive (usually with a ping).
Many machines, especially Windows, ignore ping, so Nmap says *"Host seems down"* and skips them.
`-Pn` means: *assume it's alive and scan anyway.*

```bash
nmap -Pn <target>
```

**Many targets at once**

```bash
nmap 192.168.1.10 192.168.1.20      # a list
nmap 192.168.1.1-50                 # a range
nmap 192.168.1.0/24                 # a whole network: 256 addresses
```

---

## 8. Service versions (`-sV`)

`-sV` talks to each service and asks: *who are you, and which version?*

```bash
sudo nmap -sV scanme.nmap.org
```
```
PORT     STATE    SERVICE    VERSION
22/tcp   open     ssh        OpenSSH 6.6.1p1 Ubuntu
25/tcp   filtered smtp
80/tcp   open     http       Apache httpd 2.4.7 ((Ubuntu))
646/tcp  filtered ldp
9929/tcp open     nping-echo Nping echo
```

**Why versions matter:** old software often has known security holes, collected in public databases, each with an ID called a **CVE**.

```
version  →  search "<software> <version> vulnerabilities"  →  read the CVEs
```

That's where recon turns into an attack plan, or, on your own server, into a list of things to update.

---

## 9. Scripts, OS detection and `-A`

| Flag | What it does |
|---|---|
| `-sC` | Runs Nmap's default scripts: extra checks on every open port |
| `-O` | Guesses the operating system (needs sudo) |
| `-A` | All-in-one: `-sV` + `-sC` + `-O` + traceroute |

```bash
sudo nmap -A scanme.nmap.org
```
```
22/tcp   open     ssh        OpenSSH 6.6.1p1 Ubuntu
| ssh-hostkey:
|   2048 20:3d:2d:44:62:2a:b0:5a:9d:b5:b3:05:14:c2:a6:b2 (RSA)
|_  256 33:fa:91:0f:e0:e1:7b:1f:6d:05:a2:b0:f1:54:41:56 (ED25519)
80/tcp   open     http       Apache httpd 2.4.7 ((Ubuntu))
|_http-title: Go ahead and ScanMe!

OS guess: Linux 5.0 - 5.14 (98%)
Network Distance: 19 hops
```

> `-A` is powerful but **noisy**. Fine on a lab or practice server; on a real job, think before you use it.

---

## 10. Saving results

```bash
sudo nmap -sV -oA scan_results scanme.nmap.org
ls scan_results.*
```
```
scan_results.gnmap  scan_results.nmap  scan_results.xml
```

| File | For |
|---|---|
| `.nmap` | You to read |
| `.xml` | Other tools |
| `.gnmap` | Quick searching with `grep` |

> In a real pentest you'll scan dozens of machines. **If it's not saved, it didn't happen.**

---

## 📋 Cheat sheet

| Command | What it does |
|---|---|
| `nmap <target>` | Quick scan of the top 1,000 ports |
| `nmap -sV <target>` | Find out what's really running (versions) |
| `nmap -p- <target>` | Scan every single port |
| `nmap -A <target>` | Everything at once: versions, scripts, OS, traceroute |
| `-oA <name>` | Save results in all formats |

**Bonus**

| Command | What it does |
|---|---|
| `nmap -p 22,80 <target>` | Only specific ports |
| `nmap --top-ports 100 <target>` | Top 100 ports |
| `nmap -T4 <target>` | Faster scan (labs) |
| `nmap -Pn <target>` | Skip the ping check |
| `nmap 192.168.1.0/24` | Scan a whole network |

---

<div align="center">

⭐ **Star this repo** if it helped you

<sub>Research · Learn · Practice · Improve · Stay secure</sub>

</div>
