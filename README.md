<div align="center">

# 🕵️ Footprinting & Reconnaissance — Week 2

**Passive OSINT and footprinting against a live target using Kali Linux tools**
</div>

<p align="center">
  <img src="https://img.shields.io/badge/Skill-Cybersecurity-404040?style=flat-square&labelColor=C00000" />
  <img src="https://img.shields.io/badge/Kali%20Linux-v2026.2-E87500?style=flat-square&labelColor=000000&logo=kalilinux&logoColor=white" />
  <img src="https://img.shields.io/badge/Footprinting-404040?style=flat-square&labelColor=C00000" />
  <img src="https://img.shields.io/badge/OSINT-238F89?style=flat-square&labelColor=000000" />
  <img src="https://img.shields.io/badge/theHarvester-404040?style=flat-square&labelColor=C00000" />
  <img src="https://img.shields.io/badge/GitHub-404040?style=flat-square&labelColor=0070C0&logo=github&logoColor=white" />
  <img src="https://img.shields.io/badge/NetworkWalks-404040?style=flat-square&labelColor=C00000" />
  <img src="https://img.shields.io/badge/Ethical%20Hacking-E87500?style=flat-square&labelColor=000000&logo=kalilinux&logoColor=white" />
</p>

---

## 📌 Project Overview

This project documents **Week 2** of the Networkwalks Cybersecurity Internship: the **footprinting and reconnaissance** phase of ethical hacking.

Using only publicly available information and passive/light-touch tools, a complete footprint profile of the live website **networkwalks.com** was built — covering its registrar, hosting provider, technology stack, DNS infrastructure, and WAF protection — without directly attacking or disrupting the target.

A second module used **theHarvester** to demonstrate OSINT email/sub-domain gathering against a well-known public domain (microsoft.com), showcasing how passive reconnaissance tools query dozens of public data sources.

---

## 🛡️ Authorization & Scope

All testing against **networkwalks.com** was carried out under a signed **Letter of Authorization** (Ref. `NW-LOA-B082-017`) issued by Networkwalks, scoping activity strictly to `networkwalks.com` and the tester's own LAN.

theHarvester's OSINT lookups (Task W2-PM4) query public search indexes only and never send traffic directly to the target's own infrastructure, so no systems outside the authorized scope were touched or tested.

⚠️ **Important:** This repository is for education and research purposes only. Do not use these tools or techniques against any system you do not own or have explicit written permission to test.

---

## 🎯 Modules Covered

| Module | Topic | Tools Used |
|---|---|---|
| **W2-PM1** | Footprinting with Multiple Kali Tools | whois, whatweb, nslookup, curl, wafw00f, dnsrecon |
| **W2-PM4** | Footprinting with theHarvester | theHarvester (Baidu source, then all sources) |

---

# 🪜 Module W2-PM1 — Footprinting with Multiple Kali Tools

**Target:** `networkwalks.com` (in scope per LOA)

## Task 1. whois — Domain Registration Lookup

```bash
$ whois networkwalks.com
```

![](images/whois1.png)
![](images/whois2.png)
![](images/whois3.png)

**Finding:** The domain is registered via **GoDaddy.com, LLC**, created **06 Nov 2019**, expiring **06 Nov 2027**. Name servers `NS6135.HOSTGATOR.COM` / `NS6136.HOSTGATOR.COM` reveal **HostGator** as the hosting provider.

---

## Task 2. whatweb — Technology Fingerprinting

```bash
$ whatweb networkwalks.com
```

![](images/whatweb.png)

**Finding:** Apache web server running **WordPress 7.1.1**, Bootstrap 7.1.1, JQuery 3.7.1, and the **WordPress Download Manager** plugin (v3.3.58). Server IP: `192.232.216.135`.

---

## Task 3. nslookup — DNS Resolution

```bash
$ nslookup networkwalks.com
```

![](images/nslookup.png)

**Finding:** `networkwalks.com` resolves to `192.232.216.135`.

---

## Task 4. curl -I — HTTP Response Headers

```bash
$ curl -I https://networkwalks.com
```

![](images/curl.png)

**Finding:** `HTTP/2 200`, Apache banner, WordPress REST API exposed at `/wp-json/`, session cookie `__wpdm_client` set, cached via WordPress/nginx-cache.

---

## Task 5. wafw00f — Web Application Firewall Detection

```bash
$ wafw00f networkwalks.com
```

![](images/wafw00f.png)

**Finding:** The site is protected by a **ModSecurity (SpiderLabs)** WAF.

