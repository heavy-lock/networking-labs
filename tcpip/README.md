# HEAVYLOCK — Networking Lab 01: Understanding the TCP/IP Model

🇫🇷 [Lire la version française](README.fr.md)

> [!NOTE]
> **Level:** beginner · **Estimated time:** 1.5 to 2 hours · **Environment:** Linux (Debian/Ubuntu, or RHEL/Rocky/AlmaLinux using `dnf`)

## 🎯 Lab objective

This lab teaches you how machines communicate over a network using the TCP/IP model.

You will:

- explore the four layers of the model;
- observe the network information of your own machine;
- analyze real TCP and UDP communications;
- diagnose two simple network failures.

> [!IMPORTANT]
> **The goal is not to read everything line by line.** Network commands print a lot of information, and most of it is not useful to you yet. In every exercise, a **👁️ Look only at** line tells you the 2 or 3 elements that matter. You can ignore the rest for now: you will come back to it when you need it.

> [!NOTE]
> The results shown in this lab are **examples**. Your IP addresses, interfaces, ports, and some responses will be different. That is not an error: the goal is to understand the idea, not to reproduce exactly the same results.

> [!IMPORTANT]
> **You do not need to memorize the commands.** You need to understand what they are for, know how to read their documentation (`man`), and spot the essentials in their output.

---

## ⭐ The essentials of this lab

If you only remember five ideas:

1. The TCP/IP model has **four layers**: Application, Transport, Internet, Network Access.
2. An **IP address** identifies a machine on a network; a **MAC address** identifies an interface on the local network.
3. The **routing table** decides where packets go.
4. **TCP** opens a connection before sending data (the **three-way handshake**); **UDP** sends directly, with no connection.
5. When a communication fails, the first question is: **at which layer is the problem?**

---

## 🧭 Lab overview

| Part | Topic | What you do |
|---|---|---|
| **Level 1** | The four TCP/IP layers | You observe your machine, layer by layer. |
| **Level 2** | A real TCP communication | You create a local HTTP server and capture the handshake. |
| **Level 3** | A UDP communication | You send a datagram and compare it with TCP. |
| **Level 4** | Diagnosing a failure | You trigger two deliberate failures and interpret them. |

## 🧰 Before you start

**What you need:**

- a Linux machine (physical, virtual, or VPS) with `sudo` access;
- a terminal (for some exercises, you will open three at the same time);
- the `python3`, `curl`, `tcpdump`, and `nc` (netcat) packages.

**Installation if a tool is missing:**

| System | Command |
|---|---|
| Debian / Ubuntu | `sudo apt install python3 curl tcpdump netcat-openbsd` |
| RHEL / Rocky / AlmaLinux | `sudo dnf install python3 curl tcpdump nmap-ncat` |

> [!WARNING]
> On RHEL, the `nc` command is provided by the `nmap-ncat` package. Its syntax is close to `netcat-openbsd` but not always identical. If an option does not work, check `man nc`.

**Legend:**

| Symbol | Meaning |
|---|---|
| 🧪 | The command to run |
| 👀 | The result you may get |
| 👁️ | What to look at in the result (the rest can be ignored) |
| 🧠 | How to understand this result |
| ✅ | The key idea to remember |
| 🔎 | Going further (optional) |

---

# 🧱 Level 1 — Exploring the four TCP/IP layers

| Order | Layer | Main role |
|---|---|---|
| 4 | **Application** | Provides network services to applications. |
| 3 | **Transport** | Handles communication between applications using TCP or UDP. |
| 2 | **Internet** | Handles IP addressing and packet routing. |
| 1 | **Network Access** | Allows data to be transmitted on the local network. |

When a machine **sends** data, it goes down the layers, from Application to Network Access. When it **receives** data, it goes back up in the opposite order.

---

## 1️⃣ Application layer

🎯 **Objective:** observe how a domain name is resolved and how a web server answers an HTTP request.

### Exercise 1 — Resolve a domain name

🧪 **Command:**

```bash
getent hosts example.com
```

👀 **Possible result:**

```text
93.184.216.34 example.com
```

You may also get an IPv6 address (for example `2606:4700:10::6814:179a example.com`), several addresses, or **no result** if the resolution fails.

👁️ **Look only at:** the IP address, then the name on the right.

🧠 **How to read the result**

