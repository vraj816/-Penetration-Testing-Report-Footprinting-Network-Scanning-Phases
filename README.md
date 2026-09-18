# Footprinting, Reconnaissance & Network Scanning

**W2-PM1 & W2-PM5 | Cybersecurity | Networkwalks**

---

##  Project Information

| Detail              | Information                                                                                              |
| ------------------- | -------------------------------------------------------------------------------------------------------- |
| **Pentester**       | Sandra Chkumbi                                                                                           |
| **Program / Batch** | B083 – Networkwalks                                                                                      |
| **Date**            | 18–19 September 2026                                                                                     |
| **Modules**         | W2-PM1 – Multiple Kali Linux Tools (Footprinting & Reconnaissance)<br>W2-PM5 – Zenmap (Network Scanning) |
| **Client / Target** | Networkwalks                                                                                             |
| **Permission**      | Written permission secured                                                                               |
| **Phases Covered**  | Phase 1: Reconnaissance & Footprinting<br>Phase 2: Network Scanning                                      |

---

##  Liability Disclaimer

These activities were performed only on systems and devices where written permission had been secured.

This repository is intended for **educational and research purposes only**. Do not use the material to access, scan, or test systems without proper authorization.

Unauthorized access to computer systems may constitute a criminal offence and can result in serious legal and professional consequences.

---

##  Introduction

This report documents two practical modules completed during **Week 2 of the Cybersecurity Programme at Networkwalks (Cohort B083)**:

* **W2-PM1:** Footprinting & Reconnaissance using multiple Kali Linux tools
* **W2-PM5:** Network Scanning using Zenmap

### Footprinting & Reconnaissance

Footprinting is an early phase of a penetration test that involves gathering information about a target's infrastructure, technologies, DNS configuration, and publicly exposed information.

### Network Scanning

Network scanning builds on reconnaissance by probing an **authorized network** to identify live hosts and understand the reachable network environment.

For **W2-PM1**, six Kali Linux command-line tools were used against the `networkwalks.com` domain.

For **W2-PM5**, **Zenmap**, the graphical front-end for Nmap, was used to perform a ping scan across a `/24` subnet in a controlled VirtualBox environment.

---

#  Tools Used

| Tool                        | Purpose                                                                              |
| --------------------------- | ------------------------------------------------------------------------------------ |
| **Kali Linux (VirtualBox)** | Operating system used for reconnaissance and scanning                                |
| **WHOIS**                   | Retrieves domain registration information such as registrar, dates, and name servers |
| **WhatWeb**                 | Fingerprints web technologies, CMS, plugins, frameworks, and server information      |
| **Nslookup**                | Resolves domain names to IP addresses using DNS                                      |
| **curl**                    | Inspects HTTP response headers, cookies, caching information, and exposed endpoints  |
| **wafw00f**                 | Detects Web Application Firewalls protecting a website                               |
| **dnsrecon**                | Enumerates DNS records such as NS, SOA, MX, TXT/SPF, and SRV records                 |
| **Zenmap / Nmap**           | Performs network host discovery and visualizes network topology                      |

---

#  W2-PM1: Footprinting & Reconnaissance

Passive reconnaissance was performed against:

```text
networkwalks.com
```

The following Kali Linux tools were used:

1. WHOIS
2. WhatWeb
3. Nslookup
4. curl
5. wafw00f
6. dnsrecon

---

## 1. WHOIS

### Command

```bash
whois networkwalks.com
```

### Key Observations

* Domain registered: **6 November 2019**
* Expiry date: **6 November 2027**
* Registrar: **GoDaddy.com, LLC**
* Name servers:

  * `NS6135.HOSTGATOR.COM`
  * `NS6136.HOSTGATOR.COM`
* Registrant identity protected by **Domains By Proxy, LLC**
* DNSSEC was reported as **not enabled**
* Several registrar lock/status flags were present

### Security Relevance

WHOIS information can provide an initial picture of domain ownership, registration information, and DNS/hosting arrangements.

