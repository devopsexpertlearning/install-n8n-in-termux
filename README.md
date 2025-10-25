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

### Secure using SSL 

```
apt install certbot python3-certbot-nginx

certbot --nginx -d yourdomain.com -d www.yourdomain.com

```

## Notes

- Using `--tunnel` is convenient for testing webhooks but **not recommended for production**.
- Ensure SQLite is installed and built properly, otherwise n8n may fail to start.
- PM2 ensures n8n runs in the background even if Termux is closed.
