# Digital Forensics Investigation Lab Using Autopsy

**CSN 150 | Spring 2026**
**Student:** Kimberly Martinez
**Professor:** Reed-Sanchez

---

## What This Project Is

For my final project I used **Autopsy**, **FTK Imager**, and several specialist forensic tools to investigate a disk image from a suspected insider threat. The image came from the **Hunter** challenge on CyberDefenders — a free blue team forensics lab.

The investigation uncovered evidence of network scanning, data exfiltration, Skype communication with an external attacker, file shredding, and use of Tor Browser to hide activity. All 30 lab questions were answered using a combination of the tools listed below.

---

## Primary Tools

| Tool | Purpose | Download |
|------|---------|----------|
| Autopsy 4.21.0 | Main forensic analysis tool | https://github.com/sleuthkit/autopsy/releases/tag/autopsy-4.21.0 |
| FTK Imager | Opens the AD1 disk image and extracts registry hives | https://www.exterro.com/thank-you-digital-forensics |
| CyberDefenders | Source of the Hunter lab image | https://cyberdefenders.org |

> ⚠️ **Use Autopsy 4.21.0 — not 4.23.0.** The latest version freezes on Windows during startup. See the Troubleshooting section for details.

---

## Additional Investigation Tools

These tools were required to answer questions that Autopsy alone could not address, particularly around registry analysis, prefetch files, Skype databases, and jump lists.

| Tool | What It Was Used For | Key Findings |
|------|----------------------|--------------|
| Registry Explorer | Analyzed SYSTEM, SOFTWARE, and SAM registry hives | Computer name, IP address, OS version, timezone, SID |
| RegRipper | Automated registry hive analysis | Login count, last login time, user account details |
| DCode | Decoded epoch timestamps from the registry | DHCP LeaseObtainedTime converted to readable UTC |
| PECmd (Eric Zimmerman) | Parsed Windows Prefetch files | Zenmap last run time, BCWipe executed 5 times, 174 prefetch files total |
| Skyperious | Analyzed the Skype main.db database | Contact linux-rul3z, TeamViewer agreed on, Gmail address recovered |
| DB Browser for SQLite | Alternative Skype database analysis | Cross-verified Skype conversation data |
| SysTools PST Viewer | Opened Outlook PST backup file | Email with Pictures.7z attachment sent to attacker |
| Shell Bag Explorer | Read USRCLASS.DAT shell bag artifacts | Exfil staging folder path: C:\Users\Hunter\Pictures\Exfil |
| Jump List Explorer (Eric Zimmerman) | Analyzed AutomaticDestinations jump lists | Tor Browser path confirmed via AppID aa28770954eaeaaa |

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

### Step 2 — Extract Registry Hives Using FTK Imager
Many questions require registry analysis. While FTK Imager is loaded, extract these hives for use in Registry Explorer and RegRipper:

- `C:\Windows\System32\config\SYSTEM` → computer name, IP, timezone, USB devices
- `C:\Windows\System32\config\SOFTWARE` → OS version
- `C:\Windows\System32\config\SAM` → user SID, login count, last login
- `C:\Windows\System32\config\RegBack\SAM` → backup SAM hive if needed

Right-click each file → **Export File** → save to your desktop for analysis.

### Step 3 — Create a case in Autopsy
1. Open Autopsy → click New Case
2. Case Name: `Hunter-Forensics-Lab`
3. Click Next → enter your name → Finish

![Autopsy welcome screen]

<img width="1042" height="663" alt="Screenshot 2026-05-19 160655" src="https://github.com/user-attachments/assets/67e643b8-5abd-48a1-ad87-ec0ec50f39fd" />

### Step 4 — Add the exported folder as a data source
1. Select **Logical Files** as the data source type
2. Browse to your `Hunter-Exported/[root]` folder
3. Click Next

### Step 5 — Select ingest modules
Only enable these five:

- Recent Activity
- Hash Lookup
- File Type Identification
- Extension Mismatch Detector
- Keyword Search

![Ingest modules]

<img width="1028" height="650" alt="Screenshot 2026-05-19 162004" src="https://github.com/user-attachments/assets/9cf7ec59-901d-4fc9-8470-0cfdfb5d523d" />

