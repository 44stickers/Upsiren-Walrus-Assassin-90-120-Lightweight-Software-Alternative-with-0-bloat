# Upsiren-Walrus-Assassin-90-120-Bloat-free-temp-tool
Those of you who own an Upsiren Walrus Assassin 90 / 120 cooler probably know how horrible the software is. I made a fix. Its an ultra light python script that piggy backs off of HwINFO's Cpu temp reading and pushes that number into the usb header the Cooler is plugged in to show it on the display.
# It takes only 5 minutes to setup and has 0% cpu usage and 20MB ram usage only

# WHAT TO DO
# STEP 1
FIRST DELETE ALL UPSIREN SOFTWARES YOU HAVE OR ELSE IT WILL CAUSE A CONFLICT
1) Download HwINFO https://www.hwinfo.com/download/ download the installer for windows
2) Open HwINFO and go to the general settings tab and make sure your settings look like this
<img width="696" height="424" alt="image" src="https://github.com/user-attachments/assets/4402b7ce-c05f-490b-b8ed-248c8ef0ca25" />

3) Open the sensors tab and click the gear icon in the bottom right
4) At the top bar go to "HwINFO Gadgets"
<img width="776" height="644" alt="image" src="https://github.com/user-attachments/assets/ae0fd47e-2901-411f-89a9-da7d0715d77a" />

5) Scroll down till you see "Cpu Die (average)" and enable these tick boxes
<img width="778" height="638" alt="image" src="https://github.com/user-attachments/assets/401e9f13-196c-4c2d-b09b-d0cb756978e4" />
Now HwINFO is configured, those of you who are new to HwINFO, its super light weight and wont cause you lags or stutters

# STEP 2
The python script will piggyback off of the Cpu Die avg reading it updates to the registry and pushes that number with correct formatting to the usb header the fan is plugged in
1) Download Python 3.12 from the microsoft store
2) Open command prompt as admin from the windows search box
3) Copy paste this "pip install hidapi" without the speech marks and press enter
4) Open Notepad and copy paste this code

import hid
import time
import sys
import winreg

# Upsiren Walrus Assassin 90 Identifiers
VENDOR_ID = 0x5131
PRODUCT_ID = 0x2007

def get_true_hwinfo_temp():
    try:
        # Open HWiNFO's flat VSB registry folder directly
        path = r"SOFTWARE\HWiNFO64\VSB"
        key = winreg.OpenKey(winreg.HKEY_CURRENT_USER, path)
        
        # Query how many individual data values are sitting inside the key folder
        _, num_values, _ = winreg.QueryInfoKey(key)
        
        # First loop: Find the index number associated with your CPU Die Average label
        target_index = None
        for i in range(num_values):
            name, data, _ = winreg.EnumValue(key, i)
            
            # If we find a Label name (like Label0, Label1) matching your processor sensor
            if name.startswith("Label") and "cpu die (average)" in str(data).lower():
                # Extract the tracking index number suffix from the value name string
                target_index = name.replace("Label", "").strip()
                break
                
        if target_index is not None:
            try:
                # Query the raw numeric value directly using the matching text slot suffix (e.g., ValueRaw0)
                raw_value, _ = winreg.QueryValueEx(key, f"ValueRaw{target_index}")
                
                # Convert the raw text string cleanly to a decimal and round to a whole integer
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
    print("Upsiren Walrus Assassin 90 - Flat Registry Controller")
    print("-----------------------------------------------------")
    
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
                print(f"Current HWiNFO Registry Reading: {temp} C")
                
                # THE ARRAY STRUCTURAL FIX:
                # 1. Initialize a clean 64-byte array of zeroes
                data_packet = [0x00] * 64
                
                # 2. Modify index position 1 directly without overwriting the list type
                data_packet[1] = temp  
                
                # 3. Concatenate two true list types together flawlessly
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
                
            time.sleep(2) # Updates comfortably every 2 seconds
            
    except KeyboardInterrupt:
        print("\nStopping controller cleanly.")
        device.close()

if __name__ == "__main__":
    main()



5) Save as "upsiren_control.py" without the speechmarks and select file type as "all" NOT TXT, save this in a file folder or something
6) Open another notepad and write this script in it but read the instruction i left for you in the script carefully

Set WshShell = CreateObject("WScript.Shell")
WshShell.Run "python ""THE FILE PATH WHERE YOUR FILE IS PUT IT HERE\upsiren_control.py""", 0, False

7) Save as "silent_start.vbs"
8) Right click it and make a shortcut
9) Press Win + R and type shell:startup and move that shortcut you just made in here
10) Now restart your pc and when you turn it on you will see the temperature on your cooler showing and the python script along with HwINFO running will only take 20MB ram maximum with no performance loss.
11) Nice
