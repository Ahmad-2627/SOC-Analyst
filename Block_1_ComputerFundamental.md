# Block 1 — Computer Fundamentals
### Elite SOC Analyst Academy | Study Notes

---

## Purpose of This Block

Before any cybersecurity tool or technique makes sense, you must understand **how a computer actually works** internally. Every future topic (Windows, Linux, Malware Analysis, Digital Forensics, Incident Response) stands on this foundation. If you don't know what "normal" looks like, you can never recognize what's "abnormal" (malicious).

**13 Modules + 3 Hands-On Labs**

---

## Module 1.1 — What Is a Computer?

**Core cycle every computer follows:**

```
  INPUT  --->  PROCESS  --->  OUTPUT
                  |
                  v
              STORAGE  (saves data for later, can reload it back into Process)
```

- **Input** — data/commands entering the system (keyboard, mouse, mic)
- **Process** — the CPU (the "brain") does the work
- **Output** — the result shown back (screen, speaker, printer)
- **Storage** — where data is saved permanently for future use

**Hardware vs Software vs Firmware:**
| Term | Meaning |
|---|---|
| **Hardware** | Physical parts you can touch (CPU, keyboard, screen) |
| **Software** | Instructions/programs that tell hardware what to do (Windows, Chrome) |
| **Firmware** | Special software permanently stored inside hardware (e.g. BIOS) that helps it start up |

**Key insight:** Almost every process produces *some* output — even a "silent" background save to storage counts as output (e.g. an app quietly writing a log file).

---

## Module 1.2 — Computer Architecture

The **motherboard** connects every component together — like the wiring of a building.

| Component | Role |
|---|---|
| **CPU** | The "brain" — does calculations and decisions |
| **RAM** | Short-term, **volatile** memory (data lost when powered off) |
| **Storage (HDD/SSD)** | Long-term memory — permanent even when powered off |
| **GPU** | Draws images/graphics fast (gaming, video editing) |
| **BIOS/UEFI** | Firmware that wakes up hardware at startup |
| **Buses** | Data pathways between components (like roads) |
| **Power Supply (PSU)** | Feeds electricity to every part |
| **System Clock** | Synchronizes timing across the CPU |

**Security relevance:**
- RAM is **volatile** — this is why "live memory capture" happens *before* restarting a suspect machine (evidence disappears otherwise).
- Unusual CPU usage can indicate malware (e.g. crypto-mining, ransomware encryption).

---

## Module 1.3 — CPU Fundamentals

**Structure:**

```
CPU Chip
 ├── Core 1 → Registers, L1 cache, L2 cache
 ├── Core 2 → Registers, L1 cache, L2 cache
 └── Shared L3 cache (used by all cores)
```

| Term | Meaning |
|---|---|
| **Core** | An independent mini-processor inside the CPU; more cores = more true parallel work |
| **Threads** | Virtual "extra workers" inside a core (e.g. Intel Hyper-Threading) |
| **Clock Speed (GHz)** | How many billions of cycles per second the CPU runs |
| **Registers** | Smallest, fastest storage — holds data mid-calculation |
| **Cache (L1 → L2 → L3)** | Fast memory holding frequently-used data; L1 fastest/smallest, L3 largest/shared |
| **Instruction Cycle** | Fetch → Decode → Execute → repeat, billions of times per second |
| **Interrupts** | Urgent signals that pause the CPU to handle something immediately (e.g. a keypress) |

**Memory/Speed Hierarchy:** `Registers > Cache (L1→L2→L3) > RAM > Storage`
(Each step down = slower but more capacity.)

**Important nuance:** 8 cores = 8 truly parallel tasks. But dozens of processes can *appear* to run simultaneously because the OS rapidly switches between them (**time-slicing / multitasking**).

**Security relevance:** Malware (crypto-miners, ransomware) often causes abnormally high CPU usage — a red flag during investigation.

---

## Module 1.4 — Memory Fundamentals

| Type | Property |
|---|---|
| **RAM** | Volatile working memory — holds currently running processes |
| **ROM** | Non-volatile — permanent, holds firmware instructions, normally read-only |
| **Virtual Memory** | Uses disk space as "extra RAM" when RAM is full (Windows: *page file*, Linux: *swap*) |
| **Stack** | Function-call memory, automatic management (LIFO), **short-lived** |
| **Heap** | Dynamic memory, manual management by the program — risk of **memory leaks** if not freed properly |

