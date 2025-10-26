# n8n Setup in Termux

This repository provides instructions to install and run **n8n** (workflow automation tool) on **Termux** using Node.js and SQLite.

---

## Prerequisites

- Android device with Termux installed
- Internet connection

---

## Installation Steps

### 1. Update Termux packages and add root repository
```
pkg install root-repo
pkg update
```

### 2. Install Python 3
```
pkg install python3
```

### 3. Install Node.js LTS (16.x)
```
pkg install nodejs-lts
```

### 4. Install dependencies for compiling SQLite from source
```
pkg install clang libsqlite pkg-config python make binutils
export GYP_DEFINES="android_ndk_path=''"
```

### 5. Install SQLite3 from source
```
npm install sqlite3 --build-from-source --sqlite=/data/data/com.termux/files/usr/bin/sqlite3 -g
```

### 6. Install n8n globally
```
npm install n8n -g --sqlite=/data/data/com.termux/files/usr/bin/sqlite3
```

### 7. Install PM2 to manage n8n as a background process
```
npm install pm2 -g
```

---

## Running n8n

### Start n8n without tunnel (webhooks on local network only)
```
pm2 start n8n
```

### Start n8n with tunnel (for webhooks, not recommended for production)
```
pm2 start n8n --tunnel
```

### Save PM2 process list for persistence
```
pm2 save
```

### Check the status of PM2 processes
```
pm2 status
```

---

## Network Setup

### Install net-tools to check Termux IP
```
pkg install net-tools
```

### Display network interfaces and IP address
```
ifconfig
```

### Access n8n in your browser
```
http://<your-ip>:5678
```

Replace `<your-ip>` with the IP address from `ifconfig`.

---

## Note: If want to expose this with internal IPs in internal Network with Self Signed Certificate, please use caddy as proxy server.

Please run below script for SSL, added endpoint in Caddyfile and start it

```bash

#!/bin/bash                                                 
# ------------------------------                            # Configuration                                             # ------------------------------
CERT_DIR="$HOME/caddy_certs"                                CERT_FILE="$CERT_DIR/fullchain.pem"                         KEY_FILE="$CERT_DIR/privkey.pem"
CONF_FILE="$CERT_DIR/openssl.cnf"                           CADDYFILE="$HOME/Caddyfile"
CADDY_SERVICE="caddy"  # Adjust if running differently      
mkdir -p "$CERT_DIR"                                                                                                    # ------------------------------
# Detect all internal IPv4 addresses                        # ------------------------------
IPS=$(ifconfig | grep -oP '\d+\.\d+\.\d+\.\d+' | grep -v '^127\.0\.0\.1$' | grep -v '^255\.' | grep -v '\.255$')                                                                    if [ -z "$IPS" ]; then                                          echo "No valid internal IPs found. Exiting."
    exit 1                                                  fi

echo "Detected IPs:"
echo "$IPS"

# ------------------------------
# Generate OpenSSL config with proper SANs
# ------------------------------
cat > "$CONF_FILE" <<EOL
[req]
distinguished_name = req_distinguished_name
req_extensions = v3_req
prompt = no

[req_distinguished_name]
CN = internal-server

[v3_req]
subjectAltName = @alt_names

[alt_names]
EOL

i=1
for ip in $IPS; do
    echo "IP.$i = $ip" >> "$CONF_FILE"
    ((i++))
done

# ------------------------------
# Generate self-signed certificate
# ------------------------------

openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout "$KEY_FILE" \
  -out "$CERT_FILE" \
  -config "$CONF_FILE" \
  -extensions 'v3_req'

echo "Certificate generated:"
echo "Private Key: $KEY_FILE"
echo "Certificate: $CERT_FILE"
echo "Valid for IPs:"
echo "$IPS"

# ------------------------------
# Generate Caddyfile dynamically
# ------------------------------
cat > "$CADDYFILE" <<EOL
# Disable automatic HTTPS globally
{
    auto_https off
}
EOL

for ip in $IPS; do
    cat >> "$CADDYFILE" <<EOL

https://$ip:8080 {
    reverse_proxy http://localhost:5678
    tls $CERT_FILE $KEY_FILE
}
EOL
done

echo "Caddyfile updated with current IPs:"
echo "$IPS"

# ------------------------------
# Reload Caddy
# ------------------------------
if pgrep -x "$CADDY_SERVICE" > /dev/null; then
    echo "Reloading Caddy to apply new certificate..."
    pgrep -x caddy > /dev/null && kill $(pgrep -x caddy)
    caddy run --config "$CADDYFILE" &
else
    echo "Caddy is not running. Starting it..."
    caddy run --config "$CADDYFILE" &
fi

```

## Alternative Use Nginx

```bash
#You can also use nginx with cerbot

apt install certbot python3-certbot-nginx

certbot --nginx -d yourdomain.com -d www.yourdomain.com

```

## Notes

- Using `--tunnel` is convenient for testing webhooks but **not recommended for production**.
- Ensure SQLite is installed and built properly, otherwise n8n may fail to start.
- PM2 ensures n8n runs in the background even if Termux is closed.