---

## Task 6. dnsrecon — Full DNS Enumeration

```bash
$ dnsrecon -d networkwalks.com
```

![](images/dnsrecon.png)

**Finding:** SOA/NS records confirm HostGator. MX record: `mail.networkwalks.com`. SPF (TXT) record authorizes `websitewelcome.com` to send mail. 8 SRV records found — all cPanel autodiscover entries across the `184.94.x.x` range.

---

### 📊 Summary of Findings — W2-PM1

| Tool | Key Finding |
|---|---|
| whois | GoDaddy registrar; HostGator name servers |
| whatweb | Apache + WordPress 7.1.1 + WP Download Manager 3.3.58 |
| nslookup | Resolves to 192.232.216.135 |
| curl -I | WordPress REST API exposed at /wp-json/ |
| wafw00f | Protected by ModSecurity (SpiderLabs) WAF |
| dnsrecon | HostGator DNS, cPanel autodiscover SRV records |

---

# 🪜 Module W2-PM4 — Footprinting with theHarvester

**Target:** `microsoft.com` (per lab guide — passive OSINT only, no direct interaction with target infrastructure)

## Task 1. theHarvester with Baidu Source

```bash
$ theHarvester -d microsoft.com -l 1000 -b baidu
```

![](images/harvester1.png)

**Finding:** No IPs, emails, people, or hosts were returned from the Baidu source at the time of the scan — a reminder that OSINT results vary depending on what each individual source currently has indexed.

## Task 2. theHarvester with All Sources

```bash
$ theHarvester -d microsoft.com -l 50 -b all
```

![](images/harvester2.png)
![](images/harvester3.png)

**Finding:** Most third-party sources (Bevigil, BuiltWith, Censys, Criminal IP, Shodan, SecurityTrails, VirusTotal, ZoomEye, etc.) returned "Missing API key" warnings, since Kali's default theHarvester install ships without pre-configured API keys for paid/registered services. Free, key-less sources (Chaos, Certspotter, Baidu, CRTsh, Fofa, DuckDuckGo) were still queried successfully — illustrating a real-world limitation of free-tier OSINT tooling.

---

# 🐞 Problems Encountered & Solutions

## Problem 1. Incorrect Command-Line Flag

Running Task 1 of W2-PM4 initially failed:

```
theHarvester -d microsoft.com -I 1000 -b baidu
theHarvester: error: unrecognized arguments: -I 1000
```

**Cause:** A capital `I` was mistakenly typed instead of a lowercase `l` for the `--limit` flag — the two characters look nearly identical in many terminal fonts.

**Solution:** Retyped the command manually with the correct lowercase `l`, confirmed via `theHarvester --help` (`-l, --limit LIMIT`):

```bash
theHarvester -d microsoft.com -l 1000 -b baidu
```

The command then ran successfully.

---

# 💡 What I Learned

- How six different reconnaissance tools each reveal a different piece of a target's public footprint, and how combined they build a complete profile without directly touching or attacking the target.
- How a WordPress site's exact software stack, plugin versions, and hosting provider can be fingerprinted entirely from public HTTP responses and DNS records.
- The difference between single-source and multi-source OSINT gathering with theHarvester, and why comprehensive real-world OSINT work requires registering for multiple API keys.
- The importance of double-checking command-line flags carefully — visually similar characters (capital `I` vs lowercase `l`) can cause silent command failures.
- Why passive reconnaissance is considered the quietest and hardest-to-detect phase of a security assessment.

---

# 🔐 Security & Ethical Use

This repository is intended strictly for education and research purposes, as part of the Networkwalks Cybersecurity Internship. All testing against networkwalks.com was performed under a signed Letter of Authorization. Do not use these techniques against any system without explicit written permission.

---

# 🔗 Tools & Resources

- **Kali Linux:** [https://kali.org/get-kali](https://kali.org/get-kali)
- **theHarvester:** [https://github.com/laramies/theHarvester](https://github.com/laramies/theHarvester)

---

# 👤 Author

**Boahene Prince**
Cybersecurity Intern, Batch B083B — Networkwalks

LinkedIn: [https://www.linkedin.com/in/boahene-prince-603b08372/](https://www.linkedin.com/in/boahene-prince-603b08372/)

---

## 📌 Project Information

**Program Name:** Cybersecurity at Networkwalks | **Week:** 02 | **Project:** Footprinting & Reconnaissance (Multiple Kali Tools + theHarvester) | **Repository:** GitHub