### Step 6 — Wait for processing and explore results
Once ingest finishes, the left panel will populate with all the artifacts.

![Autopsy results loaded]

<img width="1148" height="731" alt="Screenshot 2026-05-20 194026" src="https://github.com/user-attachments/assets/3f1e1cac-52f9-46e4-bb8b-d08cc016e2be" />

### Step 7 — Analyze Registry Hives in Registry Explorer and RegRipper
Open each exported hive in Registry Explorer and navigate to the following paths:

- **Computer name:** `ControlSet001\Control\ComputerName\ComputerName`
- **IP address:** `SYSTEM\CurrentControlSet\Services\Tcpip\Parameters\Interfaces` → DhcpIPAddress
- **DHCP lease time:** Same path → LeaseObtainedTime (epoch) → decode with DCode
- **Timezone:** `SYSTEM\CurrentControlSet\Control\TimeZoneInformation`
- **OS version:** `SOFTWARE\Microsoft\Windows NT\CurrentVersion`
- **SID:** Load SAM hive → `SAM\SAM\Domains\Account\Aliases\Members`
- **Login count / last login:** Run RegRipper against the SAM hive

### Step 8 — Analyze Prefetch Files with PECmd
1. In FTK Imager, navigate to `C:\Windows\Prefetch` and export all `.pf` files
2. Run PECmd against the exported folder
3. Look for ZENMAP.EXE and BCWIPE.EXE entries for execution counts and timestamps

