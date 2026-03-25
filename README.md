# sub2ip

A fast, lightweight subdomain-to-IP resolver built for bug hunters and security researchers.

Takes a list of subdomains, resolves them concurrently, and outputs **unique IPs only** — one per line. No bloat, no dependencies, pipe-friendly.

---

## Features

- Concurrent DNS resolution via threading
- Deduplicates IPs automatically
- Handles IPv4 and IPv6
- Silently drops unresolvable/dead subdomains
- Zero external dependencies (pure Python stdlib)
- Pipe-friendly — works seamlessly with subfinder, amass, nmap, nuclei, httpx

---

## Installation

```bash
git clone https://github.com/mridha-92/sub2ip.git
cd sub2ip
chmod +x sub2ip
sudo mv sub2ip /usr/local/bin/sub2ip
```

---

## Usage

```bash
# From file
sub2ip -i subdomains.txt

# From stdin (pipe)
cat subdomains.txt | sub2ip -i -

# Save output to file
sub2ip -i subdomains.txt -o ips.txt

# Tune threads and timeout
sub2ip -i subdomains.txt -w 100 -t 5
```

### Options

| Flag | Default | Description |
|------|---------|-------------|
| `-i, --input` | required | Subdomain list file or `-` for stdin |
| `-o, --output` | stdout | Output file for resolved IPs |
| `-w, --workers` | 50 | Number of concurrent threads |
| `-t, --timeout` | 3.0 | DNS timeout per host (seconds) |

---

## Recon Pipeline Examples

**Basic — subfinder → sub2ip → nmap:**
```bash
subfinder -d target.com -silent | sub2ip -i - | nmap -iL /dev/stdin -T4 --open -p 80,443,8080,8443
```

**Full pipeline with file output:**
```bash
subfinder -d target.com -silent | sub2ip -i - -o ips.txt && \
sort -u ips.txt -o ips.txt && \
nmap -iL ips.txt -T4 -p 80,443,8080,8443,8888,3000,3001,4443,5000,9090,9443 \
     --open -sV --script=http-title,http-server-header -oN nmap_results.txt
```

**Handle IPv4 and IPv6 separately:**
```bash
sub2ip -i subs.txt -o ips.txt

# IPv4
grep -v ':' ips.txt | nmap -iL /dev/stdin -T4 --open -oN nmap_v4.txt

# IPv6
grep ':' ips.txt | nmap -6 -iL /dev/stdin -T4 --open -oN nmap_v6.txt
```

**Chain with httpx for live web discovery:**
```bash
subfinder -d target.com -silent | sub2ip -i - | httpx -silent
```

**Chain with nuclei:**
```bash
sub2ip -i subs.txt | httpx -silent | nuclei -t ~/nuclei-templates/
```

---

## Requirements

- Python 3.6+
- Linux / Kali (tested on Kali 2024)

---

## Disclaimer

This tool is intended for **authorized security testing and bug bounty hunting only**. Only use it against targets you have explicit permission to test. The author is not responsible for any misuse.

---

## License

MIT License — see [LICENSE](LICENSE)
