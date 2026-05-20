# Digital Forensics Investigation Lab Using Autopsy

**CSN 150 | Spring 2026**
**Student:** Kimberly Martinez
**Professor:** Reed-Sanchez

---

## What This Project Is

For my final project I used **Autopsy** and **FTK Imager** to investigate a forensic disk image from a suspected insider threat. The image came from the **Hunter** challenge on CyberDefenders — a free blue team forensics lab.

The investigation uncovered evidence of network scanning, data exfiltration, Skype communication with an external attacker, file shredding, and use of Tor Browser to hide activity.

---

## Tools Used

| Tool | Purpose | Download |
|------|---------|----------|
| Autopsy 4.21.0 | Main forensic analysis tool | https://github.com/sleuthkit/autopsy/releases/tag/autopsy-4.21.0 |
| FTK Imager | Used to open the AD1 disk image | https://www.exterro.com/thank-you-digital-forensics |
| CyberDefenders | Source of the Hunter lab image | https://cyberdefenders.org |

> ⚠️ **Use Autopsy 4.21.0 — not 4.23.0.** The latest version freezes on Windows during startup. See the Troubleshooting section for details.

---

## How to Set This Up

### 1. Install Autopsy 4.21.0
- Download `autopsy-4.21.0-64bit.msi` from the link above
- Right-click → Run as Administrator
- Use default installation settings

### 2. Install FTK Imager
- Download and install from the link above
- Run as Administrator

### 3. Get the Hunter Lab Image
- Create a free account at cyberdefenders.org
- Search for the **Hunter** lab and download the zip file
- The zip is password protected — use the password: `cyberdefenders.org`
- After extracting you will have two files:
  - `Hunter` — the actual disk image (621,327 KB) — this is what you need
  - `Hunter.ad1` — a small text file (3 KB) — ignore this one

---

## How I Ran the Investigation

### Step 1 — Open the image in FTK Imager
Autopsy cannot open AD1 files directly, so I used FTK Imager to extract the contents first.

1. Open FTK Imager → File → Add Evidence Item → Image File
2. Browse to and select the Hunter file (621,327 KB)
3. Expand: `Hunter.ad1 → Custom Content Image → c16-Hunter:NONAME [NTFS] → [root]`
4. Right-click `[root]` → Export Files → save to a folder called `Hunter-Exported` on your Desktop
5. Wait for export to finish (662 folders, 6,660 files exported)

![FTK Imager loaded] + [File system expanded]

<img width="1150" height="733" alt="Screenshot 2026-05-20 193904" src="https://github.com/user-attachments/assets/108f5682-d687-4c83-b126-3710906e79be" />


![Export complete]

<img width="412" height="237" alt="Screenshot 2026-05-19 171804" src="https://github.com/user-attachments/assets/b641dc65-e521-4f7a-8582-3fadbccc2586" />

### Step 2 — Create a case in Autopsy
1. Open Autopsy → click New Case
2. Case Name: `Hunter-Forensics-Lab`
3. Click Next → enter your name → Finish

![Autopsy welcome screen]

<img width="1042" height="663" alt="Screenshot 2026-05-19 160655" src="https://github.com/user-attachments/assets/67e643b8-5abd-48a1-ad87-ec0ec50f39fd" />

### Step 3 — Add the exported folder as a data source
1. Select **Logical Files** as the data source type
2. Browse to your `Hunter-Exported/[root]` folder
3. Click Next

### Step 4 — Select ingest modules
Only enable these five:

- Recent Activity
- Hash Lookup
- File Type Identification
- Extension Mismatch Detector
- Keyword Search

![Ingest modules] 

<img width="1028" height="650" alt="Screenshot 2026-05-19 162004" src="https://github.com/user-attachments/assets/9cf7ec59-901d-4fc9-8470-0cfdfb5d523d" />


### Step 5 — Wait for processing and explore results
Once ingest finishes, the left panel will populate with all the artifacts.

![Autopsy results loaded]

<img width="1148" height="731" alt="Screenshot 2026-05-20 194026" src="https://github.com/user-attachments/assets/3f1e1cac-52f9-46e4-bb8b-d08cc016e2be" />


---

## What I Found — All 30 Questions Answered

| # | Question | Answer | Found In |
|---|----------|--------|----------|
| 1 | Computer name | 4ORENSICS | OS Information |
| 2 | Computer IP | 10.0.2.15 | OS Information |
| 3 | DHCP LeaseObtainedTime | 21/06/2016 02:24:12 UTC | OS Information |
| 4 | Computer SID | S-1-5-21-2489440558-2754304563-710705792 | OS Accounts |
| 5 | OS version | 8.1 (Windows 8.1 Enterprise) | OS Information |
| 6 | Timezone | UTC-07:00 | OS Information |
| 7 | Login count | 3 | OS Accounts → Hunter row |
| 8 | Last login time | 2016-06-21 01:42 | OS Accounts → Hunter row |
| 9 | Network scanner & last used | zenmap.exe, 2016-06-21 12:08:13 UTC | Run Programs |
| 10 | Port scan end time | Tue Jun 21 05:12:09 2016 | Zenmap scan log |
| 11 | Ports scanned | 1000 | Zenmap scan log |
| 12 | Open ports | 22, 80, 9929, 31337 | Zenmap scan log |
| 13 | Scanner version | 7.12 | Installed Programs |
| 14 | Skype username (other party) | linux-rul3z | Skype database |
| 15 | Agreed exfiltration app | TeamViewer | Skype conversation |
| 16 | Suspect Gmail | ehptmsgs@gmail.com | Email Addresses |
| 17 | Deleted diagram filename | home-network-design-networking-for-a-single-family-home-case-house-arkko-1433-x-792.jpg | Deleted Files |
| 18 | PDF in Documents | Ryan_VanAntwerp_thesis.pdf | Recent Documents |
| 19 | Disk encryption app | Crypto Swap | Installed Programs |
| 20 | USB serial numbers | 07B20C03C80830A9, AAI6UXDKZDV8E9OU | USB Device Attached |
| 21 | File shredder app | Jetico BCWipe | Installed Programs |
| 22 | Prefetch files found | 174 | Run Programs |
| 23 | Times shredder executed | 5 | Run Programs → BCWIPE entries |
| 24 | Last Zenmap execution | 06/21/2016 12:08:13 PM | Run Programs → ZENMAP.EXE prefetch |
| 25 | Burp Suite JAR path | C:\Users\Hunter\Downloads\burpsuite_free_v1.7.03.jar | Recent Documents |
| 26 | Email attachment name | Pictures.7z | Email artifacts |
| 27 | Exfil folder path | C:\Users\Hunter\Pictures\Exfil | Shell Bags |
| 28 | Deleted JPG (1920x1200) | ws_Small_cute_kitty_1920x1200.jpg | Deleted Files |
| 29 | Jump lists directory | AutomaticDestinations | AppData\Roaming\Microsoft\Windows\Recent |
| 30 | Tor Browser path | C:\Users\Hunter\Desktop\Tor Browser\Browser\firefox.exe | Jump Lists |

