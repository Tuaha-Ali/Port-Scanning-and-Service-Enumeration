# Port-Scanning-and-Service-Enumeration
# Web-Focused Port Scanning
        Common web service ports
 80    - HTTP
 443   - HTTPS
 8080  - HTTP alternate / proxy
 8443  - HTTPS alternate
 8000  - HTTP development
 8888  - HTTP alternate
 3000  - Node.js / Grafana
 5000  - Flask / Docker Registry
 9090  - Prometheus / various
 9443  - HTTPS alternate
 4443  - HTTPS alternate
 2083  - cPanel HTTPS
 2087  - WHM HTTPS
 10000 - Webmin
 7443  - Various
 8081  - HTTP alternate
 8181  - HTTP alternate
 9200  - Elasticsearch
 5601  - Kibana
 27017 - MongoDB
 6379  - Redis
 11211 - Memcached
 3306  - MySQL
 5432  - PostgreSQL

 **Quick web port scan**
nmap -p 80,443,8080,8443,8000,8888,3000,5000,9090,9443 -sV --open target.com

 **Full port scan**
    nmap -p- -sV --open -T4 target.com -oA full_scan

# Nmap Techniques for Web Services

           # Version detection
nmap -sV -p 80,443 target.com

 **Default scripts**
 
nmap -sC -p 80,443 target.com

 **Aggressive scan**
 
nmap -A -p 80,443 target.com

 **OS detection**
 
nmap -O target.com

 **Web-specific NSE scripts**
 
nmap -p 80,443 --script http-title target.com
nmap -p 80,443 --script http-headers target.com
nmap -p 80,443 --script http-methods target.com
nmap -p 80,443 --script http-enum target.com
nmap -p 80,443 --script http-robots.txt target.com
nmap -p 80,443 --script http-git target.com
nmap -p 80,443 --script http-config-backup target.com
nmap -p 80,443 --script http-default-accounts target.com
nmap -p 443 --script ssl-enum-ciphers target.com
nmap -p 443 --script ssl-cert target.com
nmap -p 443 --script ssl-heartbleed target.com

 **Vulnerability scanning**
 
nmap -p 80,443 --script "http-vuln-*" target.com
nmap -p 443 --script "ssl-*" target.com

 **All HTTP scripts**
 
nmap -p 80 --script "http-*" target.com

 **Scan multiple targets**
 
nmap -iL targets.txt -p 80,443,8080,8443 -sV --open -oA multi_scan

 **Speed optimization**
 
nmap -T4 -p- --min-rate=1000 target.com       # Fast
nmap -T5 -p- --min-rate=5000 target.com       # Insane (may miss results)
nmap -T2 -p 80,443 target.com                  # Polite (slow, stealthy)

 **Output formats**
 
nmap -p 80,443 -sV target.com -oN normal.txt   # Normal
nmap -p 80,443 -sV target.com -oX output.xml   # XML
nmap -p 80,443 -sV target.com -oG grep.txt     # Grepable
nmap -p 80,443 -sV target.com -oA all_formats  # All formats

# Masscan - Fast Port Scanning
      masscan -p1-65535 target.com --rate=1000 -oL masscan_results.txt
      masscan -p80,443,8080,8443,8000,3000,5000,9090 target.com --rate=500
      masscan -p80,443 10.0.0.0/24 --rate=1000 -oJ masscan.json

      **Output formats**
      
masscan -p80,443 target.com --rate=1000 -oL list.txt    # List
masscan -p80,443 target.com --rate=1000 -oJ json.txt    # JSON
masscan -p80,443 target.com --rate=1000 -oX xml.txt     # XML
masscan -p80,443 target.com --rate=1000 -oG grep.txt

**Parse masscan results for nmap follow-up**

cat masscan_results.txt | grep "open" | awk '{print $4}' | sort -u > live_ips.txt
cat masscan_results.txt | grep "open" | awk '{print $4":"$3}' | sed 's/\/tcp//' > ip_port_pairs.txt