- An IPv4 address has four numbers separated by dots.
- An IPv6 address is written in hexadecimal, separated by colons.
- A domain name can have `A` records (IPv4) and `AAAA` records (IPv6). Depending on your IPv6 connectivity, your DNS resolver, and your configuration, you may get one or the other.

> [!TIP]
> `getent hosts` may return only one address family. Also try `getent ahosts example.com` to compare. Check the exact behavior with `man getent`.

✅ **Remember:** a domain name can be associated with one or more IP addresses, in IPv4 and/or IPv6.

### Exercise 2 — Send an HTTP request

🧪 **Command:**

```bash
curl -I https://example.com
```

The `-I` option requests only the response headers.

👀 **Possible result:**

```text
HTTP/2 200
content-type: text/html; charset=utf-8
server: cloudflare
```

👁️ **Look only at:** the first line, and above all the code (`200`).

🧠 **How to read the result**

| Code or message | Meaning |
|---|---|
| `200` | The request succeeded. |
| `301` or `302` | The server indicates a redirect. |
| `403` | Access is denied. |
| `404` | The requested resource was not found. |
| Connection error | The server could not be reached. |

✅ **Remember:** HTTP lets client applications and web servers exchange requests and responses. The response code tells you how it went.

> [!TIP]
> 🔎 *Optional:* `man getent`, `man curl`. Specifications: RFC 1034 and 1035 (DNS), RFC 9110 (HTTP).

---

## 2️⃣ Transport layer

🎯 **Objective:** observe the TCP and UDP sockets present on your machine.

### Exercise 3 — Observe TCP sockets

🧪 **Command:**

```bash
ss -lnt
```

| Option | Role |
|---|---|
| `-l` | Shows listening sockets. |
| `-n` | Shows addresses and ports in numeric form. |
| `-t` | Shows TCP sockets. |

👀 **Possible result:**

```text
State   Recv-Q  Send-Q  Local Address:Port
LISTEN  0       4096    0.0.0.0:22
LISTEN  0       511     0.0.0.0:80
```

👁️ **Look only at:** `LISTEN` and the `Local Address:Port` column.

🧠 **How to read the result**

- `LISTEN`: a service is waiting for TCP connections.
- `0.0.0.0:22`: a service is listening on port 22, on all IPv4 interfaces.
- `0.0.0.0:80`: a service is listening on port 80, on all IPv4 interfaces.

You may see other ports or fewer lines: it depends on the services running on your machine. The list can even be empty.

✅ **Remember:** a service waiting for TCP connections appears in the `LISTEN` state.

### Exercise 4 — Observe UDP sockets

🧪 **Command:**

```bash
ss -lnu
```

👀 **Possible result:**

```text
State   Recv-Q  Send-Q  Local Address:Port
UNCONN  0       0       127.0.0.53:53
```

👁️ **Look only at:** the state (`UNCONN`) and the port.

🧠 **How to read the result**

- `UNCONN`: a UDP socket with no established connection.
- `127.0.0.53:53`: here, a local DNS service on port 53.
- **Why `UNCONN` and not `LISTEN`?** UDP does not establish a connection before sending data. It is therefore normal to see `UNCONN`.

✅ **Remember:** TCP and UDP are both transport protocols, but they do not handle communication the same way.

> [!TIP]
> 🔎 *Optional:* `man ss`. Specifications: RFC 9293 (TCP), RFC 768 (UDP).

---

## 3️⃣ Internet layer

🎯 **Objective:** identify your machine's IP addresses and understand how it chooses a route.

### Exercise 5 — Observe IP addresses

🧪 **Command:**

```bash
ip addr
```

👀 **Possible result:**

```text
1: lo: <LOOPBACK,UP,LOWER_UP>
    inet 127.0.0.1/8

2: enp3s0: <BROADCAST,MULTICAST,UP>
    link/ether 00:11:22:33:44:55

3: wlp2s0: <BROADCAST,MULTICAST,UP,LOWER_UP>
    inet 192.168.1.20/24
    inet6 fe80::1234:5678:abcd:1234/64
```

The names and addresses are examples.

👁️ **Look only at:** the name of each interface, and the `inet` (IPv4) and `inet6` (IPv6) lines.

🧠 **How to read the result**

| Element | Meaning |
|---|---|
| `lo` | Loopback interface: the machine communicates with itself. |
| `enp3s0` | Example of an Ethernet interface. |
| `wlp2s0` | Example of a Wi-Fi interface. |
| `inet` / `inet6` | An IPv4 address / an IPv6 address. |
| `link/ether` | The MAC address of the interface. |