DNSSEC status is also a relevant security observation during reconnaissance.

### Evidence

<img width="1918" height="918" alt="whois" src="https://github.com/user-attachments/assets/e0711efc-6709-45d4-b54f-27d5fca4b9fe" />

---

# 2. WhatWeb

### Command

```bash
whatweb networkwalks.com
```

### Key Observations

WhatWeb identified:

* Apache web server
* WordPress 7.1
* WP Download Manager 3.3.58
* Bootstrap 7.1
* jQuery 3.7.1
* Google Tag Manager
* HTML5
* Open Graph Protocol
* HTTP 301 redirect from HTTP to HTTPS
* Server IP: `192.232.216.135`
* Contact email exposed in page metadata

### Security Relevance

Publicly visible technology and version information can help an authorized tester identify software that should be checked against known security advisories.

### Evidence

<img width="1918" height="918" alt="whatsweb" src="https://github.com/user-attachments/assets/af8906d8-4dfd-4354-ad27-06462328b6d9" />

---

# 3. Nslookup

### Command

```bash
nslookup networkwalks.com
```

### Key Observations

The domain resolved to:

```text
192.232.216.135
```

The lookup used Google's public DNS resolver:

```text
8.8.8.8
```

The response was **non-authoritative** and confirmed the IP address identified during the WhatWeb scan.

### Security Relevance

DNS resolution provides the IP address associated with a domain and can help establish the target's basic network footprint.

### Evidence
<img width="1918" height="918" alt="nslookup" src="https://github.com/user-attachments/assets/cabf74c6-e809-47d6-902b-f7feb1a95313" />



---

# 4. HTTP Header Inspection with curl

### Command

```bash
curl -I https://networkwalks.com
```

### Key Observations

The response included:

* `HTTP/2 200 OK`
* Apache server information
* WordPress REST API links:

  * `/wp-json/`
  * `/wp-json/wp/v2/pages/53`
* `__wpdm_client` session cookie with `Secure` and `HttpOnly` flags
* `x-nginx-cache` header
* `Permissions-Policy` references to third-party services including Google, Cloudflare, reCAPTCHA, and hCaptcha

### Security Relevance

HTTP headers can expose useful technical information about the web application, server, caching layer, and security configuration.

The presence of WordPress REST API endpoints can also provide additional information for authorized enumeration, depending on the site's configuration.

### Evidence

<img width="1918" height="918" alt="curl" src="https://github.com/user-attachments/assets/3766a256-2ed3-4db6-ad99-f204328fea63" />


---

# 5. WAF Detection with wafw00f

### Command

```bash
wafw00f networkwalks.com
```

### Key Observation

The tool identified:

**ModSecurity (SpiderLabs)**

as the Web Application Firewall protecting the website.

### Security Relevance

Identifying a WAF is useful during an authorized security assessment because it helps the tester understand which defensive controls are present in front of the web application.

### Evidence

<img width="1918" height="918" alt="wafw00f" src="https://github.com/user-attachments/assets/c661a832-386e-42c9-a66f-489d58329c0a" />

---

# 6. DNS Enumeration with dnsrecon

### Command

```bash
dnsrecon -d networkwalks.com
```

### Key Observations

The enumeration identified:

* SOA record
* HostGator name servers
* A record: `192.232.216.135`
* MX record: `mail.networkwalks.com`
* TXT/SPF records
* Google Site Verification record
* SRV records associated with cPanel email discovery
* BIND version information

The mail host resolved to the same IP address as the web server.

### Security Relevance

DNS records can reveal information about hosting architecture, mail infrastructure, DNS configuration, and other infrastructure details useful during authorized reconnaissance.

### Evidence

<img width="1918" height="918" alt="dnsRecon" src="https://github.com/user-attachments/assets/b60ee236-048f-4506-846c-bc1157fe4c67" />

---

#  W2-PM5: Network Scanning with Zenmap

The network scanning exercise was performed in a **controlled VirtualBox NAT environment** using the:

