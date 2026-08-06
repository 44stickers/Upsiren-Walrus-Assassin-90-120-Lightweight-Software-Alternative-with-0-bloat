# Upsiren-Walrus-Assassin-90-120-Lightweight-Software-Alternative-with-0-bloat

A bloat-free Python controller for **Upsiren Walrus Assassin 90 / 120** CPU coolers.

The official Upsiren software is heavy, resource-intensive, and prone to bugs. This ultra-lightweight alternative piggybacks directly off **HWiNFO64**'s registry output and pushes live CPU temperature readings to your cooler's display via USB header.

### 🚀 Highlights

* **Zero Performance Impact:** ~0% CPU usage and under 20 MB RAM usage combined.
* **No Bloatware:** Completely bypasses the official software.
* **Silent Autostart:** Runs hidden in the background on Windows startup without console popups.

---

## ⚠️ Prerequisite: Uninstall Official Software

> **Important:** Completely uninstall all official Upsiren software before starting. Running both simultaneously will cause USB interface conflicts.

---

## 🛠️ Setup Instructions

### Step 1: Configure HWiNFO64

1. Download and run the Windows installer from [HWiNFO.com](https://www.hwinfo.com/download/).
2. Open **HWiNFO64**, go to **General Settings**, and copy these settings:
   <img width="680" height="400" alt="image" src="https://github.com/user-attachments/assets/88a8d8d9-1bfa-4f6a-ade3-a83a2d07babf" />

4. Open the **Sensors** tab and click the **Gear Icon** ⚙️ in the bottom right corner.
5. Go to the **HWiNFO Gadgets** tab across the top:
6. Scroll down to **CPU Die (average)** (or your CPU's main temperature metric) and copy whatever you see here:
   <img width="776" height="638" alt="image" src="https://github.com/user-attachments/assets/d6a9ce70-a193-4d16-8375-4abe60e7e10d" />

7. You can close out of HwINFO now (it looks like a heavy piece of software if you are new but trust me its very light)
---

### Step 2: Set Up Python & Dependencies

1. Install **Python 3.12** (or newer) from the Microsoft Store
2. Open **Command Prompt as Administrator** and install `hidapi`:
```bash
pip install hidapi

```



---

### Step 3: Create the Python Controller

1. Open **Notepad** and paste the following code:

```python
import hid
import time
import sys
import winreg

# Upsiren Walrus Assassin Identifiers
VENDOR_ID = 0x5131
PRODUCT_ID = 0x2007

def get_true_hwinfo_temp():
    try:
        # Open HWiNFO's flat VSB registry folder directly
        path = r"SOFTWARE\HWiNFO64\VSB"
        key = winreg.OpenKey(winreg.HKEY_CURRENT_USER, path)
        
        # Query how many individual data values are sitting inside the key folder
        _, num_values, _ = winreg.QueryInfoKey(key)
        
        # Find index matching the CPU Die Average label
        target_index = None
        for i in range(num_values):
            name, data, _ = winreg.EnumValue(key, i)
            if name.startswith("Label") and "cpu die (average)" in str(data).lower():
                target_index = name.replace("Label", "").strip()
                break
                
        if target_index is not None:
            try:
                # Query raw value directly using the matching text slot suffix
                raw_value, _ = winreg.QueryValueEx(key, f"ValueRaw{target_index}")
                
                # Convert raw string to rounded integer
                temp = int(round(float(str(raw_value).strip())))
                if 20 <= temp <= 95:
                    winreg.CloseKey(key)
                    return temp
            except:
                pass
                
        winreg.CloseKey(key)
    except:
        pass
    return None

def main():
    print("Upsiren Walrus Assassin 90/120 - Flat Registry Controller")
    print("---------------------------------------------------------")
    
    try:
        device = hid.device()
        device.open(VENDOR_ID, PRODUCT_ID)
        print("Connected to hardware header successfully!")
    except Exception as e:
        print(f"Error opening USB device: {e}")
        sys.exit(1)

    print("Sending live direct registry data... Press Ctrl+C to stop.")
    
    try:
        while True:
            temp = get_true_hwinfo_temp()
            
            if temp is not None:
                print(f"Current HWiNFO Registry Reading: {temp} °C")
                
                # 64-byte HID report construction
                data_packet = [0x00] * 64
                data_packet[1] = temp  
                final_packet = [0x00] + data_packet
                
                try:
                    device.write(final_packet)
                except:
                    try:
                        device.open(VENDOR_ID, PRODUCT_ID)
                    except:
                        pass
            else:
                print("Searching for active 'CPU Die (average)' value inside VSB block...")
                
            time.sleep(2) # Updates every 2 seconds
            
    except KeyboardInterrupt:
        print("\nStopping controller cleanly.")
        device.close()

if __name__ == "__main__":
    main()

```

2. Save the file as **`upsiren_control.py`** (Change *Save as type* to **All Files**).

---

### Step 4: Configure Silent Autostart

1. Open a new **Notepad** document and paste the following script:

```vbs
Set WshShell = CreateObject("WScript.Shell")
userProfile = WshShell.ExpandEnvironmentStrings("%USERPROFILE%")

' Resolves path automatically regardless of user profile directory
scriptPath = userProfile & "\Downloads\SOFTWARE\upsiren_control.py"

' Run hidden (0) without waiting for exit (False)
WshShell.Run "python """ & scriptPath & """", 0, False

```

> *(Note: Make sure to adjust `\Downloads\SOFTWARE\` in line 5 if you saved `upsiren_control.py` in a different subfolder under your user directory).*

2. Save this file as **`silent_start.vbs`** in the same folder as `upsiren_control.py`.
3. Right-click **`silent_start.vbs`** and select **Show more options → Create shortcut**.
4. Press **Win + R**, type `shell:startup`, and hit **Enter**.
5. Move the **`silent_start.vbs - Shortcut`** file into the Startup folder.

---

🎉 **You're all set!** On your next reboot, HWiNFO64 and this script will run silently in the background, keeping your cooler display updated continuously.
