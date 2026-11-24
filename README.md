# HEAVYLOCK — Networking Labs

> **Build locks. Learn how they break!**

Hands-on Linux laboratories that go with the HEAVYLOCK networking articles on Dev.to.

Each lab follows the HEAVYLOCK method:

1. **Understand** the mechanism.
2. **Experiment** with it on a real system.
3. **Break** it on purpose in a controlled environment.
4. **Troubleshoot** it and restore normal operation.

## Before You Start

Read the [HEAVYLOCK profile README](https://github.com/heavy-lock/heavy-lock) first. It presents the topics covered and the kind of resources published in this space.

Do each lab after reading the matching article, because the lab assumes you understand the theory.

## Language Support

Every laboratory is available in two languages:

- 🇬🇧 **English:** the original version.
- 🇫🇷 **French:** a French translation.

Language links are provided directly inside each laboratory README, allowing you to switch between versions.

## Labs

| Article | Topic | Lab |
|---|---|---|
| [Understanding the TCP/IP Model](PASTE_ARTICLE_URL_HERE) | TCP/IP model, encapsulation, IP addresses, ports, TCP and UDP | [TCP/IP Lab](tcpip/README.md) |

**How to read this table:**

- **Article:** the Dev.to article that explains the theory. Click it to read the article.
- **Topic:** the main concepts covered by the article and the lab.
- **Lab:** the practical exercises. Each lab includes a French version.

New articles and labs are added to this table as they are published.

## Requirements

- A Linux machine, virtual machine or VPS (Debian or Ubuntu unless the lab says otherwise). You don't need Linux on your personal computer: a VPS or a virtual machine is enough.
- A terminal and basic command-line knowledge.
- A test environment. **Never break things on a production server.** Use a machine you can safely reset.

## Repository Structure

```text
networking-labs/
├── README.md
├── tcpip/
│   ├── README.md
│   └── README.fr.md
├── ipv4/
│   ├── README.md
│   └── README.fr.md
├── routing/
│   ├── README.md
│   └── README.fr.md
├── dns/
│   ├── README.md
│   └── README.fr.md
├── dhcp/
│   ├── README.md
│   └── README.fr.md
└── http-https-tls/
    ├── README.md
    └── README.fr.md
```

Each directory contains its corresponding laboratory in English and French.

---

> **HEAVYLOCK**
>
> **Build locks. Learn how they break!**
