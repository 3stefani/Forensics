# ⭐ Digital Forensics Course Capstone - Writeup ⭐

![Digital Forensics](https://img.shields.io/badge/Digital_Forensics-Analysis-blue)
![Status](https://img.shields.io/badge/Status-Completed-success)
![Difficulty](https://img.shields.io/badge/Difficulty-Beginner-green)
![Evidence](https://img.shields.io/badge/Evidence-4%2F4-orange)
![Tools](https://img.shields.io/badge/Tools-Linux_CLI%2C_Steghide%2C_John-purple)

## Table of Contents

- [Overview](#overview)
- [Challenge Description](#challenge-description)
- [Environment Setup](#environment-setup)
- [Investigation Workflow](#investigation-workflow)
  - [Evidence 1/4: Hidden Message in Form1.jpg](#evidence-14-hidden-message-in-form1jpg)
  - [Evidence 2/4: VPN Credentials in laptop.jpg](#evidence-24-vpn-credentials-in-laptopjpg)
  - [Evidence 3/4: Employee Database in Protected ZIP](#evidence-34-employee-database-in-protected-zip)
  - [Evidence 4/4: Colin Andrews Intelligence](#evidence-44-colin-andrews-intelligence)
  - [Bonus: File Extension Manipulation](#bonus-file-extension-manipulation)
- [Evidence Summary](#evidence-summary)
- [Key Findings](#key-findings)
- [Lessons Learned](#lessons-learned)
- [Tools Reference](#tools-reference)
- [Resources](#resources)
- [Author](#author)

## Overview

This repository contains my writeup for the **Course Capstone Challenge** from Security Blue Team's "Introduction to Digital Forensics" course. The challenge simulates a real-world digital forensics investigation involving data exfiltration by an employee.

## Challenge Description

**Scenario**: The SOC received an anonymous report about a user (J. Harrison) potentially exfiltrating company data. A forensic image of the user's hard drive was obtained, and we must analyze it to find 4 pieces of hidden evidence.

**Objectives**:
- Discover 4 pieces of evidence (marked with `{X of 4}`)
- Apply Linux CLI navigation techniques
- Identify incorrect file extensions
- Discover hidden files and folders
- Use steganography techniques
- Crack password-protected files

**Starting Point**: The most recent file on the hard drive was an email file with an attachment in the "Saved Emails" directory.

## Environment Setup

**Virtual Machine**:
- VMware Workstation (alternative to VirtualBox)
- Kali Linux

**Tools Used**:
- `ls -a` - List hidden files
- `cat` - Read file contents
- `strings` - Extract text strings from files
- `file` - Identify true file type
- `steghide` - Extract steganographic data
- `john` - Password cracking (alternative to fcrackzip)
- `zip2john` - Convert ZIP to hash format
- `unzip` - Extract ZIP files
- `find` - Search for files
- `grep` - Search patterns in text

**Wordlist**:
- `/usr/share/wordlists/rockyou.txt`

## Investigation Workflow

### Evidence 1/4: Hidden Message in Form1.jpg

**Location**: `Saved Emails/Form1.jpg`

**Step 1: Analyze the Email**

Navigate to starting directory:

```
cd "Saved Emails"
ls -la
```

Found files:
- `Form1.jpg`
- `Website Update Report 01_10_2019.eml`

Read email content:
```
cat 'Website Update Report 01_10_2019.eml'
```
**Email content**:
From: Price T price.t@dicksonunited.com
To: otnacask18491@outlook.com
Subject: Website Update Report 01/10/2019
Hi Jonathan,
Here's the report for the website uptime and updates that were applied
today (01/10/2019).
Please let me know if there are any issues. Attached as Form1.jpg
All the best,
Price.T
Lead Web Developer
DicksonUnited

**Step 2: Analyze Form1.jpg**

I proceed to open the Form1.jpg image referred to in the message. It appears to be a blank form, with no relevant information.

Extract hidden strings from image:

```
strings Form1.jpg

```

**🎯 EVIDENCE 1/4 FOUND**:
```
Simon, I have usernames and passwords for the VPN. Still on my work PC.
Don't want to risk emailing them just yet. When I do, the file is a .jpg
image + password for extraction is password. Use steghide. Talk again soon
{1 of 4}
```
**Information Obtained**:
- **File**: `Form1.jpg`
- **Directory**: `Saved Emails`
- **Technique**: String extraction
- **Evidence**: Hidden message revealing:
  - VPN credentials exist
  - Hidden in JPG with steghide
  - Extraction password: `password`

### Evidence 2/4: VPN Credentials in laptop.jpg

Based on the previous clue, we know there's a JPG image with extractable information using steghide with password `password`.

**Step 1: Search for Images**

Explore disk for JPG files. I go through the images and, sure enough, I find one called laptop.jpg from which I can extract the "passwords" file using steghuide.


**Step 2: Extract from laptop.jpg**

Test steghide on images found:
```
steghide extract -sf laptop.jpg -p password

```
**Output**:
```
wrote extracted data to "passwords.txt"

```
**Step 3: Analyze Extracted File**
```
cat passwords.txt

```
**🎯 EVIDENCE 2/4 FOUND**:

**File contains passwords**

**Information Obtained**:
- **File**: `laptop.jpg`
- **Directory**: (images directory)
- **Technique**: Steganography with steghide
- **Evidence**: `passwords.txt` file containing passwords

### Evidence 3/4: Employee Database in Protected ZIP

**Step 1: Search for Hidden Files**

Using `ls -a` to find hidden files/directories.

After a file search, I find the following file ‘.a0415ns.zip’ hidden in /WebDev work/unfinidhed webpages/to-do/
```
cd to-do
ls -la
```

**Step 2: Attempt to extract**

Found: `.a0415ns.zip`

Attempt to extract:

```
unzip .a0415ns.zip

```
**Error**: Password protected

This zip file is password-protected. To do this, we used John the Ripper with the Rockyou dictionary.


**Step 3: Password Cracking with John the Ripper**

Convert ZIP to hash format:

```
zip2john .a0415ns.zip > zip.hash

```
Run John with rockyou wordlist:

```
john --wordlist=/usr/share/wordlists/rockyou.txt zip.hash

```
**Password found**: `vendy13031998`

View cracked password:

```
john --show zip.hash

```
**Step 4: Extract ZIP Contents**

```
unzip -P vendy13031998 .a0415ns.zip

```
Extracted file: `employeedump`

**Step 5: Analyze CSV**

```
cat employeedump
```

**🎯 EVIDENCE 3/4 FOUND**:


```
{3 of 4}
,First Name,Last Name,Gender,Country,Age,Date,VPN UserID
1,Dulce,Abril,Female,United States,32,15/10/2017,DA32
2,Mara,Hashimoto,Female,Great Britain,25,16/08/2016,MH25
3,Philip,Gent,Male,France,36,21/05/2015,PG36
[... 291 employee records total ...]
```
**Information Obtained**:
- **File**: `.a0415ns.zip` → `employeedump.csv`
- **Directory**: `to-do/` (hidden)
- **Technique**: Hidden file discovery + password cracking
- **Evidence**: Employee database with 291 records containing:
  - Personal information (name, gender, age, country)
  - VPN UserIDs
  - Sensitive company data

**Techniques Applied**:
- Hidden directory search
- Password cracking with dictionary attack
- Structured data analysis

### Evidence 4/4: Colin Andrews Intelligence

**Step 1: Comprehensive Search**

Continue exploring the disk for the fourth piece of evidence:


**Step 2: Analyze Suspicious Files**

Found file with unusual name: `bootstrap.min.abc`

The `.abc` extension is uncommon. Investigate:

```
file bootstrap.min.abc

```
Result: ASCII text file

Read content:

```
cat bootstrap.min.abc

```

**🎯 EVIDENCE 4/4 FOUND**:

Inside the file, commented in the code:

```
{4 of 4}
/*
this is in case Colin tries to screw me. I'll expose him.
Colin Andrews
31 years old
lives in Suffolk, UK
drives a Kia sportage
buys and sells valid company credentials to hackers
phishing attacks, malware distribution
(still got his email addresses, domains, mobile no. and BTC wallet address on my personal PC)
*/
```
**Information Obtained**:
- **File**: `bootstrap.min.abc`
- **Directory**: (web files directory)
- **Technique**: Suspicious file analysis
- **Evidence**: Intelligence about Colin Andrews:
  - Sells corporate credentials to hackers
  - Participates in phishing attacks
  - Distributes malware
  - Personal and contact information

### Bonus: File Extension Manipulation

During investigation, found another suspicious file.

**File: posidon.xml**

Verify true file type:


```
file posidon.xml

```

**Output**:

```
posidon.xml: PNG image data, 1920 x 1080, 8-bit/color RGB, non-interlaced

```
**The file is NOT XML, it's a PNG image!**

**Correct Extension**


```
mv posidon.xml posidon.png

```
Open image:

```
eog posidon.png

```
**Content**: Map or blueprint with company office locations.

**Obfuscation technique**: File extension changed to evade superficial detection.

## Evidence Summary

| # | File | Location | Technique | Evidence Found |
|---|------|----------|-----------|----------------|
| 1/4 | Form1.jpg | Saved Emails | `strings` command | Hidden message about VPN credentials |
| 2/4 | laptop.jpg | (images) | Steganography (steghide) | passwords.txt with VPN credentials |
| 3/4 | .a0415ns.zip | to-do/ (hidden) | Password cracking (John) | employeedump.csv with 291 records |
| 4/4 | bootstrap.min.abc | (web files) | Text analysis | Intelligence on Colin Andrews |
| Extra | posidon.xml | (root) | Extension verification | Office map (PNG disguised as XML) |

## Key Findings

### Security Incidents Identified

1. **VPN Credential Exfiltration**: Use of steganography to hide passwords in images
2. **Employee Database Theft**: 291 records with personal information and VPN UserIDs
3. **Insider Threat Evidence**: Documentation of Colin Andrews selling company credentials
4. **Anti-Forensic Techniques Used**:
   - Steganography in image files
   - Hidden files/folders (`.` prefix)
   - Password-protected archives
   - File extension manipulation
   - Information hidden in legitimate-looking files

### Attack Timeline

1. Employee (J. Harrison) communicated with external party (Simon)
2. Collected VPN credentials from multiple users
3. Extracted employee database with 291 records
4. Used multiple obfuscation techniques to hide evidence
5. Maintained intelligence on another insider threat (Colin Andrews)
6. Stored infrastructure information (office locations)

### Forensic Techniques Applied

- **Linux CLI Navigation**: Efficient file system exploration
- **Hidden File Discovery**: Using `ls -a` and `find` commands
- **File Type Verification**: `file` command to detect manipulated extensions
- **Steganography Extraction**: `steghide` for hidden data in images
- **Password Cracking**: `john the ripper` with dictionary attack
- **String Extraction**: `strings` command for text in binary files
- **Chain of Custody**: Documentation of evidence location and extraction methods

## Lessons Learned

### 1. Extensions Don't Define Files

Always verify file types with the `file` command. Attackers commonly change extensions to evade detection.

### 2. Search Everywhere

Hidden files with `.` prefix are commonly used to hide malicious content. Always use `ls -a` and recursive searches.

### 3. Context Matters

Information in one file often leads to others. The email led to Form1.jpg, which led to laptop.jpg, creating a chain of evidence.

### 4. Multiple Layers of Obfuscation

Sophisticated attackers use combined techniques:
- Steganography + password protection
- Hidden directories + encrypted archives
- Legitimate file names with hidden content

### 5. Documentation is Critical

Maintaining proper chain of custody and documentation is essential for:
- Legal admissibility in court
- Reproducibility of findings
- Professional accountability

### 6. Think Like the Attacker

Understanding common anti-forensic techniques helps anticipate where evidence might be hidden:
- Steganography in common file types
- Password-protected archives with weak passwords
- Extension manipulation
- Hidden system folders

## Resources

### Course

- [Security Blue Team - Introduction to Digital Forensics](https://securityblue.team/)

### Tools

- [Kali Linux](https://www.kali.org/) - Penetration testing and forensics distribution
- [Steghide](http://steghide.sourceforge.net/) - Steganography tool
- [John the Ripper](https://www.openwall.com/john/) - Password cracker
- [VMware Workstation](https://www.vmware.com/products/workstation-pro.html) - Virtualization platform

### Wordlists

- [SecLists](https://github.com/danielmiessler/SecLists) - Collection of wordlists
- rockyou.txt - Pre-installed in Kali Linux at `/usr/share/wordlists/rockyou.txt`

### Standards & References

- [RFC 3227 - Guidelines for Evidence Collection and Archiving](https://tools.ietf.org/html/rfc3227)
- [NIST SP 800-86 - Guide to Integrating Forensic Techniques into Incident Response](https://csrc.nist.gov/publications/detail/sp/800-86/final)
- [SANS Digital Forensics & Incident Response](https://www.sans.org/cyber-security-courses/advanced-incident-response-threat-hunting-training/)

### Additional Reading

- Digital Forensics and Incident Response (DFIR) methodologies
- Order of Volatility in digital evidence
- Chain of Custody best practices
- Anti-forensic techniques and countermeasures

## Challenge Questions Summary

### General Knowledge Questions

1. Most volatile evidence: **Cache**
2. Chain of Custody: Paper trail showing evidence location and possession
3. Locked laptop scenario: Wait for user to sign in themselves
4. Social media as evidence: Yes
5. Most valuable evidence type: **Real Evidence**
6. fcrackzip flags for brute force: `-b -c a1`
7. Digital Forensics entails: Identification, preservation, and analysis of electronic evidence
8. HTCIA: **High Technology Crime Investigation Association**
9. Computer Forensics vs Data Recovery: **False** (different disciplines)
10. Corporate Policies: Can dictate **all of the above**

### Practical Challenge Questions

All 4 pieces of evidence successfully located and documented with:
- File names and extensions
- Directory locations
- Extraction techniques used
- Evidence content identified


## Acknowledgments

- **Security Blue Team** for creating this excellent hands-on challenge
- The digital forensics community for developing and maintaining open-source tools
- Everyone who shared writeups and learning resources

## License

This writeup is for educational purposes only. All techniques demonstrated were performed in a controlled lab environment as part of authorized training.

**Disclaimer**: Do not use these techniques on systems you don't own or without explicit authorization. Unauthorized access to computer systems is illegal.

## Contributing

Found a mistake or have suggestions? Feel free to:
- Open an issue
- Submit a pull request
- Contact me directly

## Tags

`digital-forensics` `steganography` `password-cracking` `linux` `dfir` `cybersecurity` `blue-team` `incident-response` `writeup` `ctf` `security-blue-team`