Three states worth knowing:

| State | Meaning |
|---|---|
| `UP` | The interface is enabled. |
| `LOWER_UP` | The network link is operational. |
| `NO-CARRIER` | No link detected (for example, no Ethernet cable plugged in). |

✅ **Remember:** `ip addr` tells you which IP addresses are assigned to the interfaces.

### Exercise 6 — Observe network interfaces

🧪 **Command:**

```bash
ip link
```

👀 **Possible result:**

```text
2: enp3s0: <BROADCAST,MULTICAST,UP>
    link/ether 00:11:22:33:44:55

3: wlp2s0: <BROADCAST,MULTICAST,UP,LOWER_UP>
    link/ether aa:bb:cc:dd:ee:ff
```

👁️ **Look only at:** the interface name, its state (`UP`), and the `link/ether` line.

🧠 **How to read the result**

| Command | What it shows |
|---|---|
| `ip addr` | Interfaces and their **IP addresses**. |
| `ip link` | Interfaces, their **state**, and their **MAC addresses**. |

✅ **Remember:** an IP address is used for addressing at the Internet layer; a MAC address identifies an interface at the Network Access layer.

### Exercise 7 — Understand the routing table

🧪 **Command:**

```bash
ip route
```

👀 **Possible result:**

```text
default via 192.168.1.1 dev wlp2s0
192.168.1.0/24 dev wlp2s0
```

👁️ **Look only at:** the line that starts with `default`.

🧠 **How to read the result**

| Element | Meaning |
|---|---|
| `default` | Route used when no more specific route matches. |
| `via 192.168.1.1` | The gateway (often your router). |
| `dev wlp2s0` | The interface used to send the packets. |

The second line means the `192.168.1.0/24` network is directly reachable through `wlp2s0`.

On your machine the addresses will be different (for example `default via 10.0.0.1 dev enp3s0`). The default route may also be missing if no gateway is configured.

✅ **Remember:** the routing table helps the system choose which interface and which gateway to use to send a packet.

> [!TIP]
> 🔎 *Optional:* `man ip`, `man ip-route`. Specification: RFC 791 (IPv4).

---

## 4️⃣ Network Access layer

This layer deals with data transmission on the local network, Ethernet frames, and MAC addresses.

### Exercise 8 — Observe neighbors on the local network

🧪 **Command:**

```bash
ip neigh
```

👀 **Possible result:**

```text
192.168.1.1 dev wlp2s0 lladdr aa:bb:cc:dd:ee:ff REACHABLE
```

👁️ **Look only at:** the IP address, the address after `lladdr` (the MAC), and the state at the end.

🧠 **How to read the result**

- Your machine knows this neighbor's MAC address and considers it reachable (`REACHABLE`).
- You may also see `STALE`, `DELAY`, or `INCOMPLETE`. The table can be empty if no neighbor has been discovered yet.

✅ **Remember:** on a local IPv4 network, ARP maps an IP address to a MAC address. IPv6 uses NDP for equivalent functions.

> [!TIP]
> 🔎 *Optional:* `man ip-neighbour`. Specifications: RFC 826 (ARP), RFC 4861 (NDP).

---

# 🔬 Level 2 — Observing a real TCP communication

🎯 **Objective:** create a local HTTP server, establish a TCP connection, and spot the handshake in the capture.

We use `127.0.0.1`: the communication stays on your machine.

> [!NOTE]
> Open **three terminals**: one for the server, one for the client, one for the capture.

## Step 1 — Prepare the server

```bash
mkdir -p /tmp/tcpip-lab
echo "Hello TCP/IP" > /tmp/tcpip-lab/index.html
```

No output is expected if everything succeeds.

## Step 2 — Start the HTTP server

**Terminal 1:**

```bash
python3 -m http.server 8080 --bind 127.0.0.1 --directory /tmp/tcpip-lab
```

👀 **Expected result:**

```text
Serving HTTP on 127.0.0.1 port 8080
```

Keep this terminal open.

## Step 3 — Check the TCP port

**Terminal 2:**

```bash
ss -lnt
```

👀 **Expected result:** a line with port `8080`, for example:

```text
LISTEN  0  5  127.0.0.1:8080  0.0.0.0:*
```

