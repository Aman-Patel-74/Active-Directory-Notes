# 🔎 Enum4linux & Enum4linux-ng

> Offensive-Active-Directory / Enum4linux-and-Enum4linux-ng

---

## 📌 Table of Contents

- [Overview](#overview)
- [Installation](#installation)
- [Enum4linux Usage](#enum4linux-usage)
- [Enum4linux-ng Usage](#enum4linux-ng-usage)
- [Key Differences](#key-differences)

---

## Overview

`enum4linux` and `enum4linux-ng` are tools used to enumerate information from **Windows / SMB services**. They are commonly used during **Active Directory reconnaissance** to gather:

- User accounts
- Group memberships
- Shares
- Password policies
- Domain information

---

## Installation

### Install enum4linux

```bash
apt install enum4linux
```

### Install enum4linux-ng

```bash
apt install enum4linux-ng
```

---

## Enum4linux Usage

### Basic Enumeration

Run a comprehensive scan against a target with the `-a` flag:

```bash
enum4linux -a 192.168.1.50
```

---

## Enum4linux-ng Usage

### Help Menu

```bash
enum4linux-ng -h
```

### Full Enumeration

```bash
enum4linux-ng -A 192.168.1.50
```

> The `-A` flag runs a **comprehensive scan**, similar to `-a` in enum4linux but with improvements:

- Better parsing and output formatting
- More reliable enumeration
- Supports modern SMB configurations
- Faster and cleaner results

---

## Key Differences

| Feature | enum4linux | enum4linux-ng |
|---|---|---|
| **Maintenance** | Old / Deprecated | Actively maintained |
| **Output** | Messy | Clean & structured |
| **Speed** | Slower | Faster |
| **Reliability** | Less reliable | More accurate |
| **SMB Support** | Limited | Improved |

---

<p align="center"><sub>📚 Notes compiled for personal study & offensive security learning.</sub></p>