**Follow up with nmap for detailed service info**
nmap -iL live_ips.txt -p $(cat masscan_results.txt | grep "open" | awk '{print $3}' | sed 's/\/tcp//' | sort -u | tr '\n' ',') -sV -sC

 **Service Fingerprinting**
 
    # Banner grabbing
nc -v target.com 80 <<< "HEAD / HTTP/1.0\r\n\r\n"
nc -v target.com 22
nc -v target.com 21

 **Nmap service probes**
 
nmap -sV --version-intensity 5 -p 80,443 target.com

 **Custom service identification**
 
curl -sI https://target.com | grep -iE "(Server|X-Powered|Via|X-Generator)"

 **WhatWeb for detailed fingerprinting**
 
whatweb -a 3 https://target.com

 **Identify web server behind CDN**
 
 Check origin IP:
curl -sI https://target.com | grep -iE "(cf-ray|x-served-by|x-cache|x-amz|x-azure)"

 **Direct IP access (bypass CDN)**
 
curl -sI -H "Host: target.com" https://ORIGIN_IP

 **SSL certificate analysis**
 
openssl s_client -connect target.com:443 -servername target.com 2>/dev/null | \
  openssl x509 -noout -subject -issuer -dates

# Service-Specific Enumeration
    # Elasticsearch (9200)
curl -s http://target.com:9200/
curl -s http://target.com:9200/_cat/indices?v
curl -s http://target.com:9200/_cluster/health
curl -s http://target.com:9200/_search?q=*

 **Kibana (5601)**
 
curl -s http://target.com:5601/api/status

 **Redis (6379)**
 
redis-cli -h target.com INFO
redis-cli -h target.com CONFIG GET *

 **MongoDB (27017)**
 
mongosh --host target.com --eval "db.adminCommand('listDatabases')"

 **Docker Registry (5000)**
 
curl -s http://target.com:5000/v2/_catalog
curl -s http://target.com:5000/v2/IMAGE/tags/list

 **Jenkins (8080)**
 
curl -s http://target.com:8080/api/json
curl -s http://target.com:8080/script  # Groovy console

 **Grafana (3000)**
 
curl -s http://target.com:3000/api/health
curl -s http://target.com:3000/api/datasources

 **Prometheus (9090)**
 
curl -s http://target.com:9090/api/v1/targets
curl -s http://target.com:9090/api/v1/label/__name__/values

 **Consul (8500)**
 
curl -s http://target.com:8500/v1/catalog/services
curl -s http://target.com:8500/v1/kv/?recurse

 **etcd (2379)**
 
curl -s http://target.com:2379/v2/keys/?recursive=true
  
# HTTP/HTTPS Service Analysis

        **httpx - HTTP toolkit**
        
echo target.com | httpx -ports 80,443,8080,8443,8000,3000,5000,9090 \
  -status-code -title -tech-detect -web-server -content-length \
  -follow-redirects -silent

 **Scan multiple hosts**
 
cat hosts.txt | httpx -ports 80,443,8080,8443 -status-code -title -silent -o httpx_results.txt

 **Detailed probe**
 
httpx -l hosts.txt -status-code -title -tech-detect -web-server \
  -content-length -content-type -cdn -ip -cname -tls-grab \
  -favicon -jarm -hash sha256 -o detailed_httpx.txt

 **Filter results**
 
cat httpx_results.txt | grep "200" > alive_200.txt
cat httpx_results.txt | grep "401\|403" > auth_required.txt
cat httpx_results.txt | grep -i "admin\|panel\|dashboard" > admin_panels.txt

 **JARM fingerprinting (TLS fingerprint)**
 
httpx -l hosts.txt -jarm -o jarm_results.txt
 Compare JARM hashes to identify similar servers

  **SSL/TLS Analysis**
  
    # testssl.sh - comprehensive SSL/TLS test
testssl.sh target.com
testssl.sh --full target.com
testssl.sh --vulnerable target.com
testssl.sh -U target.com  # vulnerabilities only

 **sslscan**
 
sslscan target.com

 **sslyze**
 
sslyze target.com
sslyze --regular target.com

