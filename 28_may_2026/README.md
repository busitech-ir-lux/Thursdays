# DNS & Web Enumeration Notes

> Structured reconnaissance and enumeration notes written in Markdown for Obsidian and GitHub compatibility.

---

# Task 1 — WHOIS Enumeration

## Question 1

### Objective

Perform a WHOIS lookup against the `paypal.com` domain and identify the Registrar IANA ID.

### Methodology

I used the `whois` utility to retrieve domain registration information for `paypal.com`.
To reduce the output and focus only on the registrar information, I filtered the results using `grep`.

### Command

```bash
$ whois paypal.com | grep -i iana
```

### Result

```text
Registrar IANA ID: 292
Registry Domain ID: 8017040_DOMAIN_COM-VRSN
```

### Answer

**Registrar IANA ID:** `292`

---

## Question 2

### Objective

Identify the administrative email contact for the `tesla.com` domain.

### Methodology

I queried the WHOIS information for `tesla.com` and searched for email-related entries using `grep`.

### Command

```bash
$ whois tesla.com | grep -i email
```

### Result

```text
Tech Email: admin@dnstinations.com
```

### Answer

**Administrative Email:** `admin@dnstinations.com`

---

## Question 3

### Objective

Identify the registrar name for `dlh.lu`.

### Methodology

I performed a WHOIS lookup against the domain and filtered the output for registrar-related fields.

### Command

```bash
$ whois dlh.lu | grep -i registrar
```

### Result

```text
registrar-name: Fondation Restena
```

### Answer

**Registrar Name:** `Fondation Restena`

---

# Task 2 — DNS Enumeration

## Question 1

### Objective

Identify the IP addresses associated with `inlanefreight.com`.

### Methodology

I used `nslookup` against Google's public DNS server (`8.8.8.8`) to retrieve the DNS resolution records for the domain.

### Command

```bash
$ nslookup inlanefreight.com 8.8.8.8
```

### Result

```text
Server:         8.8.8.8
Address:        8.8.8.8#53

Non-authoritative answer:

Name:   inlanefreight.com
Address: 134.209.24.248

Name:   inlanefreight.com
Address: 2a03:b0c0:1:e0::32c:b001
```

### Answer

* IPv4 Address: `134.209.24.248`
* IPv6 Address: `2a03:b0c0:1:e0::32c:b001`

---

## Question 2

### Objective

Identify the PTR record associated with `134.209.24.248`.

### Methodology

I performed a reverse DNS lookup using `dig -x` to identify which domain points back to the target IP address.

### Command

```bash
$ dig -x 134.209.24.248
```

### Result

```text
248.24.209.134.in-addr.arpa. 1371 IN PTR inlanefreight.com.
```

### Answer

**PTR Record:** `inlanefreight.com`

---

## Question 3

### Objective

Identify the mail server configured for `facebook.com`.

### Methodology

I queried the MX (Mail Exchange) records using `dig` in order to determine which mail server handles email traffic for the domain.

### Command

```bash
$ dig facebook.com MX
```

### Result

```text
facebook.com. 2605 IN MX 10 smtpin.vvv.facebook.com.
```

### Answer

**Mail Server:** `smtpin.vvv.facebook.com`

---

# Task 3 — Subdomain Enumeration

## Question 1

### Objective

Enumerate subdomains associated with `inlanefreight.com`.

### Methodology

I used `subfinder`, a passive subdomain enumeration tool, to discover publicly known subdomains related to the target domain.

The `--silent` flag was used to simplify the output and display only discovered subdomains.

### Command

```bash
$ subfinder -d inlanefreight.com --silent
```

### Result

```text
api.inlanefreight.com
azure.inlanefreight.com
gateway.inlanefreight.com
www.inlanefreight.com
support.inlanefreight.com
blog.inlanefreight.com
mail1.inlanefreight.com
ns1.inlanefreight.com
ns2.inlanefreight.com
vpn.inlanefreight.com
customer.inlanefreight.com
my.inlanefreight.com
ns3.inlanefreight.com
```