**Memory Lifecycle:** Program starts → OS allocates memory → program uses it → program ends → memory is freed for reuse.

**Security relevance — very important:**
- **Memory Forensics** — analysts capture a **memory dump** (RAM snapshot) to see what was running, including passwords and malware code that never touched the disk.
- **Fileless Malware** — malware that runs *only* in RAM, never writing to disk — traditional antivirus (which scans disk) often misses it. Only memory analysis catches it.

**Performance note:** When RAM is full and virtual memory kicks in, the system slows down significantly (disk is much slower than RAM). Excessive swapping back and forth is called **thrashing**.

---

## Module 1.5 — Storage Fundamentals

**Structure:**

```
Physical Disk (HDD or SSD)
 ├── Partition 1 (e.g. Boot Partition)
 │     └── Volume (formatted, e.g. NTFS)
 │           └── Files → broken into Blocks
 └── Partition 2 (Data Partition)
       └── Volume → Files, Apps
```

| Term | Meaning |
|---|---|
| **HDD** | Hard Disk Drive — spinning disk, moving parts, cheaper, slower |
| **SSD** | Solid State Drive — no moving parts, faster, more common now |
| **Blocks** | Fixed-size chunks a file is split into when stored on disk |
| **Partition** | A large logical section of a disk (e.g. C: drive, D: drive) |
| **Volume** | A formatted, usable partition |
| **Formatting** | Preparing a disk with a specific file system structure |
| **Boot Partition** | Special partition holding files needed to start the OS |

**Critical forensic fact:** Deleting a file does **not** erase the data immediately — only its address/pointer is removed and the space is marked "free." The actual data remains recoverable **until it's overwritten**. This is the basis of **data recovery / disk forensics**.

**HDD vs SSD for forensics:** HDDs are generally *easier* to recover deleted data from. SSDs have a **TRIM** feature that proactively clears deleted blocks in the background, making recovery much harder.

---

## Module 1.6 — Boot Process

**Sequence:**

```
Power Button Pressed
        |
        v
Firmware (BIOS/UEFI)  →  wakes up hardware
        |
        v
Hardware Initialization (POST)  →  checks CPU, RAM, disks
        |
        v
Bootloader  →  finds and loads the OS (Windows Boot Manager / GRUB)
        |
        v
OS Kernel loads  →  core of OS starts, manages hardware resources
        |
        v
System Services start  →  background programs launch
        |
        v
Login Screen
        |
        v
User Session begins
```

**Security relevance — Persistence:**
Attackers want their malicious code to **survive a restart**. Common places they "plant" themselves:
- **System Services** — registering a fake service
- **Bootloader** — advanced attack called a **bootkit**
- **Startup programs list**

SOC Analysts specifically check: *"Is there a new/unknown service or startup program?"* — a classic persistence indicator.

**Bootloader vs Kernel:** Bootloader = finds & loads the OS into RAM (temporary, one-time job). Kernel = manages hardware resources for the entire session.

---

## Module 1.7 — Operating System Fundamentals (Intro)

The OS is the manager/receptionist between hardware and users/apps.

**Five Core Responsibilities:**

| Responsibility | What It Does |
|---|---|
| **Process Management** | Decides which program gets CPU time, handles switching |
| **Memory Management** | Allocates RAM to programs, frees it when done |
| **File Management** | Organizes files/folders on disk |
| **Device Management** | Talks to hardware via **drivers** |
| **User Management** | Controls logins and **permissions** (who can do what) |

**Connection:** Process Management and Memory Management work together — when the OS gives a process CPU time to run, it must also allocate memory space for that process to operate in.

**Security relevance:** **Privilege Escalation** — an attacker's goal of moving from a normal/standard user to admin/root access without authorization. This is a core interview term.

---

## Module 1.8 — Files and File Systems

**Anatomy of a file path:**
```
C:\Users\Ahmad\Documents\report.docx
   |      |         |         |    |
 Drive  Folder    Folder    Name  Extension
```