---

## Investigation Screenshots

![OS Accounts]

<img width="1151" height="728" alt="Screenshot 2026-05-20 194210" src="https://github.com/user-attachments/assets/7a34d511-f23d-4af9-9c62-042be9e64ee1" />

*OS Accounts — suspect user Hunter, 3 logins, last login 2016-06-20*

![OS Information]

<img width="1151" height="733" alt="Screenshot 2026-05-20 194125" src="https://github.com/user-attachments/assets/18999e89-0108-4150-9882-e52d76895120" />

*Operating System Information — computer name 4ORENSICS, Windows 8.1 Enterprise*

![Recent Documents]<img width="770" height="730" alt="Screenshot 2026-05-19 172134" src="https://github.com/user-attachments/assets/da24069f-fb8d-47ee-8b48-9e3667b18d01" />
*Recent Documents — Burp Suite JAR, exfiltration diagram, and staged files*

![USB Devices]<img width="770" height="718" alt="Screenshot 2026-05-19 172147" src="https://github.com/user-attachments/assets/176caa7d-87c0-40f7-8427-b422a13c1fe5" />
*USB Device Attached — two storage drives identified*

![Shell Bags]<img width="767" height="731" alt="Screenshot 2026-05-19 172159" src="https://github.com/user-attachments/assets/62999271-a42d-4d24-82bb-3999c82394dc" />
*Shell Bags — suspect browsed to the Exfil staging folder*

![Web History]<img width="770" height="728" alt="Screenshot 2026-05-19 172323" src="https://github.com/user-attachments/assets/8e42c720-d7b6-4d02-92c3-6db6fad2311b" />
*Web History — Skype session and local exfil file access*

![Installed Programs]<img width="772" height="730" alt="Screenshot 2026-05-19 172350" src="https://github.com/user-attachments/assets/b94e8e4e-d55c-4bda-87b2-7dba9e8b5bf2" />
*Installed Programs — BCWipe, Crypto Swap, Zenmap 7.12*

![Run Programs]<img width="771" height="731" alt="Screenshot 2026-05-19 172402" src="https://github.com/user-attachments/assets/ad89f8af-e040-4f32-a7b2-d6d6061c18a5" />
*Run Programs — 477 prefetch entries including BCWipe and Zenmap executions*

---

## Troubleshooting

**Autopsy 4.23.0 freezes on the splash screen**
The latest version has stability issues on Windows. Downgrade to 4.21.0 and ignore the update notification when it appears.

![Autopsy frozen]<img width="1151" height="733" alt="Screenshot 2026-05-19 143428" src="https://github.com/user-attachments/assets/70ed994f-ebd6-4a81-baf3-c9a5181e3b53" />

**Autopsy can't open AD1 files**
AD1 is a proprietary AccessData format. Use FTK Imager to export the contents first, then load the exported folder into Autopsy as Logical Files.

**The zip file is password protected**
Use the password `cyberdefenders.org` — this is the standard password for all CyberDefenders challenge downloads.

![Password protected]<img width="1029" height="275" alt="Screenshot 2026-05-19 170811" src="https://github.com/user-attachments/assets/73bf25fc-dcfe-48a5-8101-83817cccc7cd" />
![Properly extracted]<img width="765" height="226" alt="Screenshot 2026-05-19 171226" src="https://github.com/user-attachments/assets/9b0e35b5-7df8-4984-b695-2dbc6f2c6ea6" />

**Wrong file loaded into Autopsy**
Make sure you select the Hunter file that is 621,327 KB — not the Hunter.ad1 text file that is only 3 KB.

**Data Artifacts panel shows nothing**
The ingest modules may still be running. Wait for the progress bar at the bottom of Autopsy to finish before checking results.

---

## What I Learned

- Forensics is about telling a story — every artifact (shellbags, prefetch, jump lists, web history) helped piece together a complete picture of what the suspect did and when
- Not all forensic tools support all image formats — FTK Imager and Autopsy work well together as a pair
- Deleted files are not truly gone — the suspect deleted files and ran a file shredder, but Autopsy still recovered key evidence from the Recycle Bin
- The same conclusion can be reached through multiple artifact types, which makes the evidence stronger and harder to dispute

---

*CSN 150 | Spring 2026 | Kimberly Martinez*