### Answer

Discovered subdomains:

* `api.inlanefreight.com`
* `azure.inlanefreight.com`
* `gateway.inlanefreight.com`
* `www.inlanefreight.com`
* `support.inlanefreight.com`
* `blog.inlanefreight.com`
* `mail1.inlanefreight.com`
* `ns1.inlanefreight.com`
* `ns2.inlanefreight.com`
* `vpn.inlanefreight.com`
* `customer.inlanefreight.com`
* `my.inlanefreight.com`
* `ns3.inlanefreight.com`

---

# Task 4 — Virtual Host Enumeration

## Question 1

### Objective

Identify valid virtual hosts configured on `inlanefreight.htb`.

### Methodology

I used `ffuf` to fuzz potential virtual hosts by modifying the `Host` HTTP header.

The supplied wordlist (`virtualhosts-DLH.txt`) contained possible subdomain prefixes.
The `-fs 116` option filtered out responses with a size of 116 bytes to eliminate false positives.

### Command

```bash
$ ffuf -w virtualhosts-DLH.txt \
-H "HOST: FUZZ.inlanefreight.htb" \
-u http://inlanefreight.htb:31120 \
-fs 116
```

### Result

```text
support [Status: 200]
browse  [Status: 200]
blog    [Status: 200]
admin   [Status: 200]
vm5     [Status: 200]
forum   [Status: 200]
```

### Answer

Discovered virtual hosts:

* `support.inlanefreight.htb`
* `browse.inlanefreight.htb`
* `blog.inlanefreight.htb`
* `admin.inlanefreight.htb`
* `vm5.inlanefreight.htb`
* `forum.inlanefreight.htb`

---

# Task 5 — Web Technology Enumeration

## Question 1

### Objective

Determine the Apache version running on `inlanefreight.com`.

### Methodology

I first used `whatweb` to fingerprint the web technologies used by the target website.

To verify the web server information, I then used `curl -I` to inspect the HTTP response headers.

### Commands

```bash
$ whatweb https://inlanefreight.com
```

```bash
$ curl -I https://inlanefreight.com
```

### Result

```text
Server: Apache/2.4.41 (Ubuntu)
```

### Answer

**Apache Version:** `2.4.41`

---

## Question 2

### Objective

Identify the CMS and version used on `inlanefreight.com`.

### Methodology

I fetched the website source code and searched for references to WordPress-related assets and identifiers.

### Command

```bash
$ curl inlanefreight.com --follow | grep -i wordpress
```

### Observation

Several WordPress references were identified in the HTML source code and linked resources.

### Answer

* CMS: `WordPress`
* Version: `Not explicitly disclosed`

---

## Question 3

### Objective

Determine the operating system running the web server.

### Methodology

I used both Wappalyzer and `whatweb` to fingerprint the server-side technologies and operating system.

### Commands

```bash
$ whatweb https://inlanefreight.com | grep -i server
```

### Result

```text
HTTPServer[Ubuntu Linux][Apache/2.4.41 (Ubuntu)]
```

### Answer

**Operating System:** `Ubuntu Linux`

---

# Task 6 — Web Crawling & Spidering

## Question 1

### Objective

Identify where future reports will be stored after spidering `inlanefreight.com`.

### Methodology

After spidering the target website, I inspected the discovered URLs and API endpoints for references to future reports.

The WordPress REST API endpoint revealed references to pages associated with future reports.

### Findings

```text
https://www.inlanefreight.com/index.php/news/
```

### Answer

**Location:** `www.inlanefreight.com`

./Screenshot_2026-05-28_12_08_49.png

---

# Task 7 — Historical Reconnaissance

## Question 1

### Objective

Identify the username of the user posting on ChatGPT on July 3rd, 2023 at 22:56:32.

### Methodology

I inspected archived historical snapshots and reviewed the post metadata.

### Result

```text
by [adminGPT] Jul 3rd, 2023
```

### Answer

**Username:** `adminGPT`

---

## Question 2

### Objective

Determine which application previously owned the `facebook.com` domain on February 17th, 2003.