#\ OpenSSL manual tests
 **Check TLS versions**
 
openssl s_client -connect target.com:443 -tls1    2>/dev/null && echo "TLS 1.0 supported"
openssl s_client -connect target.com:443 -tls1_1  2>/dev/null && echo "TLS 1.1 supported"
openssl s_client -connect target.com:443 -tls1_2  2>/dev/null && echo "TLS 1.2 supported"
openssl s_client -connect target.com:443 -tls1_3  2>/dev/null && echo "TLS 1.3 supported"

 **Check for weak ciphers**
 
nmap -p 443 --script ssl-enum-ciphers target.com

 **Certificate details**
 
echo | openssl s_client -connect target.com:443 2>/dev/null | \
  openssl x509 -noout -text | grep -A 2 "Subject\|Issuer\|Not After\|DNS:"

 **Check for Heartbleed**
 
nmap -p 443 --script ssl-heartbleed target.com

 **POODLE check**
 
nmap -p 443 --script ssl-poodle target.com


  **UDP Service Discovery**
        # UDP scan (slower than TCP)
nmap -sU -p 53,161,500,4500,1900 target.com

 **Common UDP services on web infrastructure**:
 53    - DNS
 161   - SNMP
 500   - IKE (VPN)
 4500  - IPsec NAT-T
 1900  - SSDP/UPnP
 123   - NTP
 514   - Syslog
 69    - TFTP

 **SNMP enumeration**
 
snmpwalk -v2c -c public target.com

 **NTP info**
 
ntpq -c readvar target.com

 **Network Mapping**
 
    Traceroute to target
traceroute target.com
traceroute -T -p 443 target.com  # TCP traceroute

 **Identify network boundaries**
 
 CDN → Load Balancer → Web Server → Application Server → Database

 **Detect load balancers**
 
 Multiple requests to see different server IDs
for i in $(seq 1 10); do
    curl -sI https://target.com | grep -iE "(Server|X-Served|X-Backend|X-Cache)"
done

 **lbd - load balancer detection**
 
lbd target.com

 **halberd**
 
halberd target.com

 Detect reverse proxy
 Signs: Via header, X-Forwarded-For processing, different error pages
curl -sI https://target.com | grep -i "via"

#  Port Scan Results Analysis Script(bash script)

          #!/bin/bash
 port_analysis.sh
TARGET=$1
OUTDIR="recon/${TARGET}/ports"
mkdir -p $OUTDIR

echo "=== Port Scanning & Service Enumeration: $TARGET ==="

 Phase 1: Quick discovery with masscan
echo "[1/4] Quick port discovery..."
masscan -p1-65535 $TARGET --rate=1000 -oL $OUTDIR/masscan.txt 2>/dev/null
OPEN_PORTS=$(grep "open" $OUTDIR/masscan.txt 2>/dev/null | awk '{print $3}' | sed 's/\/tcp//' | sort -un | tr '\n' ',')
echo "[*] Open ports: $OPEN_PORTS"

 Phase 2: Detailed nmap scan on discovered ports
echo "[2/4] Service detection..."
if [ -n "$OPEN_PORTS" ]; then
    nmap -sV -sC -p ${OPEN_PORTS%,} $TARGET -oA $OUTDIR/nmap_detail 2>/dev/null
fi

 Phase 3: HTTP probing on all ports
echo "[3/4] HTTP probing..."
echo $TARGET | httpx -ports 80,443,8080,8443,8000,8888,3000,5000,9090,9200,9443,10000 \
  -status-code -title -tech-detect -silent > $OUTDIR/httpx.txt 2>/dev/null

 Phase 4: SSL analysis on HTTPS ports
echo "[4/4] SSL analysis..."
for port in 443 8443 9443 4443; do
    timeout 5 openssl s_client -connect $TARGET:$port 2>/dev/null | \
      openssl x509 -noout -subject -issuer -dates >> $OUTDIR/ssl_info.txt 2>/dev/null
done

echo "[*] Scan complete. Results in $OUTDIR/"