🧠 The server is waiting for TCP connections on port 8080, only on the loopback address.

## Step 4 — Start the network capture

**Terminal 3:**

```bash
sudo tcpdump -i lo -nn -A 'tcp port 8080'
```

| Element | Role |
|---|---|
| `-i lo` | Captures on the loopback interface. |
| `-nn` | Keeps addresses and ports in numeric form. |
| `-A` | Displays captured data as readable text. |
| `tcp port 8080` | Keeps only TCP packets on port 8080. |

Leave the capture running.

## Step 5 — Send an HTTP request

In **terminal 2**:

```bash
curl http://127.0.0.1:8080
```

👀 **Expected result:**

```text
Hello TCP/IP
```

In terminal 1, the server displays a line similar to `127.0.0.1 - - "GET / HTTP/1.1" 200 -`. The request reached the server, which answered with code `200`.

## Step 6 — Read the capture (without drowning)

In **terminal 3**, stop the capture with `Ctrl + C`.

The real `tcpdump` output is long: timestamps, sequence numbers, window sizes… **You can ignore all of that.** Here is the same capture, reduced to what matters (the `...` replace details you do not need to read):

👀 **Example capture (reduced to the essentials):**

```text
IP 127.0.0.1.52114 > 127.0.0.1.8080: Flags [S],  ...
IP 127.0.0.1.8080 > 127.0.0.1.52114: Flags [S.], ...
IP 127.0.0.1.52114 > 127.0.0.1.8080: Flags [.],  ...
IP 127.0.0.1.52114 > 127.0.0.1.8080: Flags [P.], ...
GET / HTTP/1.1
IP 127.0.0.1.8080 > 127.0.0.1.52114: Flags [P.], ...
HTTP/1.0 200 OK
IP 127.0.0.1.8080 > 127.0.0.1.52114: Flags [F.], ...
IP 127.0.0.1.52114 > 127.0.0.1.8080: Flags [F.], ...
```

👁️ **Look only at three things:**

1. **`Flags [...]`**: what each packet does.
2. **The `>` direction**: who is talking to whom (client to server, or the reverse). Port `52114` is a temporary port chosen by `curl`; yours will be different.
3. **The text lines** `GET ...` and `HTTP/1.0 200 OK`: this is the HTTP data.

You do **not** need to understand `seq`, `ack`, `win`, or `length` for this lab.

🧠 **The flags to know:**

| Flag | Meaning |
|---|---|
| `[S]` | The client requests a connection (SYN). |
| `[S.]` | The server accepts (SYN-ACK). |
| `[.]` | Simple acknowledgment, no data (ACK). |
| `[P.]` | One side sends data (PUSH + ACK): this is where `GET` and the response travel. |
| `[F.]` | One side closes the connection (FIN). |

The first three packets form the **three-way handshake**: SYN, then SYN-ACK, then ACK. Only then does the client send `GET / HTTP/1.1` and the server answer. Finally, the connection is closed with `FIN` packets.

> [!NOTE]
> Details vary from one machine to another. The `python3 -m http.server` server usually answers with `HTTP/1.0`, while `curl` sends its request as `HTTP/1.1`. This is normal.

✅ **Remember:** TCP establishes a connection (handshake) before transmitting application data.

> [!TIP]
> 🔎 *Optional:* `man tcpdump`, `man pcap-filter`. Specification: RFC 9293.

---

# 📨 Level 3 — Observing a UDP communication

🎯 **Objective:** send a UDP message and compare it with TCP.

## Step 1 — Prepare a UDP listener

**Terminal 1:**

```bash
nc -u -l 9999
```

This command opens a UDP listener on port `9999`. Keep the terminal open.

## Step 2 — Capture the UDP packets

**Terminal 2:**

```bash
sudo tcpdump -i lo -nn 'udp port 9999'
```

## Step 3 — Send a message

**Terminal 3:**

```bash
echo "Hello UDP" | nc -u -w 1 127.0.0.1 9999
```

👀 **Expected result in terminal 1:**

```text
Hello UDP
```

👀 **Expected result in terminal 2 (example):**

```text
IP 127.0.0.1.41234 > 127.0.0.1.9999: UDP, length 10
```

👁️ **Look only at:** the word `UDP`, and what is **missing**: no `Flags`, no `[S]` or `[S.]`. A single packet is enough.

Stop the capture with `Ctrl + C`, then close the listener with `Ctrl + C`.

