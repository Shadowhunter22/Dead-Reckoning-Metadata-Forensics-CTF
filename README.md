# 🕵️ Dead Reckoning — Metadata Forensics CTF Write-Up

> **Challenge:** Dead Reckoning — Metadata Forensics CTF  
> **Category:** Digital Forensics / Metadata Analysis  
> **Difficulty:** Advanced  
> **Author:** Nana Sei Anyemedu — Hive Consult, Ghana  
> **Analyst:** Abubakar Sadik  

---

## 📖 Case Background

A senior operations analyst at **Accra Petroleum Holdings (APH)** was found dead in his apartment on the night of **April 9, 2024**. His company laptop was missing. Three days prior, highly classified petroleum exploration contracts were leaked to an unknown foreign entity.

Police recovered a USB drive at the scene containing **four image files**. A prime suspect — **Kofi Mensah-Addo**, APH Operations Department — claimed he was in London at the time, about to board a flight back to Accra.

**Mission:** Prove or disprove his alibi using metadata.

---

## 🗂️ Evidence Files

| File | Description |
|------|-------------|
| `crime_scene_photo.jpg` | CCTV capture from the scene |
| `suspect_device.jpg` | Photo from suspect's personal phone |
| `leaked_document_scan.jpg` | Scan of the leaked contract |
| `alibi_photo.jpg` | Suspect's claimed airport selfie |

---

## 🛠️ Tools Used

| Tool | Purpose |
|------|---------|
| `exiftool` | Extract metadata from all image files |
| `exiftool -b \| strings` | Extract binary MakerNote hidden data |
| `base64 -d` | Decode base64-encoded hidden flag |

---

## 🔍 Step 1 — Clone the Lab & Inspect Files

```bash
git clone https://github.com/RedHatPentester/DIGITAL-FORENSICS-CTF-LAB.git
cd DIGITAL-FORENSICS-CTF-LAB/Metadata\ Analysis\ CTF/
ls -la
cat README.md
```

![Step 1 - Cloning and listing files](ss1.png)

This revealed four JPEG evidence files and a README detailing the 8 questions to solve.

---

## 🔍 Step 2 — Dump Metadata from All Files

```bash
exiftool crime_scene_photo.jpg suspect_device.jpg leaked_document_scan.jpg alibi_photo.jpg
```

![Step 2 - ExifTool output for crime_scene_photo.jpg](ss2.png)

---

## ✅ Q1 — True Location in Crime Scene Photo

From `crime_scene_photo.jpg` metadata:

```
GPS Latitude  : 5 deg 38' 59.64" N
GPS Longitude : 0 deg 10' 36.84" W
GPS Position  : 5 deg 38' 59.64" N, 0 deg 10' 36.84" W
```

📍 **Finding:** The GPS coordinates place the CCTV camera at **Meridian Towers, Accra, Ghana** — contradicting the official report's stated location.

---

## ✅ Q2 — Two Timestamps & The Discrepancy

```
Date/Time Original : 2024:04:06 19:47:11   ← when photo was taken
Modify Date        : 2024:04:09 22:14:33   ← when file was modified
```

⚠️ **Finding:** The footage was originally captured on **April 6** but was modified/re-processed on **April 9 at 22:14** — the same night as the murder. This is clear evidence of **footage tampering**.

---

## ✅ Q3 — Timezone in Suspect's Phone Photo

![Step 3 - suspect_device.jpg metadata](ss3.png)

```
Offset Time Original : +00:00
```

🌍 **Finding:** London in April operates on **BST (UTC+1)**. The suspect's Samsung Galaxy S24 Ultra recorded timezone **+00:00 (UTC/Ghana time)**. This proves his phone was **not in the UK** when the photo was taken.

---

## ✅ Q4 — Device Serial Number (Hidden in Binary MakerNote)

```bash
exiftool -b -MakerNoteUnknownText suspect_device.jpg | strings
```

```
Samsung|SN:R5CNA04JKBM|IMEI:356841112947603|[REDACTED]
```

📱 **Finding:**
- **Device:** Samsung SM-S928B (Galaxy S24 Ultra)
- **Serial Number:** `R5CNA04JKBM`
- **IMEI:** `356841112947603`
- 🚩 **Flag:** `[REDACTED]`

---