### Step 9 — Analyze Skype Database with Skyperious
1. In FTK Imager, navigate to `C:\Users\Hunter\AppData\Roaming\Skype\`
2. Export the `main.db` file
3. Open it in Skyperious to view conversation history, contacts, and recovered email addresses

### Step 10 — Analyze Email with SysTools PST Viewer
1. In FTK Imager, navigate to `C:\Users\Hunter\AppData\Roaming\Microsoft\Outlook\` or the Dropbox folder
2. Export `backup.pst`
3. Open in SysTools PST Viewer to find sent emails and attachments

### Step 11 — Analyze Shell Bags and Jump Lists
1. Export `USRCLASS.DAT` from `C:\Users\Hunter\AppData\Local\Microsoft\Windows\`
2. Open in Shell Bag Explorer to find folder navigation history
3. Export jump list files from `C:\Users\Hunter\AppData\Roaming\Microsoft\Windows\Recent\AutomaticDestinations\`
4. Open in Jump List Explorer and search for AppID `aa28770954eaeaaa`

---

## What I Found — All 30 Questions Answered

| # | Question | Answer | Tool Used |
|---|----------|--------|-----------|
| 1 | Computer name | 4ORENSICS | Registry Explorer (SYSTEM hive) |
| 2 | Computer IP | 10.0.2.15 | Registry Explorer (SYSTEM hive) |
| 3 | DHCP LeaseObtainedTime | 21/06/2016 02:24:12 UTC | RegRipper / DCode |
| 4 | Computer SID | S-1-5-21-2489440558-2754304563-710705792 | Registry Explorer (SAM hive) |
| 5 | OS version | 8.1 (Windows 8.1 Enterprise) | Registry Explorer (SOFTWARE hive) |
| 6 | Timezone | UTC-07:00 | Registry Explorer (SYSTEM hive) |
| 7 | Login count | 3 | RegRipper (SAM hive) |
| 8 | Last login time | 2016-06-21 01:42 | RegRipper (SAM hive) |
| 9 | Network scanner & last used | zenmap.exe, 2016-06-21 12:08:13 UTC | PECmd (Prefetch files) |
| 10 | Port scan end time | Tue Jun 21 05:12:09 2016 | nmapscan.xml on Hunter's Desktop |
| 11 | Ports scanned | 1000 | nmapscan.xml on Hunter's Desktop |
| 12 | Open ports | 22, 80, 9929, 31337 | nmapscan.xml on Hunter's Desktop |
| 13 | Scanner version | 7.12 | nmapscan.xml on Hunter's Desktop |
| 14 | Skype username (other party) | linux-rul3z | Skyperious (main.db) |
| 15 | Agreed exfiltration app | TeamViewer | Skyperious (main.db) |
| 16 | Suspect Gmail | ehptmsgs@gmail.com | Skyperious |
| 17 | Deleted diagram filename | home-network-design-networking-for-a-single-family-home-case-house-arkko-1433-x-792.jpg | SysTools PST Viewer (backup.pst) |
| 18 | PDF in Documents | Ryan_VanAntwerp_thesis.pdf | Autopsy — Recent Documents |
| 19 | Disk encryption app | Crypto Swap | FTK Imager (BCWipe uninstall log) |
| 20 | USB serial numbers | 07B20C03C80830A9, AAI6UXDKZDV8E9OU | Registry Explorer (USBSTOR) |
| 21 | File shredder app | Jetico BCWipe | Autopsy — Installed Programs |
| 22 | Prefetch files found | 174 | PECmd |
| 23 | Times shredder executed | 5 | PECmd (BCWIPE.EXE prefetch) |
| 24 | Last Zenmap execution | 06/21/2016 12:08:13 PM | PECmd (ZENMAP.EXE prefetch) |
| 25 | Burp Suite JAR path | C:\Users\Hunter\Downloads\burpsuite_free_v1.7.03.jar | Autopsy — Recent Documents |
| 26 | Email attachment name | Pictures.7z | SysTools PST Viewer (backup.pst) |
| 27 | Exfil folder path | C:\Users\Hunter\Pictures\Exfil | Shell Bag Explorer (USRCLASS.DAT) |
| 28 | Deleted JPG (1920x1200) | ws_Small_cute_kitty_1920x1200.jpg | Autopsy — Deleted Files |
| 29 | Jump lists directory | AutomaticDestinations | FTK Imager (file path navigation) |
| 30 | Tor Browser path | C:\Users\Hunter\Desktop\Tor Browser\Browser\firefox.exe | Jump List Explorer (AppID aa28770954eaeaaa) |

---

## Investigation Screenshots

![OS Accounts]

<img width="1151" height="728" alt="Screenshot 2026-05-20 194210" src="https://github.com/user-attachments/assets/7a34d511-f23d-4af9-9c62-042be9e64ee1" />

*OS Accounts — suspect user Hunter, 3 logins, last login 2016-06-20*

![OS Information]

<img width="1151" height="733" alt="Screenshot 2026-05-20 194125" src="https://github.com/user-attachments/assets/18999e89-0108-4150-9882-e52d76895120" />

*Operating System Information — computer name 4ORENSICS, Windows 8.1 Enterprise*

![Recent Documents]

<img width="1149" height="727" alt="Recent Documents" src="https://github.com/user-attachments/assets/abe912b8-0626-455b-9ecd-a13a16979b47" />

*Recent Documents — Burp Suite JAR, exfiltration diagram, and staged files*

![Web History]

<img width="1147" height="731" alt="Web History" src="https://github.com/user-attachments/assets/610e1fdc-d77b-4b5a-98ee-8e534d1e65ad" />

*Web History — Skype session and local exfil file access*

![Installed Programs]

<img width="1151" height="734" alt="Installed Programs" src="https://github.com/user-attachments/assets/be71e837-5f34-410e-8eed-dabac5362ada" />

*Installed Programs — BCWipe, Crypto Swap, Zenmap 7.12*

![Run Programs]

<img width="1151" height="730" alt="Run Program" src="https://github.com/user-attachments/assets/bb8ed4b6-3805-4b16-b97a-8df055313668" />

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

**Registry hives appear dirty or incomplete in Registry Explorer**
This happens when the image was taken while the system was still running. The registry remains in its last committed state — this is normal and does not prevent analysis.

---

## What I Learned

- Forensics is about telling a story — every artifact (shellbags, prefetch, jump lists, web history) helped piece together a complete picture of what the suspect did and when
- No single tool answers everything — Autopsy, FTK Imager, Registry Explorer, PECmd, Skyperious, and others each played a specific role in the investigation
- Deleted files are not truly gone — the suspect deleted files and ran a file shredder 5 times, but evidence was still recovered through multiple artifact types
- The same conclusion can be reached through multiple sources, which makes the evidence stronger and harder to dispute
- Registry hives, Prefetch files, Skype databases, PST emails, Shell Bags, and Jump Lists all independently confirmed the same insider threat activity

---

*CSN 150 | Spring 2026 | Kimberly Martinez*
