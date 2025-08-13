Subdomain pakai database sertifikat publik dari crt.sh + dig + openssl.
Jadi ini cukup ringan di Termux dan langsung bisa jalan.

# 1. Script-nya
Buat file:
```
nano subcheck-lite.sh
```
Isi : 
```
#!/bin/bash

DOMAIN="$1"

if [ -z "$DOMAIN" ]; then
    echo "Usage: ./subcheck-lite.sh domain.com"
    exit 1
fi

echo "=== [ WHOIS INFO ] ==="
whois $DOMAIN | grep -E "Registrar:|Creation Date:|Registry Expiry Date:|Name Server:"

echo ""
echo "=== [ SUBDOMAIN ENUMERATION - crt.sh ] ==="
curl -s "https://crt.sh/?q=%25.$DOMAIN&output=json" \
    | jq -r '.[].name_value' \
    | sed 's/\*\.//g' \
    | sort -u > subdomains.txt
cat subdomains.txt

echo ""
echo "=== [ SUBDOMAIN SSL CHECK ] ==="
while read sub; do
    echo ""
    echo "--- $sub ---"
    IP=$(dig +short $sub | tail -n 1)
    echo "IP: $IP"
    if [ -n "$IP" ]; then
        echo | openssl s_client -connect ${sub}:443 -servername ${sub} 2>/dev/null \
        | openssl x509 -noout -issuer -subject -dates
    else
        echo "Tidak ada IP / kemungkinan offline"
    fi
done < subdomains.txt
```
# 2. Install Tools
Pastikan Termux punya semua paket:
```
pkg update && pkg upgrade -y
pkg install whois jq curl openssl dnsutils -y
```
# 3. Jalankan 
ganti <domain_target> dengan domain yang ingin kamu check
```
chmod +x subcheck-lite.sh
./subcheck-lite.sh <domain_target>
```
Atau 
```
./subcheck-lite.sh <domain_target>
```
contoh ./subcheck-lite.sh monad.xyz

# Sekian Terima Kasih
