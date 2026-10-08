# Week 6 Forensic Lab Report: Deleted Files Recovered on Terminated Employee's Laptop
**Course:** CPSC 4584 · Computer Forensics  
**Case ID:** FOR-2026-1005-001  
**Organization:** Maplewood Health System Security Operations Center  
**Analyst:** Tier 1 SOC Forensic Analyst  
**Date:** October 5, 2026  

---

## 1. Executive Summary & Incident Overview
On October 3, 2026, at 2:15 PM, billing coordinator Jordan Ellis was delivered an immediate termination notice following a workplace conduct investigation. Security escorted Ellis from the premises at 2:47 PM. Due to an administrative coordination failure between HR and IT, Ellis's assigned company laptop was not surrendered upon termination. Ellis retained unrestricted physical possession of the laptop until voluntarily returning it to the IT Helpdesk on October 5 at 9:30 AM (a ~43-hour custody gap).

Operating system event logs indicated device activity and multiple file deletion events occurring between October 3 at 11:30 PM and October 4 at 12:04 AM. Forensic imaging was conducted using FTK Imager on October 5 at 10:15 AM with SHA-256 hash verification. Five files were carved from unallocated disk space using Foremost.

---

## 2. Chain of Custody & Evidence Log

### Custody Timeline & Control Gap
* **Oct 3, 2:15 PM:** Termination notice delivered in HR conference room (laptop not collected).
* **Oct 3, 2:47 PM:** Ellis escorted from building by Security while retaining laptop.
* **Oct 3, 2:47 PM – Oct 5, 9:30 AM:** **UNACCOUNTED (~43-hour custody gap).** Device in Ellis's unrestricted possession; deletion events logged Oct 3, 11:30 PM – Oct 4, 12:04 AM.
* **Oct 5, 9:30 AM:** Device voluntarily returned at IT Helpdesk; hardware write-blocker applied immediately.
* **Oct 5, 10:15 AM:** Forensic image created with FTK Imager; SHA-256 hash verified against source drive.
* **Oct 5, 11:42 AM:** File carving completed via Foremost; 5 files recovered from unallocated space.

### Recovered Evidence Table
| Filename | Type | Size | Last Modified | Deletion Timestamp | Recovery Status |
| :--- | :--- | :--- | :--- | :--- | :--- |
| `insurance_export_final.xlsx` | Spreadsheet | 2.4 MB | Oct 3, 11:47 PM | Oct 3, 11:52 PM | Complete |
| `personal_vacation_2025.jpg` | Image | 3.1 MB | Jul 14, 2025 | Oct 3, 11:58 PM | Complete |
| `vendor_contact_list_external.docx` | Document | 89 KB | Oct 2, 3:22 PM | Oct 3, 11:55 PM | Complete |
| `mhs_billing_schema_db.sql` | Database | 156 KB | Oct 1, 9:14 AM | Oct 3, 11:53 PM | Complete |
| `__tmp_8f2a.dat` | Partial | 22 KB / ~400 KB | Unreadable | Unreadable | Partial (Header intact) |

---

## 3. Written Forensic Analysis

### Prompt 1: Chain of Custody Gap Analysis
Between October 3 at 2:47 PM and October 5 at 9:30 AM, Jordan Ellis retained unrestricted physical possession of the laptop for approximately 43 hours following termination due to an IT/HR coordination failure. Because Maplewood Health System lacked physical control or system monitoring throughout this period, the provenance and integrity of the device during this window cannot be independently verified. From a forensic standpoint, this custody gap means that while recovered artifacts prove activity occurred on the machine, the analyst cannot conclusively prove who performed the actions or guarantee that external anti-forensic tools or secondary physical access did not alter system state without corroborating network or authentication logs.

### Prompt 2: Deletion Activity and Evidentiary Implications
File system deletion updates directory entries and inode allocation metadata while leaving underlying data blocks in unallocated space until overwritten. Foremost carved five files (`insurance_export_final.xlsx`, `personal_vacation_2025.jpg`, `vendor_contact_list_external.docx`, `mhs_billing_schema_db.sql`, and partial `__tmp_8f2a.dat`) using file header/footer signatures. While timestamps show file modification and deletion between 11:30 PM and 12:04 AM following termination, this technical evidence establishes *what* was deleted and *when*, but does **not** prove intent, authorization context, or bad faith. The analyst must document these technical facts objectively without inferring employee culpability.

### Prompt 3: Forensic Escalation Report Summary
As a Tier 1 SOC Forensic Analyst, my role is restricted to preserving evidence, documenting recovery techniques (bit-stream imaging, Foremost carving, SHA-256 verification), and reporting verified technical findings. Determining policy violations, legal liability, or criminal intent falls strictly under the authority of legal counsel and HR reviewers. This technical report summarizes the verified timeline, carved artifacts, and the 43-hour custody gap. It is formally escalated to SOC management and authorized legal/HR representatives to integrate with broader investigative findings (such as badge logs, network auth, and HR records).

---

## 4. Practice Metadata Inspection Log (CyLab Practice)

* **Terminal Entry 1 (`stat`):** Retrieved full inode metadata, file size, permissions, and MACB timestamps for practice target `practice_file.txt`.
* **Terminal Entry 2 (`ls -li`):** Listed directory files alongside inode index numbers, illustrating directory-to-inode mapping.
* **Terminal Entry 3 (`xxd -l 48`):** Examined the initial 48 bytes (first 3 hex rows) of `README.txt` to identify magic byte file header signatures.
* **Terminal Entry 4 (`find`):** Executed `-mmin` time-window searches to isolate files created or altered within targeted temporal boundaries.