```text
10.0.0.0/24
```

subnet.

---

# 7. Zenmap Ping Scan

### Command

```bash
nmap -sn 10.0.0.0/24
```

### Scan Configuration

| Parameter             | Value                           |
| --------------------- | ------------------------------- |
| **Target**            | `10.0.0.0/24`                   |
| **Scan Type**         | Ping Scan / Host Discovery      |
| **Addresses Covered** | 256                             |
| **Environment**       | Internal VirtualBox NAT Network |
| **Scan Time**         | 3.09 seconds                    |
| **Timestamp**         | 16 September 2026, 15:38        |

### Live Hosts Discovered

| IP Address | Observation                                                                |
| ---------- | -------------------------------------------------------------------------- |
| `10.0.0.1` | Host up; latency approximately 0.0051 seconds; QEMU virtual NIC identified |
| `10.0.0.2` | Host up; Kali Linux VM used as the scanner                                 |

### Security Relevance

The scan demonstrated how host discovery can identify reachable systems before performing more detailed authorized scanning.

### Evidence

<img width="1918" height="918" alt="zenmap-host" src="https://github.com/user-attachments/assets/4144ad00-a58a-4bfc-a02d-ab740ecc8ae5" />

---

# 8. Zenmap Topology View

After the ping scan, the **Topology** tab in Zenmap was used to visualize the discovered network.

The topology view displayed:

* `10.0.0.1`
* `10.0.0.2`
* Their relationship within the `10.0.0.0/24` network
* The topology legend and connection information

This provides a quick visual representation of the network structure discovered during the scan.

### Evidence

<img width="1918" height="918" alt="zemap-topology" src="https://github.com/user-attachments/assets/06bf75ca-6f41-4b97-a328-1cc173feb5a9" />

---

# My Learning

This practical demonstrated the following cybersecurity concepts:

* Domain and registration reconnaissance using **WHOIS**
* Web technology fingerprinting using **WhatWeb**
* DNS resolution using **Nslookup**
* HTTP header inspection using **curl**
* WAF identification using **wafw00f**
* DNS record enumeration using **dnsrecon**
* Host discovery using **Nmap / Zenmap**
* Basic network topology visualization
* Security observation and risk documentation
* Translating reconnaissance observations into security recommendations
* Working with a controlled VirtualBox networking environment

---

#  Evidence Collected

The following evidence was collected during the practical exercise:

| # | Evidence         | Command / Activity                 |
| - | ---------------- | ---------------------------------- |
| 1 | WHOIS            | `whois networkwalks.com`           |
| 2 | WhatWeb          | `whatweb networkwalks.com`         |
| 3 | Nslookup         | `nslookup networkwalks.com`        |
| 4 | curl             | `curl -I https://networkwalks.com` |
| 5 | wafw00f          | `wafw00f networkwalks.com`         |
| 6 | dnsrecon         | `dnsrecon -d networkwalks.com`     |
| 7 | Zenmap Ping Scan | `nmap -sn 10.0.0.0/24`             |
| 8 | Zenmap Topology  | Zenmap → Topology                  |


# 👤 Author

## Vraj Patel

**Cybersecurity Intern – B083**

| Detail        | Information                   |
| ------------- | ----------------------------- |
| **Programme** | Cybersecurity at Networkwalks |
| **Week**      | 02                            |
| **Modules**   | W2-PM1 & W2-PM5               |
| **Cohort**    | B083                          |

### 🔗 LinkedIn

[LinkedIn Profile](https://www.linkedin.com/in/vraj-patel-vp8816/)

---

#  Scope & Authorization

All reconnaissance and network-scanning activities documented in this repository were conducted within the authorized scope of the **Networkwalks educational programme**.

> **No exploitation, vulnerability scanning, or active attack was performed.**

---

##  Educational Project

**Cybersecurity Practical Project — Networkwalks**

**Cohort:** B083
**Week:** 02
**Modules:** W2-PM1 & W2-PM5