### Methodology

I used historical web archive records to inspect older snapshots of the domain.

### Result

```text
AboutFace: Software to Manage Your World
```

### Answer

**Previous Owner/Application:** `AboutFace`

---

## Question 3

### Objective

Identify the original landing page message for `dlh.lu`.

### Methodology

I reviewed archived snapshots of the domain to identify its earliest available homepage message.

### Result

```text
DLH Stay Tuned....
```

### Answer

**Message:** `DLH Stay Tuned....`

---

# Task 8 — Advanced Enumeration

## Question 1

### Objective

Identify the Registrar IANA ID for `inlanefreight.com`.

### Methodology

I queried the WHOIS information for the domain and filtered the output for the IANA field.

### Command

```bash
$ whois inlanefreight.com | grep -i iana
```

### Result

```text
Registrar IANA ID: 468
```

### Answer

**Registrar IANA ID:** `468`

---

## Question 2

### Objective

Identify the HTTP server software powering `inlanefreight.htb`.

### Methodology

I used `whatweb` to fingerprint the technologies and HTTP server running on the target system.

### Command

```bash
$ whatweb http://inlanefreight.htb:30587
```

### Result

```text
HTTPServer: nginx
```

### Answer

**HTTP Server Software:** `nginx`

---

## Question 3

### Objective

Identify the API key located inside the hidden administrative directory.

### Methodology

First, I attempted to discover additional subdomains using `ffuf`.

After identifying a valid host, I added it to the `/etc/hosts` file so the system could properly resolve it locally.

Next, I used OWASP ZAP spidering functionality to enumerate hidden directories and endpoints.

While reviewing the `robots.txt` file, I identified the following hidden directory:

```text
/admin_h1dd3n
```

I then manually browsed to the directory and located the exposed API key.

### Command

```bash
$ ffuf -w subdomains_DLH.txt \
-u http://FUZZ.inlanefreight.htb:30587
```

### Discovered URL

```text
http://web1337.inlanefreight.htb:30587/admin_h1dd3n/
```

### Answer

**API Key:** `e963d863ee0e82ba7080fbf558ca0d3f`

---

## Question 4

### Objective

Identify the email address discovered after crawling `inlanefreight.htb`.

### Methodology

To discover additional virtual hosts, I performed vhost fuzzing using `ffuf` and the provided wordlist.

After discovering the `dev` virtual host, I added it to the `/etc/hosts` file.

Next, I launched OWASP ZAP and crawled the entire website with unrestricted spider depth to ensure all pages and endpoints were enumerated.

After the crawl completed, I searched the collected responses for the `@` symbol to identify exposed email addresses.

### Command

```bash
$ ffuf -w ./subdomains_DLH.txt \
-H "HOST: FUZZ.web1337.inlanefreight.htb:30587" \
-u http://web1337.inlanefreight.htb:30587 \
-fs 120
```

### Discovered Virtual Host

```text
dev.web1337.inlanefreight.htb
```

### Answer

**Discovered Email:** `1337testing@inlanefreight.htb`

---

## Question 5

### Objective

Identify the new API key planned by the developers.

### Methodology

After completing the previous crawling and enumeration steps, I searched through the spidered responses and source files for references to API-related values.

While inspecting the collected content, I identified another API key value that appeared to be intended for future deployment.

### Result

```text
ba988b835be4aa97d068941dc852ff33
```

### Answer

**New API Key:** `ba988b835be4aa97d068941dc852ff33`

---

# Tools & Technologies Used

* `whois`
* `dig`
* `nslookup`
* `subfinder`
* `ffuf`
* `curl`
* `whatweb`
* `OWASP ZAP`
* `Wappalyzer`

---

# Key Enumeration Techniques

* WHOIS reconnaissance
* DNS enumeration
* PTR record analysis
* MX record analysis
* Subdomain discovery
* Virtual host fuzzing
* Web fingerprinting
* CMS detection
* Spidering and crawling
* Historical OSINT analysis
* Hidden directory discovery

---