## ✅ Q5 & Q6 — Leaked Document Author & Email

![Step 4 - leaked_document_scan.jpg metadata](ss4.png)

```
Artist   : Kofi Mensah-Addo | APH Operations Dept | Unauthorized Copy
Software : APH-DocVault v2.1.4 [INTERNAL] | User: k.mensah-addo@aph.com.gh
Scanner  : Canon imageFORMULA DR-C230
```

📄 **Findings:**
- **Real Author:** Kofi Mensah-Addo
- **Internal Software:** APH-DocVault v2.1.4
- **Email embedded:** `k.mensah-addo@aph.com.gh`

The Artist field self-incriminates with the tag **"Unauthorized Copy"**. The suspect used his own corporate account to scan and leak classified documents.

---

## ✅ Q7 — Alibi Photo GPS vs Heathrow

![Step 5 - alibi_photo.jpg metadata](ss5.png)

```
GPS Latitude  : 5 deg 36' 18.72" N
GPS Longitude : 0 deg 10' 0.48" W
GPS Position  : 5 deg 36' 18.72" N, 0 deg 10' 0.48" W
```

| Location | Coordinates |
|----------|-------------|
| **Embedded in photo** | 5.6052°N, 0.1668°W → **Kotoka Intl. Airport, Accra** |
| **Heathrow (claimed)** | 51.4700°N, 0.4543°W → London, England |
| **Distance apart** | ~5,160 km |

✈️ **Finding:** The suspect claimed to be at Heathrow Airport. The GPS in his iPhone 15 Pro Max places him at **Kotoka International Airport, Accra, Ghana**. Alibi destroyed.

---

## ✅ Q8 — Base64 Flag in XPComment Field

![Step 6 - MakerNote extraction and base64 decode](ss6.png)

```
XP Comment : SElWRXtERUFEX1JFQ0tPTklOR19DT01QTEVURV9LT0ZJX01FTlNBSF9BRERPX0lTX1lPVVJfTUFOfQ==
```

```bash
echo "SElWRXtERUFEX1JFQ0tPTklOR19DT01QTEVURV9LT0ZJX01FTlNBSF9BRERPX0lTX1lPVVJfTUFOfQ==" | base64 -d
```

```
[REDACTED]
```

🚩 **Flag:** `[REDACTED]`

---

## 🏁 Flags Captured

| # | Flag |
|---|------|
| 🚩 Flag 1 | `[REDACTED]` |
| 🚩 Flag 2 | `[REDACTED]` |

---

## 📋 Full Answers Summary

| Question | Answer |
|----------|--------|
| **Q1** — True GPS location | `5° 38' 59.64" N, 0° 10' 36.84" W` → Meridian Towers, Accra |
| **Q2** — Timestamp discrepancy | Taken Apr 6 19:47, modified Apr 9 22:14 → footage tampered |
| **Q3** — Timezone offset | `+00:00` (Ghana/UTC) not `+01:00` (BST) → not in London |
| **Q4** — Device serial | `R5CNA04JKBM` / IMEI: `356841112947603` |
| **Q5** — Document author & software | Kofi Mensah-Addo / APH-DocVault v2.1.4 |
| **Q6** — Embedded email | `k.mensah-addo@aph.com.gh` |
| **Q7** — Alibi photo GPS | Kotoka Airport, Accra — NOT Heathrow |
| **Q8** — Decoded flag | `[REDACTED]` |

---

## 🧠 Key Lessons

- **ExifTool** is the single most powerful tool for metadata forensics CTFs
- GPS coordinates embedded in images **never lie** — always cross-check claimed locations
- **Timezone offsets** in EXIF data can disprove location-based alibis
- Binary MakerNote fields can hide **device identifiers and flags** — always extract with `-b | strings`
- **Timestamps** can reveal tampering — compare `Date/Time Original` vs `Modify Date`
- Base64 strings hidden in obscure fields like `XPComment` are classic CTF flag placement

---

## 📚 References

- [ExifTool Documentation](https://exiftool.org/)
- [CTF Lab Repository](https://github.com/RedHatPentester/DIGITAL-FORENSICS-CTF-LAB)
- [CyberChef — Online Decoder](https://gchq.github.io/CyberChef)

---

*Write-up by **Abubakar Sadik** — Digital Forensics CTF*
