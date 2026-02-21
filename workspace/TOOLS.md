# EagleEye — Tools Reference

## Quick Reference

### Recon Tools in PATH

```bash
subfinder httpx nuclei naabu dnsx katana amass 
ffuf gobuster nmap whatweb nikto shodan gau waybackurls
```

### PDF/Reporting Tools

```bash
# CLI tools
wkhtmltopdf qpdf pdftotext tesseract

# Python libraries (use in scripts)
# weasyprint - HTML/CSS to PDF (recommended)
# reportlab - programmatic PDF generation
# pypdf - PDF manipulation
# pdfplumber - PDF text/table extraction
```

**Skill Reference**: `/mnt/skills/public/pdf/SKILL.md`

---

## Output Conventions

|Type|Path Pattern|
|---|---|
|Scan output|`workspace/scans/{target}/{domain}-{tool}-{YYYY-MM-DD}.{ext}`|
|Reports|`workspace/reports/{domain}-summary-{YYYY-MM-DD}.md`|
|Targets|`target/TARGETS.md`|

---

## Phase-to-Tool Mapping

|Phase|Primary Tools|Decision Reference|
|---|---|---|
|1. OSINT|theHarvester, shodan, whois, dnsrecon|Passive only|
|2. Subdomains|subfinder → amass → dnsx|See SKILL.md §1-3|
|3. Live Hosts|httpx|See SKILL.md §6|
|4. Ports|naabu → nmap|See SKILL.md §4-5|
|5. Fingerprint|whatweb, wafw00f, sslscan|See SKILL.md §10|
|6. URL/Paths|waybackurls, gau, katana, gobuster, ffuf|See SKILL.md §7-9|
|7. Vuln Scan|nuclei, nikto|See SKILL.md §11-12|
|8. Report|weasyprint, reportlab|See pdf skill|

---

## Tool Decision Trees (Summary)

### Subdomain Enumeration

```
Quick scan needed?
├─► YES → subfinder -d $TARGET -silent
└─► NO  → subfinder + amass (both)
         └─► Need brute-force?
             └─► YES → + puredns with balanced wordlists
```

### Port Scanning

```
Quick discovery for pipeline?
├─► YES → naabu (faster)
└─► Need detailed services?
    └─► YES → nmap -sV -sC (after naabu)
    
Have root access?
├─► YES → naabu -scan-type s (SYN)
└─► NO  → naabu -scan-type c (CONNECT)
```

### Directory Discovery

```
Simple dir brute-force?
├─► YES → gobuster dir
└─► Need param/header fuzzing?
    └─► YES → ffuf
```

### Vulnerability Scanning

```
First scan of target?
├─► YES → nuclei -tags tech,exposure (discovery)
└─► Have tech fingerprint?
    ├─► YES → nuclei -t {tech}/ (targeted)
    └─► NO  → nuclei -tags cve,misconfig (broad)
```

---

## Standard Pipelines

### Full Subdomain Pipeline

```bash
subfinder -d $TARGET -all -silent > subs.txt
amass enum -passive -d $TARGET >> subs.txt
sort -u subs.txt | dnsx -a -silent | httpx -silent
```

### Quick Recon Pipeline

```bash
subfinder -d $TARGET -silent | httpx -silent | nuclei -tags tech
```

### Port-to-Vuln Pipeline

```bash
naabu -host $TARGET -top-ports 100 -silent | httpx -silent | nuclei -severity high,critical
```

### Path Discovery Pipeline

```bash
echo $TARGET | waybackurls > urls.txt
echo $TARGET | gau >> urls.txt
katana -u https://$TARGET -silent >> urls.txt
sort -u urls.txt
```

---

## Tool-Specific Notes

### subfinder

- Default: passive sources only
- `-all`: use all configured sources
- `-recursive`: find sub-subdomains
- **Always combine with amass for completeness**

### amass

- `enum -passive`: stealth mode
- `enum -active`: includes DNS queries
- `intel`: organization intelligence
- **Slower but more thorough than subfinder**

### dnsx

- Primary use: resolve subdomains to IPs
- `-cname`: check for takeover candidates
- `-wildcard`: filter wildcard responses
- **Bridge between subdomain and HTTP probing**

### naabu

- Fast port discovery
- Requires root for SYN scan
- `-top-ports 100`: common ports
- **Use before nmap for speed**

### nmap

- Detailed service enumeration
- `-sV`: version detection
- `-sC`: default scripts
- `-A`: aggressive (OS + version + scripts)
- **Use after naabu identifies open ports**

### httpx

- HTTP/HTTPS probing
- `-tech-detect`: identify technologies
- `-status-code -title`: basic metadata
- **Critical bridge to vuln scanning**

### katana

- Intelligent web crawler
- `-js-crawl`: parse JavaScript
- `-depth N`: crawl depth
- **Better than brute-force for finding endpoints**

### gobuster

- Directory brute-forcing
- `dir`: directory mode
- `dns`: subdomain mode
- `vhost`: virtual host discovery
- **Simple and fast for basic enum**

### ffuf

- Flexible fuzzing
- `FUZZ` placeholder
- Multi-position support
- **Use for param/header fuzzing**

### nuclei

- Template-based vuln scanning
- `-tags`: filter by category
- `-severity`: filter by risk
- **Primary vulnerability scanner**

### nikto

- Web server scanning
- `-Tuning`: select check types
- **Noisy - avoid when stealth needed**

---

## Wordlist Locations

### Subdomains

```
/usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt   # Quick (5K)
/usr/share/seclists/Discovery/DNS/all.txt                           # Standard (10K)
```

### Directories

```
/usr/share/seclists/Discovery/Web-Content/common.txt                 # Quick (5K)
/usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt  # Standard (220K)
```

### API

```
/usr/share/seclists/Discovery/Web-Content/api/api-endpoints.txt     # API paths
```

### Parameters

```
/usr/share/seclists/Discovery/Web-Content/burp-parameter-names.txt  # Common params
```

---

## Token-Saving Tips

### Pipeline Commands

```bash
# Run as single pipeline (saves 3 separate tool-call messages)
subfinder -d $TARGET -silent | dnsx -a -silent | httpx -silent
```

### Output Filtering

```bash
# Skip low-severity findings
nuclei -l urls.txt -severity medium,high,critical
```

### Summary Only

```bash
# Report counts, not full output
echo "Found: $(wc -l < results.txt) entries"
echo "Critical: $(grep -c '\[critical\]' nuclei.txt)"
```

### File References

```bash
# Reference file path, don't paste content
echo "Results saved to: $SCANDIR/$TARGET-nuclei-$DATE.txt"
```

---

## Troubleshooting

### Permission Issues

```bash
# Port scanning requires root for SYN
sudo naabu -host $TARGET -scan-type s

# Alternative without root
naabu -host $TARGET -scan-type c
```

### Rate Limiting

```bash
# Slow down tools when rate limited
subfinder -d $TARGET -sc 5 -delay 2
httpx -l hosts.txt -c 10 -timeout 15 -delay 2s
nuclei -l urls.txt -c 10 -rate-limit 30
```

### Memory Issues

```bash
# Process in batches
for d in $(cat domains.txt); do subfinder -d $d -o $d.txt; done

# Use streaming mode
httpx -l large.txt -stream
dnsx -l large.txt -stream
```

### Timeout Issues

```bash
# Increase timeouts
subfinder -d $TARGET -timeout 30
httpx -l hosts.txt -timeout 15
nmap -T2 --host-timeout 30m $TARGET
```
