# 🔍 Digital Forensics Investigation Lab Using Autopsy
**CSN 150 | Final Project | Spring 2026**
**Student:** Kimberly Martinez
**Professor:** Reed-Sanchez

---

## 📌 Project Overview

This project demonstrates a structured digital forensics investigation using **Autopsy**, a free and open-source forensic platform trusted by law enforcement agencies, incident responders, and cybersecurity professionals worldwide.

For this lab, I used the **Hunter** challenge from [CyberDefenders](https://cyberdefenders.org) — a beginner-friendly blue team CTF that provides a real forensic disk image to investigate. The goal was to analyze the image, recover artifacts, and answer key investigative questions using Autopsy's built-in analysis tools.

### What is Autopsy?
Autopsy is a graphical interface built on top of **The Sleuth Kit (TSK)**, a collection of command-line forensic tools. It allows investigators to:
- Examine disk images without altering the original evidence
- Recover deleted files and hidden data
- Analyze browser history, downloads, and user activity
- Search for keywords and flag known malicious files
- Build timelines of system events

### Why This Project Matters
In the real world, digital forensics is a critical part of incident response. When a system is compromised, attackers often try to delete files or clear logs to cover their tracks. Autopsy gives investigators the ability to recover that data and reconstruct exactly what happened — which can be essential for legal proceedings, insurance claims, and preventing future attacks.

---

## 🛠️ Tools & Resources

| Resource | Link |
|----------|------|
| Autopsy Official Site | https://www.sleuthkit.org/autopsy/ |
| Autopsy Documentation | https://www.sleuthkit.org/autopsy/docs.php |
| The Sleuth Kit GitHub | https://github.com/sleuthkit/autopsy |
| CyberDefenders (Hunter Lab) | https://cyberdefenders.org |
| BlueTeamLabs Online | https://blueteamlabs.online |
| Digital Corpora (Practice Images) | https://digitalcorpora.org |

---

## 💻 System Requirements

| Component | Minimum | Recommended |
|-----------|---------|-------------|
| OS | Windows 10 | Windows 10/11 64-bit |
| RAM | 8 GB | 16 GB |
| Storage | 50 GB free | 100 GB free (SSD preferred) |
| Java | Included in installer | — |

> **Note:** Autopsy is most stable on Windows. Running it on an SSD significantly reduces processing time during ingest.

---

## 📥 Installation Instructions

### Step 1 — Download Autopsy
1. Go to https://www.sleuthkit.org/autopsy/
2. Click **"Download"** and select the latest **Windows 64-bit** installer
3. The file will be named something like `autopsy-4.21.0-64bit.msi`

### Step 2 — Run the Installer
1. Double-click the `.msi` file to launch the installer
2. Accept the license agreement
3. Leave the default installation directory as-is
4. Click **Install** and wait for it to complete (this takes a few minutes)
5. Click **Finish** — do **not** launch Autopsy yet

### Step 3 — Get the Hunter Lab Disk Image
1. Go to [cyberdefenders.org](https://cyberdefenders.org) and create a free account
2. Navigate to **Labs** → filter by **Forensics** and **Easy**
3. Search for and open the **Hunter** lab
4. Click **Download** to get the challenge zip file
5. Once downloaded, **right-click → Extract All** to unzip
6. Inside you will find the disk image file (`.E01` or `.dd` format)

---

## ⚙️ Configuration & Setup

### Step 1 — Launch Autopsy & Create a New Case
1. Open Autopsy from your Start menu
2. Wait for the splash screen to finish loading
3. Click **"New Case"**
4. Fill in the following:
   - **Case Name:** `Hunter-Forensics-Lab`
   - **Base Directory:** choose a folder with plenty of storage
5. Click **Next**
6. Enter your name in the **Examiner** field
7. Click **Finish**

### Step 2 — Add the Disk Image as a Data Source
1. When prompted to add a data source, select **"Disk Image or VM File"**
2. Click **Next**
3. Click **Browse** and navigate to your unzipped Hunter lab image
4. Select the `.E01` or `.dd` file
5. Leave the time zone at default unless the lab specifies otherwise
6. Click **Next**

### Step 3 — Select Ingest Modules
When the ingest module screen appears, make sure the following are checked:

| Module | Purpose |
|--------|---------|
| ✅ File Type Identification | Identifies files by content, not just extension |
| ✅ Recent Activity | Recovers browser history, downloads, searches |
| ✅ Keyword Search | Allows searching for specific terms across the image |
| ✅ Hash Lookup | Flags known malicious files using hash databases |
| ✅ Extension Mismatch Detector | Catches files that have been renamed to hide their type |

Click **Finish** — Autopsy will begin processing. This typically takes **15–45 minutes** depending on image size and your hardware.

### Step 4 — Analyze the Results
Once ingest is complete, use the left panel to explore:

- **Data Sources** → browse the full file system
- **Views → File Types** → filter by images, documents, executables, etc.
- **Results → Extracted Content → Web History** → browser activity
- **Results → Extracted Content → Recent Documents** → recently opened files
- **Results → Keyword Hits** → anything flagged by your keyword search
- **Timeline** (Tools menu) → chronological view of all system events

### Step 5 — Generate a Report
1. Go to **Tools → Generate Report**
2. Select **HTML Report** for easy viewing
3. Click **Next → Finish**
4. Autopsy will save the report in your case folder — open `index.html` in a browser to view it

---

## 🖼️ Screenshots & Diagrams

### Autopsy Splash Screen (Loading)
> *Screenshot: Autopsy launching for the first time showing the splash screen and loading bar*

![Autopsy Splash Screen](screenshots/autopsy-splash.png)

### New Case Setup
> *Screenshot: Case name and base directory configuration screen*

![New Case](screenshots/new-case-setup.png)

### Ingest Modules Selected
> *Screenshot: Ingest module selection screen with recommended modules checked*

![Ingest Modules](screenshots/ingest-modules.png)

### Autopsy Processing the Image
> *Screenshot: Autopsy running ingest with progress bar visible at the bottom*

![Processing](screenshots/autopsy-processing.png)

### Investigation Results
> *Screenshot: Left panel showing recovered artifacts, file tree, and keyword hits*

![Results](screenshots/investigation-results.png)

### Generated Report
> *Screenshot: HTML report exported from Autopsy showing findings summary*

![Report](screenshots/autopsy-report.png)

---

## 🧩 Forensic Investigation Workflow

```
Disk Image (.E01 / .dd)
        │
        ▼
  Open in Autopsy
        │
        ▼
  Run Ingest Modules
  ┌─────────────────────────────┐
  │ File Type ID                │
  │ Recent Activity             │
  │ Keyword Search              │
  │ Hash Lookup                 │
  │ Extension Mismatch Detector │
  └─────────────────────────────┘
        │
        ▼
  Analyze Artifacts
  ┌──────────────────────┐
  │ Deleted Files        │
  │ Browser History      │
  │ Recent Documents     │
  │ Keyword Hits         │
  │ Timeline Events      │
  └──────────────────────┘
        │
        ▼
  Generate Report
```

---

## ⚠️ Troubleshooting Notes & Challenges

### Challenge 1 — Autopsy Takes a Long Time to Load
**Issue:** The splash screen appears frozen on first launch.
**Solution:** This is normal behavior. Autopsy loads all its modules in the background before showing the main interface. Wait 2–5 minutes without clicking anything. Do not force close it.

### Challenge 2 — Ingest Takes Longer Than Expected
**Issue:** Processing the disk image runs for 30+ minutes.
**Solution:** This is also normal, especially on a mechanical hard drive or with limited RAM. Keep Autopsy as the active application and close other programs to free up memory. Running the image from an SSD significantly speeds things up.

### Challenge 3 — Can't Find the Disk Image File After Unzipping
**Issue:** After unzipping the Hunter lab download, the image file isn't visible.
**Solution:** The `.E01` file may be inside a subfolder within the zip. Open the unzipped folder and look inside any subfolders. Also make sure Windows is showing file extensions (View → Show → File name extensions).

### Challenge 4 — Learning Which Artifacts to Look For
**Issue:** Autopsy surfaces a lot of data and it's unclear where to start.
**Solution:** Read the Hunter lab questions on CyberDefenders first. Each question points you toward a specific artifact type (e.g., "What website was visited?" → check Web History; "What file was downloaded?" → check Downloads). Use the questions as a roadmap.

### Challenge 5 — Understanding Forensic File Formats
**Issue:** Unfamiliar with `.E01` vs `.dd` image formats.
**Solution:** Both are forensic disk image formats that Autopsy supports natively. `.E01` (EnCase format) is more common in professional forensics and includes metadata and checksums. `.dd` is a raw bit-for-bit copy. Either will work the same way in Autopsy.

### Challenge 6 — Hardware Resource Constraints
**Issue:** Computer slows down significantly during ingest.
**Solution:** Close all browser tabs and background applications before starting ingest. If the computer has less than 8GB of RAM, consider using a smaller practice image first.

---

## 📝 Key Takeaways

- Digital forensics is not just about finding files — it's about **telling a story** of what happened on a system, in what order, and by whom.
- Autopsy's **ingest modules** do the heavy lifting, but the investigator still needs to know what questions to ask and where to look.
- **Non-destructive analysis** is a core principle — always work from a copy of the original image, never the original itself.
- Deleted files are often recoverable because deleting a file only removes its reference in the file system — the actual data remains on disk until overwritten.
- This project reinforced real-world incident response skills that are directly applicable to roles in cybersecurity, particularly in **blue team** and **forensics** positions.

---

*CSN 150 | New York | Spring 2026*
