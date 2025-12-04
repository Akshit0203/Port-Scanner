# 🔍 Multi-threaded Port Scanner (Python)

A fast TCP port scanner built using Python's `socket` and `threading` modules.  
Scans ports from **1 to 65535** concurrently and displays open ports.

---

## 🚀 Features
- Multi-threaded scanning for high speed
- Scans full port range (1–65535)
- Graceful exit on `Ctrl + C`
- Resolves hostnames to IP address
- Simple and beginner-friendly structure

---

## 📦 Requirements
- Python 3.x

No external modules required.

---

## 🕹 Usage

### Run the script:
```bash
python scanner.py <target>
```

Example:
```python scanner.py 192.168.1.10```
or
```python scanner.py google.com```

### 📌 Output

```
--------------------------------------------------
Scanning target 142.250.182.14
Time started: 2025-12-04 14:32:10.923423
--------------------------------------------------
Port 80 is open
Port 443 is open
Scan completed!
```

### 🧠 How it Works
- Creates one thread per port to increase speed
- Detects successful connections (result == 0)
- Prints open ports to the terminal