| Term | Meaning |
|---|---|
| **Extension** | Tells you the file type (.docx, .jpg, .exe) and which program opens it |
| **Metadata** | "Data about data" — created date, modified date, owner, size (not the file's actual content) |
| **Hidden Files** | Files deliberately hidden from normal view (Windows: "Hidden" attribute; Linux: filename starts with `.`) |
| **File Attributes** | Read-only, Hidden, System, etc. |

**Common file systems:**
- **NTFS** — Windows default, supports permissions
- **FAT32** — old, simple, small USB drives
- **exFAT** — newer, large USB/SD cards
- **ext4** — Linux default

**Security relevance:**
- Metadata is a major forensic clue — e.g., a "Modified Date" earlier than the "Created Date" is a **timestamp anomaly**, suggesting the file was copied from elsewhere or its timestamps were tampered with. *(Rule: never declare something malicious without verifying — could also be a normal copy artifact.)*
- Attackers sometimes hide files using the Hidden attribute or disguise extensions (e.g. `photo.jpg.exe`).

---

## Module 1.9 — Programs, Processes & Services

```
Program (file on disk, not running)
      |  [double-click / execute]
      v
Process (running instance, loaded into RAM, using CPU/memory)
      |
      v
Thread (a smaller task running inside the process)
```

| Term | Meaning |
|---|---|
| **Program** | A file on disk, not yet running (e.g. `chrome.exe`) |
| **Process** | The program actually running in RAM, using CPU/memory |
| **Thread** | A sub-task running inside a process (parallel work) |
| **Service** | A special process that runs silently in the background, often starting at boot |
| **Foreground Process** | The one you're actively interacting with |
| **Background Process** | Running but not directly visible/interacted with |

**Real example:** `chrome.exe` on disk = Program → double-click loads it into RAM = Process → each tab may run as its own thread/sub-process (isolation: one tab crashing doesn't crash the whole browser).

**Security relevance — Process Masquerading:** Attackers name their malicious process to closely resemble a legitimate one (e.g. `scvhost.exe` instead of the real `svchost.exe`) to avoid detection in Task Manager. Analysts must carefully check exact process names.

---

## Module 1.10 — Users & Permissions

| Term | Meaning |
|---|---|
| **User Account** | An individual identity on the system (username, password, own files/settings) |
| **Administrator (Windows) / Root (Linux)** | Full control account — can install software, change system settings |
| **Standard User** | Limited permissions — own files/apps only |
| **Groups** | A bundle of users sharing the same permissions — set once, apply to many |
| **Permissions** | Rules defining who can Read / Write / Execute |
| **Least Privilege Principle** | Give each user/process only the access they actually need — nothing more |

**Why Least Privilege matters:** If every employee had admin access "for convenience," one compromised account could give an attacker control of the entire company system. Minimal permissions = minimal blast radius if compromised.

**Security relevance:** Privilege Escalation (from Module 1.7/1.6) becomes much harder for an attacker in an environment that properly enforces Least Privilege.

---

## Module 1.11 — System Monitoring Basics

| Tool | Purpose |
|---|---|
| **Task Manager** (`Ctrl+Shift+Esc`) | Real-time view: running processes, CPU/RAM/Disk usage, startup programs |
| **Resource Monitor** | Deeper view — which process uses which file/network connection |
| **Device Manager** | Lists connected hardware and their drivers |
| **Event Viewer** | Logs of past system events — **critical for SOC investigation work** |
| **System Information** (`msinfo32`) | Full hardware/OS summary |
| **Command Prompt / PowerShell** | Text-based system control; PowerShell is more powerful, security/automation-focused |

**Key distinction:** Task Manager shows what's happening **right now** (a snapshot). Event Viewer shows **history** — what happened over time (e.g. failed login attempts 2 hours ago). Investigations almost always require looking *backward*, which is why Event Viewer logs matter so much.

---

## Module 1.12 — Troubleshooting Fundamentals

**The Structured Investigation Cycle:**

```
1. Observe          →  Notice something is wrong
2. Gather Evidence   →  Collect facts, logs, data (don't guess)
3. Form Hypothesis   →  Make an educated guess based on evidence
4. Test              →  Check if the hypothesis is correct
5. Verify            →  Confirm the problem is actually fixed
6. Document          →  Write down what happened, for future reference
```

**Why this matters:** This is the same disciplined, evidence-based approach used in real **Incident Response** (Identify → Contain → Eradicate → Recover → Lessons Learned). Never skip straight to "Test" without gathering evidence first — you'll be testing a random guess, not an informed hypothesis.

**Why "Document" matters long-term:** Creates institutional memory — a future analyst (or future you) can solve the same issue in minutes instead of hours by referencing past documentation.

---

## Module 1.13 — Security Perspective (Recap Through the Attacker's Eyes)

Same six areas from this block, now viewed through an attacker's lens:

| Area | Attacker's Angle |
|---|---|
| **Memory** | Fileless malware hides here — leaves no disk trace |
| **Files** | Hidden/renamed files disguise malicious payloads |
| **Processes** | Process masquerading — naming malware like a legitimate process |
| **Users** | Privilege escalation — standard user → admin/root |
| **Startup** | Persistence — surviving a restart via services/bootloader |
| **Storage** | Hiding evidence; exploiting the fact that deleted data is often still recoverable |

**Biggest takeaway of Block 1:** Cybersecurity is fundamentally built on strong computer fundamentals. You cannot recognize what's *abnormal* (malicious) until you deeply understand what's *normal*.

**Connection point:** Startup + Processes work together for persistence — malware is planted at startup (service/bootloader) *and* disguised as a legitimate-looking process so it doesn't get noticed once it's running.

---

## Hands-On Labs (Completed on Ahmad's Own PC)

**Test Machine:** Lenovo ThinkBook 14 G6 IRL | Windows 11 Enterprise (Build 26200) | Intel i7-1355U (10 cores / 12 logical processors) | 8 GB RAM | UEFI boot | Secure Boot: On

### Lab 1 — System Information (`msinfo32`)
Identified real OS version, CPU, RAM, BIOS version, and disk/partition layout.
- **Key finding:** Only 1.17 GB RAM free of 8 GB → system actively relying on the **page file** (virtual memory) — directly ties to Module 1.4.
- **Key finding:** UEFI + Secure Boot On + Credential Guard/VBS running — modern security features that block bootkits and protect stored credentials, ties to Module 1.6.
- 5 partitions identified, including a small ~260MB likely EFI/boot partition and a ~1.95GB recovery partition.

### Lab 2 — Task Manager
- **Chrome (18 processes)** — one process per tab/extension → isolation benefit: one tab crashing doesn't crash the whole browser (Module 1.9 in action).
- **Antimalware Service Executable, Search, Office Click-to-Run** identified as top resource users.
- Disabled all Startup Apps except Chrome (noted "High" startup impact).
- **Services tab:** Confirmed that "Stopped" services have no PID (not loaded into RAM) — a live demonstration of the **Program vs Process** distinction from Module 1.9.

### Lab 3 — File Explorer
- Enabled "File name extensions" — confirmed Windows hides extensions by default (security risk: enables tricks like `photo.jpg.exe`).
- Enabled "Hidden items" — found system folders `PerfLogs`, `ProgramData`, `Recovery`.
- Checked file/folder Properties — learned that the "Read-only" checkbox behaves differently on **folders** (often just a UI quirk) vs. **files** (a real, enforced restriction).

---

## Common Beginner Mistakes (Avoid These)

- ❌ Confusing programs with processes
- ❌ Believing RAM permanently stores files
- ❌ Thinking the CPU executes many instructions *simultaneously* (it's fast switching, not true simultaneity, beyond the number of cores)
- ❌ Ignoring metadata
- ❌ Confusing firmware with the operating system
- ❌ Assuming every background process is automatically malicious

---

## Must-Master Checklist (Before Block 2)

- [x] Explain how a computer starts (boot process)
- [x] Explain the role of CPU, RAM, Storage, and OS
- [x] Distinguish hardware, software, firmware, programs, processes, services
- [x] Navigate basic Windows system information tools
- [x] Understand where common evidence may exist
- [x] Describe execution flow from boot to user login
- [x] Apply a structured troubleshooting mindset

**Status: ✅ Block 1 Complete (13 Modules + 3 Labs)**

---

## Connection to Next Block

Block 1 explained: **"How a computer works."**

Block 2 explains: **"How to safely build virtual computers for learning."**

Virtualization provides the isolated environment where nearly all future labs (malware analysis, incident simulation, Windows/Linux practice, networking) will actually take place.
