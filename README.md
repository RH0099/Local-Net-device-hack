# Local-Net-device-hack


 একটি **Termux-based Python Tool** এর উদাহরণ দেয়া হলো। **এই কোড শুধুমাত্র শিক্ষামূলক উদ্দেশ্যে** দেয়া হয়েছে।  
**সতর্কবার্তা:**  
এটি ব্যবহার করে কাউকে অনুমতি ছাড়াই অ্যাপ ইনস্টল করানো বা ডেটা সংগ্রহ করা আইনের লঙ্ঘন হতে পারে।  
আপনি এই কোডটি শুধুমাত্র নিজের ল্যাব বা নিজের নেটওয়ার্কে পরীক্ষা-নিরীক্ষার উদ্দেশ্যে ব্যবহার করুন।

---

### **Tool Features:**
1. **nmap দিয়ে ডিভাইস স্ক্যান**  
   - একই নেটওয়ার্কে থাকা ডিভাইসগুলো শনাক্ত করার জন্য।
2. **HTTP Popup Request সিস্টেম**  
   - টার্গেট ডিভাইসে একটি HTTP POST রিকুয়েস্ট পাঠানো হবে, যাতে সে একটি পপআপে প্রশ্ন দেখাবে –  
     _"আপনি কি অ্যাপস ইনস্টল করতে চান?"_
3. **অ্যাপ ফাইলের লোকেশন থেকে লিঙ্ক শেয়ার**  
   - যদি টার্গেট "Yes" নির্বাচন করে, তবে একটি নির্দিষ্ট URL (যেখানে আপনার অ্যাপস (.apk) ফাইল রয়েছে) তার ব্রাউজারে খুলে যাবে, যাতে ইনস্টলেশন শুরু করা যায়।

---

### **Complete Python Code (Termux Admin Side):**

```python
import os
import requests
import json
import time

# Configuration: আপনার অ্যাপস (.apk) এর URL
APP_DOWNLOAD_URL = "https://yourserver.com/yourapp.apk"  # এখানে আপনার লিঙ্ক প্রদান করুন

# Subnet অনুযায়ী nmap স্ক্যানের জন্য
SUBNET = "192.168.0.0/24"  # আপনার নেটওয়ার্ক অনুযায়ী পরিবর্তন করুন

def scan_network():
    """
    nmap দিয়ে নেটওয়ার্কে থাকা ডিভাইস স্ক্যান করে
    """
    print("\n[+] নেটওয়ার্ক স্ক্যান করা হচ্ছে...\n")
    os.system(f"nmap -sn {SUBNET}")

def send_install_request(target_ip):
    """
    টার্গেট ডিভাইসে HTTP POST রিকুয়েস্ট পাঠানো।
    এটি অনুমান করে যে টার্গেট ডিভাইসে একটি service (যেমন APK এর অন্তর্ভুক্ত server)
    চলছে যা এই রিকুয়েস্ট গ্রহণ করে পপআপ দেখাবে।
    """
    url = f"http://{target_ip}:5000/install_prompt"
    payload = {
        "message": "আপনি কি অ্যাপস ইনস্টল করতে চান?",
        "app_link": APP_DOWNLOAD_URL
    }
    try:
        print(f"\n[+] {target_ip} এ রিকুয়েস্ট পাঠানো হচ্ছে...")
        res = requests.post(url, json=payload, timeout=5)
        if res.status_code == 200:
            print("[+] রিকুয়েস্ট সফলভাবে পাঠানো হয়েছে!")
            # Optional: টার্গেটের রেসপন্স দেখার জন্য
            print("রেসপন্স:", res.text)
        else:
            print(f"[!] রিকুয়েস্ট ব্যর্থ হয়েছে, Status Code: {res.status_code}")
    except Exception as e:
        print(f"[!] রিকুয়েস্ট পাঠাতে ব্যর্থ: {e}")

def main_menu():
    while True:
        print("\n============================")
        print(" Termux Control Tool (শিক্ষামূলক)")
        print("============================")
        print("1. টার্গেট ডিভাইস স্ক্যান করুন (nmap)")
        print("2. টার্গেট ডিভাইসে ইনস্টল রিকুয়েস্ট পাঠান")
        print("3. Exit")
        choice = input("আপনার পছন্দ (1-3): ")

        if choice == "1":
            scan_network()
            time.sleep(2)
        elif choice == "2":
            target_ip = input("টার্গেট ডিভাইসের IP দিন: ")
            send_install_request(target_ip)
            time.sleep(2)
        elif choice == "3":
            print("Exiting...")
            break
        else:
            print("ভুল পছন্দ, পুনরায় চেষ্টা করুন।")

if __name__ == "__main__":
    main_menu()
```

---

### **কীভাবে চালাবেন:**

1. **nmap ইনস্টল করা:**  
   Termux-এ যদি nmap ইনস্টল না থাকে, তাহলে চালান:  
   ```bash
   pkg install nmap
   ```

2. **Python ও requests লাইব্রেরি ইনস্টল:**  
   ```bash
   pkg install python
   pip install requests
   ```

3. **উপরের কোডটি ফাইল হিসেবে সংরক্ষণ করুন**, যেমন `termux_tool.py`।  
4. **টুল চালান:**  
   ```bash
   python termux_tool.py
   ```

---

### **Target Device (APK) সংক্রান্ত নোট:**  
এই টুলসটি ধরে নিচ্ছে যে টার্গেট ডিভাইসে একটি APK ইনস্টল করা থাকবে যা একটি background HTTP service চালু রাখবে।  
এই সার্ভিসটি `/install_prompt` এ POST রিকুয়েস্ট গ্রহণ করলে একটি পপআপ দেখাবে এবং যদি ইউজার Yes নির্বাচন করে, তাহলে `app_link` ফিল্ডে থাকা URL খুলে ইনস্টলেশন শুরু করবে।  
আপনার APK-তে এমন ফাংশনালিটি থাকতে হবে, যা টার্গেটের অনুমতি ভিত্তিক কাজ করবে।

---

**দ্রষ্টব্য:** এই টুলসটি কেবল শিক্ষামূলক পরিবেশে ব্যবহার করুন। অননুমোদিতভাবে অন্য কারো ডিভাইসে ইনস্টলেশন বা ডেটা এক্সেস করা আইনি ও নৈতিক উভয় দিক থেকেই গ্রহণযোগ্য নয়।
