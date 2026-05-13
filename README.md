# Dastardly Deed with Disappeared Data Complex 2

## Overview

This walkthrough covers how to decrypt a password-protected ZIP file using steganography, multi-layer encoding, AES decryption, and a known-plaintext attack with PkCrack. This builds on the methods from Complex 1 with additional steps involving a hidden message inside a PNG image and a secret key encoded in Base64 and Morse code.

> **Please be advised that some of your commands will be different based on where your files are located. This is only the steps on what I did, giving you an idea on what to do.**

---

## Step 1: Download and Install Java on Windows

DIIT is a Java-based steganography tool and Java is not pre-installed. Download and install Java for Windows from https://www.java.com and verify the installation:

```powershell
java -version
```

---

## Step 2: Extract the Hidden Message from the PNG Using DIIT

The PNG image on the flash drive contains a hidden message embedded using steganography. Standard tools like `stegdetect` will not work on PNG files — they are JPEG only. Use the Digital Invisible Ink Toolkit (DIIT) instead.

Launch DIIT directly with Java since the batch launcher will fail if Java is not in the system PATH:

```powershell
java -jar "E:\Ione\tools\steganography\diit dirt\diit-1.5.jar"
```

In the DIIT GUI:
1. Click the **Decode** tab
2. Click **Get Image** → select `E:\Ione\dino.png`
3. Click **Set Message** → set output to `C:\Users\playerone\Desktop\hidden.txt`
4. Change the algorithm from **BattleSteg** to **BlindHide**
5. Click **Go** → "Success! Message was retrieved."

> **Note:** The default BattleSteg algorithm will return "Could not find a message." BlindHide is the correct algorithm for this challenge.

The output file will contain a hash string:
```
141CB0BD4EB6B9EDC57188F3B7859A05D9744C64FCAA91A312A60B4135E0DD8B
```

---

## Step 3: Use the Hash to Unlock the Password-Protected ZIP

The extracted hash (in lowercase) is the password for `safe.zip`. Use 7-Zip to extract it:

```powershell
& "C:\Program Files\7-Zip\7z.exe" e E:\Ione\safe.zip "-p141cb0bd4eb6b9edc57188f3b7859a05d9744c64fcaa91a312a60b4135e0dd8b" -o"E:\Ione\safe_extracted"
```

> **Note:** The hash must be **lowercase**. The uppercase version will return "Wrong password."

This extracts:
- `astral.jpg.aes` — AES encrypted image
- `secretkey.txt` — encoded key file

---

## Step 4: Decode the Secret Key

The `secretkey.txt` file is encoded in multiple layers.

**Layer 1 — Base64:** Decode it in PowerShell:

```powershell
[System.Text.Encoding]::UTF8.GetString([System.Convert]::FromBase64String((Get-Content E:\Ione\safe_extracted\secretkey.txt)))
```

The output will be Morse code.

**Layer 2 — Morse Code:** Use the online cipher tool at https://rumkin.com/tools/cipher/morse-code/ to decode the Morse code into the final AES key.

Final AES key:
```
2D80EC37DE221616C7725166418AF36CB5D5F6A30ECECF093333C04C6E2ECFED
```

---

## Step 5: Decrypt the AES-Encrypted File Using AESCrypt

```powershell
E:\Ione\tools\AESCrypt\AESCrypt_console_v309_64\aescrypt.exe -d -p 2D80EC37DE221616C7725166418AF36CB5D5F6A30ECECF093333C04C6E2ECFED E:\Ione\safe_extracted\astral.jpg.aes
```

This produces `astral.jpg` in `E:\Ione\safe_extracted\`.

---

## Step 6: Download and Install FileZilla on Windows

FileZilla is used to transfer files between the Windows machine and Kali. Download and install it from https://filezilla-project.org and open the application once installed.

---

## Step 7: Transfer Files to Kali

Use FileZilla to transfer `astral.jpg` and `exfiltratroll.zip` to `~/Desktop/Ione/` on Kali:

- **Host:** Kali IP
- **Username:** playerone
- **Password:** password123
- **Port:** 22

---

## Step 8: Install PkCrack on Kali

Clone the PkCrack repository and make the binary executable:

```bash
cd ~
git clone https://github.com/keyunluo/pkcrack
chmod +x ~/pkcrack/bin/pkcrack
```

> **Note:** The `chmod +x` step is required. Without it you will get a `permission denied` error when trying to run PkCrack.

---

## Step 9: Create a ZIP of astral.jpg Using 7-Zip

```bash
cd ~/Desktop/Ione
7z a astral.zip astral.jpg
```

> **Note:** Must use 7-Zip specifically. The Windows built-in ZIP utility uses a different compression method and will cause PkCrack to fail.

---

## Step 10: Run PkCrack (Known-Plaintext Attack)

```bash
~/pkcrack/bin/pkcrack -C ~/Desktop/Ione/exfiltratroll.zip -c astral.jpg -P ~/Desktop/Ione/astral.zip -p astral.jpg -d ~/Desktop/Ione/cracked.zip -a
```

**Flag breakdown:**
| Flag | Description |
|------|-------------|
| `-C` | The encrypted ZIP you want to crack |
| `-c` | The filename inside the encrypted ZIP to use as the known plaintext |
| `-P` | Your ZIP containing the plaintext file |
| `-p` | The filename inside your plaintext ZIP |
| `-d` | Output path for the decrypted ZIP |
| `-a` | Search all files in the encrypted ZIP for a match |

If successful you will see output like:

```
Decrypting 2EWX83.txt ... OK!
Decrypting astral.jpg ... OK!
Decrypting fog.jpg ... OK!
Decrypting prince.jpg ... OK!
Finished on Tue May 12 19:57:05 2026
```

---

## Step 11: Extract the Decrypted ZIP

```bash
cd ~/Desktop/Ione
unzip cracked.zip
```

The decrypted files are now accessible.

---

## Key Takeaways

- `stegdetect` only works on JPEG files — use DIIT for PNG steganography
- DIIT's default algorithm (BattleSteg) will fail — try **BlindHide** instead
- The hash extracted from steganography is the ZIP password but must be used in **lowercase**
- The secret key is layered: Base64 → Morse code → plaintext AES key
- AESCrypt on Windows decrypts `.aes` files once you have the key
- PkCrack performs a **known-plaintext attack** — it needs one file from inside the encrypted ZIP in plaintext form
- The plaintext ZIP **must** be created with 7-Zip to match the compression method used by the attacker
- If PkCrack returns `permission denied`, run `chmod +x` on the binary before executing
- FileZilla is a reliable way to transfer files between Windows and Kali