🧠 **To understand:** unlike TCP, UDP performs no three-way handshake. The message goes out directly as a datagram.

### ⚖️ TCP and UDP at a glance

| Criterion | TCP | UDP |
|---|---|---|
| Prior connection | Yes (three-way handshake) | No |
| State in `ss` | `LISTEN` | `UNCONN` |
| In the capture | `[S]`, `[S.]`, `[.]`, `[P.]`, `[F.]` | A single `UDP` packet |

✅ **Remember:** UDP lets you send data without first establishing a connection.

> [!TIP]
> 🔎 *Optional:* `man nc`. Look into which real-world uses fit TCP (web, SSH, mail) and UDP (DNS, streaming, online games).

---

# 🛠️ Level 4 — Diagnosing a communication that fails

🎯 **Objective:** understand how a failure blocks a communication, and learn to tell **where** it happens.

We reuse the local HTTP server from level 2.

## Failure 1 — The server is no longer available

Stop the Python server with `Ctrl + C`, then run:

```bash
curl http://127.0.0.1:8080
```

👀 **Possible result:**

```text
curl: (7) Failed to connect to 127.0.0.1 port 8080
```

The exact message varies depending on your `curl` version.

🧠 The machine still has its loopback address, but **no server is listening** on port 8080. The TCP connection cannot be established.

## Failure 2 — The server answers, but the resource does not exist

Restart the server:

```bash
python3 -m http.server 8080 --bind 127.0.0.1 --directory /tmp/tcpip-lab
```

In another terminal:

```bash
curl -i http://127.0.0.1:8080/page-inexistante
```

👀 **Expected result:** an HTTP response with a `404` code.

🧠 The TCP connection succeeded and the server received the request, but the resource does not exist: it returns `404 Not Found`.

### 🩺 Where is the failure?

| Symptom | Layer involved | What it means |
|---|---|---|
| `curl: (7) Failed to connect` | Transport (TCP) | Nobody is listening on this port: the connection fails. |
| `404 Not Found` | Application (HTTP) | The connection works, but the resource does not exist. |

✅ **Remember:** a successful network connection does not mean the resource exists. To troubleshoot, first find the layer where it breaks.

> [!TIP]
> 🔎 *Optional:* redo failure 1 with `tcpdump` running and observe how your machine answers when nobody is listening. Look up what a `RST` packet is.

---

# 🏁 Lab summary

| Layer | What you learned to observe | Tools used |
|---|---|---|
| **Application** | Name resolution and HTTP responses. | `getent`, `curl` |
| **Transport** | TCP/UDP sockets, ports, and the TCP handshake. | `ss`, `tcpdump`, `nc` |
| **Internet** | IP addresses and the routing table. | `ip addr`, `ip route` |
| **Network Access** | Interfaces, MAC addresses, and neighbors. | `ip link`, `ip neigh` |

---

# 🚀 Go further: do not limit yourself to this lab

> [!IMPORTANT]
> This lab is a **starting point**, not a destination. It gives you the essentials to observe the network, but every command and every protocol hides far more detail. You do not have to understand everything today: go step by step.

When something feels unclear, **look elsewhere**. This is what system administrators do every day.

1. **Read the manual.** Every command has a `man` page (`man ss`, `man ip`, `man tcpdump`). Also try `command --help`.
2. **Change the options.** Run a command differently and observe what changes.
3. **Go to the official sources.** RFCs are the reference texts: RFC 9293 (TCP), RFC 768 (UDP), RFC 791 (IP), RFC 826 (ARP), RFC 4861 (NDP), RFC 1034 and 1035 (DNS), RFC 9110 (HTTP).
4. **Cross-check sources.** A different explanation can unlock your understanding.
5. **Break it and fix it.** Change a port, stop a service, then explain why the result changed.
6. **Ask yourself "why?"** Do not only remember what a command does: look for why the protocol works the way it does.

**Ideas to explore next:**

- Why does TCP use a three-step handshake rather than two?
- What happens when a TCP packet is lost?
- How does DNS work in detail?
- How do NAT and private addresses work?
- What does a capture of an HTTPS request show, and why is the content unreadable?

> [!NOTE]
> Found a mistake or a command that does not work on your machine? Open an *Issue* on this repository, mentioning your distribution and the command involved.

---

**HEAVYLOCK — Build locks. Learn how they break!**
