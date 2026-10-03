Here is a Python script that automates the hex replacement for the Sublime Text 4 (Build 4215) binary on macOS.
This script creates a backup of the original binary before modifying the bytes, searches for the specific target sequence, and replaces it.

```python
import os
# Paths for Sublime Text 4 on macOSAPP_PATH = "/Applications/Sublime Text.app/Contents/MacOS/sublime_text"BACKUP_PATH = APP_PATH + ".bak"
# Hex sequences for Build 4215 (Apple Silicon)TARGET_HEX = bytes.fromhex("09 31 40 39 4A 00 80 52 3F 01 00 71 E9 13 8A 1A")REPLACE_HEX = bytes.fromhex("29 00 80 52 09 31 00 39 E9 03 1F 2A 1F 20 03 D5")
def patch_sublime():
    if not os.path.exists(APP_PATH):
        print(f"[-] Error: Sublime Text binary not found at {APP_PATH}")
        print("    Please ensure Sublime Text is installed in your Applications folder.")
        return

    # Create a backup if it doesn't exist yet
    if not os.path.exists(BACKUP_PATH):
        print(f"[+] Creating backup at: {BACKUP_PATH}")
        try:
            with open(APP_PATH, 'rb') as src, open(BACKUP_PATH, 'wb') as dst:
                dst.write(src.read())
        except PermissionError:
            print("[-] Permission denied. Please run the script with 'sudo'.")
            return
    else:
        print("[*] Backup already exists. Proceeding...")

    # Read the binary data
    with open(APP_PATH, 'rb') as f:
        data = f.read()

    # Search and replace
    if TARGET_HEX in data:
        count = data.count(TARGET_HEX)
        print(f"[+] Found {count} match(es) for the target hex pattern.")
        
        patched_data = data.replace(TARGET_HEX, REPLACE_HEX)
        
        # Write the patched data back
        with open(APP_PATH, 'wb') as f:
            f.write(patched_data)
        print("[+] Binary patched successfully!")
        print("[!] Next step: You must re-sign the application to prevent macOS from crashing it.")
    elif REPLACE_HEX in data:
        print("[*] Binary appears to already be patched.")
    else:
        print("[-] Error: Target hex pattern not found. Verify you are using Build 4215.")
if __name__ == "__main__":
    patch_sublime()
```

## How to Run and Complete the Process

   1. Close Sublime Text completely.
   2. Save the code above to a file named patch.py.
   3. Open your Terminal and run the script with sudo privileges (required to modify files in the /Applications directory):
   
```bash
sudo python3 patch.py
```

   4. Important: Because the binary structure was altered, macOS gatekeeper will reject it as corrupted until you strip the attributes and force a self-signature. Run these two terminal commands immediately after running the script:

```bash
   sudo xattr -rc "/Applications/Sublime Text.app"
   sudo codesign --force --deep --sign - "/Applications/Sublime Text.app"
```
   
