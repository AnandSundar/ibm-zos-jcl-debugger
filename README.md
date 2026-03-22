# 🎸 ibm-zos-jcl-debugger

> **From CC 0008 to CC 0000 — debugging enterprise JCL on IBM z/OS so production never stops.**

<div align="center">

[![IBM Z Xplore](https://img.shields.io/badge/IBM%20Z%20Xplore-Advanced-blue?style=for-the-badge&logo=ibm&logoColor=white)](https://ibmzxplore.influitive.com/)
[![Platform](https://img.shields.io/badge/Platform-IBM%20z%2FOS-red?style=for-the-badge&logo=ibm)](https://www.ibm.com/products/zos)
[![Language](https://img.shields.io/badge/Language-JCL%20%2F%20REXX-orange?style=for-the-badge)](https://www.ibm.com/docs/en/zos)
[![Status](https://img.shields.io/badge/Job%20Status-CC%200000%20✅-brightgreen?style=for-the-badge)](https://github.com)
[![IDE](https://img.shields.io/badge/IDE-VS%20Code-007ACC?style=for-the-badge&logo=visual-studio-code)](https://code.visualstudio.com/)

<img src="docs/images/ibm_z_hero.png" width="600" alt="IBM Z Xplore Banner"/>

</div>

---

## 🤔 What Is This? (Plain English)

> Imagine you have a giant, ultra-reliable computer — so powerful it runs the banking systems, airline reservations, and healthcare networks that billions of people depend on every single day. That computer is an **IBM Mainframe**, and the language used to tell it *what jobs to run and what files to use* is called **JCL (Job Control Language)**.

This project is about **finding and fixing a bug** in a JCL program on a real IBM mainframe (z/OS). The broken program was supposed to combine three data files — guitar nicknames, brand names, and artist names — into one neat report called "Legendary Guitars." But it was missing a critical file instruction, causing it to crash with an error code.

**I found the bug, wrote the fix, and got the job running perfectly — from `CC 0008` (error) to `CC 0000` (success).** ✅

---

## 📺 The Big Picture

```
 ┌─────────────────────────────────────────────────────┐
 │              IBM Mainframe (z/OS)                   │
 │                                                     │
 │  📂 ZXP.PUBLIC.SOURCE                               │
 │     ├── NICKNAME  ──┐                               │
 │     ├── BRAND    ───┼──► COMBINE Program ──► 🎸 GUITAR Report
 │     └── ARTIST   ──┘  (IRXJCL + REXX)              │
 │                                                     │
 │  JES2 Job Scheduler runs everything                 │
 └─────────────────────────────────────────────────────┘
```

---

## 🗺️ How the Job Works — Step by Step

```mermaid
flowchart TD
    A([🧑‍💻 Developer in VS Code]) --> B[Copy JES2JOB1 from Public Library]
    B --> C[Paste into Personal JCL Dataset]
    C --> D[Review the JCL Code]
    D --> E[Submit Job to JES2 Scheduler]
    E --> F{Check Condition Code}
    F -->|CC 0008 — Something is wrong| G[Read JESYSMSG Error Log]
    G --> H[Discover: Missing ARTIST DD Statement]
    H --> I[Add the Missing Line of JCL]
    I --> J[Save Changes and Resubmit]
    J --> K{Check Condition Code Again}
    K -->|CC 0000 — All good!| L[🎸 GUITAR Report Generated]
    K -->|Still failing| I
    L --> M([✅ Challenge Complete!])

    style A fill:#1565C0,color:#fff,stroke:#0D47A1
    style M fill:#2E7D32,color:#fff,stroke:#1B5E20
    style F fill:#E65100,color:#fff,stroke:#BF360C
    style K fill:#E65100,color:#fff,stroke:#BF360C
    style H fill:#B71C1C,color:#fff,stroke:#7F0000
    style L fill:#1565C0,color:#fff,stroke:#0D47A1
```

---

## 🔍 The Debugging Process

```mermaid
flowchart LR
    A[Submit Job] --> B[CC 0008 Error]
    B --> C[Open JESYSMSG Log]
    C --> D[Error: IRX0670E
EXECIO GET/PUT Failure]
    D --> E{What is missing?}
    E --> F[REXX COMBINE script
needs 3 inputs]
    F --> G[JCL only has 2:
NICKNAME + BRAND]
    G --> H[Missing: ARTIST DD Statement]
    H --> I[Add ARTIST DD Statement]
    I --> J[Resubmit]
    J --> K[CC 0000 ✅ Success]

    style B fill:#c62828,color:#fff
    style K fill:#2e7d32,color:#fff
    style H fill:#f57f17,color:#fff
    style D fill:#6a1b9a,color:#fff
```

---

## 👣 Walkthrough With Screenshots

### Step 1 — Find the Job in the Public Library

Navigate to `ZXP.PUBLIC.JCL` and locate the member `JES2JOB1`. Right-click and select **Copy**.

<img src="docs/images/step1_copy_dataset.png" width="700" alt="Step 1 - Copy JES2JOB1 from public library"/>

---

### Step 2 — Make It Your Own

Paste the member into your personal JCL dataset so you can edit it without affecting the shared library.

<img src="docs/images/step2_paste_menu.png" width="500" alt="Step 2 - Paste into personal dataset"/>

---

### Step 3 — Review the Broken JCL

Open the member and read through the code. It looks like it runs IRXJCL with a COMBINE parameter — but something critical is missing.

<img src="docs/images/step3_jcl_code.png" width="750" alt="Step 3 - Review the JCL code"/>

---

### Step 4 — Submit the Job (Watch It Fail)

Right-click your copy of JES2JOB1 and select **Submit Job**. Note the job number that appears in the bottom-right corner.

<img src="docs/images/step4_submit_job_list.png" width="700" alt="Step 4 - Submit the job"/>

---

### Step 5 — Check the Output: CC 0008 Error

Go to the **JOBS** section in VS Code. You'll see the job completed with **CC 0008** — that's an error code. Something broke.

<img src="docs/images/step5_jobs_cc0008.png" width="600" alt="Step 5 - Job shows CC 0008 error"/>

> 🚨 **CC 0008 = Error.** The job ran but something went wrong. Time to investigate.

---

### Step 6 — Read the Error Log (JESYSMSG)

Expand the job output and look at `JESYSMSG`. There is one very clear clue buried in the output.

<img src="docs/images/step6_jesysmsg_error.png" width="750" alt="Step 6 - JESYSMSG shows the error"/>

> 🔎 **The error is:** `IRX0670E EXECIO error while trying to GET or PUT a record`
> This means the REXX script tried to read a file that wasn't defined in the JCL.

---

### Step 7 — Fix the JCL (Add the Missing Line)

Go back to your JCL and add the missing `//ARTIST` DD statement. Copy the pattern from the `BRAND` line and change the DD name and dataset member.

<img src="docs/images/step7_jcl_fix.png" width="750" alt="Step 7 - Add the missing ARTIST DD statement"/>

> ✅ **The fix is one line:** `//ARTIST DD DSN=ZXP.PUBLIC.SOURCE(ARTIST),DISP=SHR`

---

### Step 8 — Save and Resubmit

**Always save before resubmitting.** If you forget to save, it will run the old broken version again. Right-click → Submit Job.

<img src="docs/images/step8_submit_fixed.png" width="500" alt="Step 8 - Submit the fixed job"/>

---

### Step 9 — CC 0000: Success! 🎉

Check the JOBS section again. This time you should see **CC 0000** — no errors. The GUITAR report has been written to your OUTPUT dataset.

<img src="docs/images/step9_guitar_output.png" width="750" alt="Step 9 - GUITAR output generated successfully"/>

---

## 🛠️ Technical Specifications

| Property | Value |
|---|---|
| **Platform** | IBM z/OS (Mainframe Operating System) |
| **Job Scheduler** | JES2 (Job Entry Subsystem 2) |
| **Program Executed** | `IRXJCL` — IBM's REXX interpreter for batch jobs |
| **Script Language** | REXX (Restructured Extended Executor) |
| **Parameter** | `PARM='COMBINE'` — triggers the COMBINE REXX routine |
| **IDE / Editor** | Visual Studio Code with IBM Z Open Editor extension |
| **Input Files** | `ZXP.PUBLIC.SOURCE(NICKNAME)`, `(BRAND)`, `(ARTIST)` |
| **Output Dataset** | `<USERID>.OUTPUT(GUITAR)` |
| **Error Before Fix** | `CC 0008` — IRX0670E EXECIO error |
| **Status After Fix** | `CC 0000` — Job completed successfully |
| **Challenge Level** | Advanced (IBM Z Xplore) |
| **Estimated Time** | 45 minutes |

---

## 📄 The JCL Code — Before & After

### ❌ Broken Version (CC 0008)

```jcl
//JES2JOB1 JOB
//COMBINE  EXEC PGM=IRXJCL,PARM='COMBINE'
//SYSEXEC  DD DSN=ZXP.PUBLIC.EXEC,DISP=SHR
//SYSTSPRT DD DSN=&SYSUID..OUTPUT(GUITAR),DISP=SHR
//SYSTSIN  DD DUMMY
//NICKNAME DD DSN=ZXP.PUBLIC.SOURCE(NICKNAME),DISP=SHR
//BRAND    DD DSN=ZXP.PUBLIC.SOURCE(BRAND),DISP=SHR
```
> 🚨 **The ARTIST data file is missing!** The COMBINE program expects three inputs but only two are provided.

---

### ✅ Fixed Version (CC 0000)

```jcl
//JES2JOB1 JOB
//COMBINE  EXEC PGM=IRXJCL,PARM='COMBINE'
//SYSEXEC  DD DSN=ZXP.PUBLIC.EXEC,DISP=SHR
//SYSTSPRT DD DSN=&SYSUID..OUTPUT(GUITAR),DISP=SHR
//SYSTSIN  DD DUMMY
//NICKNAME DD DSN=ZXP.PUBLIC.SOURCE(NICKNAME),DISP=SHR
//BRAND    DD DSN=ZXP.PUBLIC.SOURCE(BRAND),DISP=SHR
//ARTIST   DD DSN=ZXP.PUBLIC.SOURCE(ARTIST),DISP=SHR   ← Added this line
```
> ✅ **One line. That's all it took.** The ARTIST DD statement tells the job where to find the artist names.

---

## 📋 DD Statement Breakdown (What Each Line Does)

> DD = **Data Definition** — it's how you tell the mainframe "here's a file the program needs"

| DD Name | Dataset Location | Purpose |
|---|---|---|
| `SYSEXEC` | `ZXP.PUBLIC.EXEC` | Where the REXX scripts live (the program code) |
| `SYSTSPRT` | `&SYSUID..OUTPUT(GUITAR)` | Where the final output report gets written |
| `SYSTSIN` | `DUMMY` | Placeholder — no interactive input needed |
| `NICKNAME` | `ZXP.PUBLIC.SOURCE(NICKNAME)` | File containing guitar nicknames (Blackie, Lucille, etc.) |
| `BRAND` | `ZXP.PUBLIC.SOURCE(BRAND)` | File containing guitar brand/model names |
| `ARTIST` | `ZXP.PUBLIC.SOURCE(ARTIST)` | ✅ **The missing file** — guitar owners/artists |

---

## 🚦 Understanding Condition Codes

Think of Condition Codes like the **check engine light** in your car:

| Condition Code | Meaning | Plain English |
|---|---|---|
| `CC 0000` | ✅ **Success** | Everything ran perfectly, no problems |
| `CC 0004` | ⚠️ **Warning** | Job finished but something was slightly off |
| `CC 0008` | ❌ **Error** | Something went wrong — job may not have worked |
| `CC 0012` | 🚨 **Severe Error** | Serious failure — results cannot be trusted |
| `CC 0016` | 💀 **Critical** | System-level failure, job aborted |

---

## 🎸 The Final Output — Legendary Guitars Report

| # | Guitar Nickname | Brand | Artist |
|---|---|---|---|
| 1 | **Blackie** | Fender Strat. (custom) | Eric Clapton |
| 2 | **Cloud** | Custom build | Prince |
| 3 | **Lucille** | Gibson (almost burned) | B.B. King |
| 4 | **Frankenstrat** | Fender/Gibson Frankencaster | Eddie Van Halen |
| 5 | **Monterey Stratocaster** | Fender Strat. Flower Power | Jimi Hendrix |
| 6 | **Concorde** | Jackson Flying V | Randy Rhoads |

---

## 🏗️ System Architecture

```mermaid
graph TD
    subgraph VSCode["💻 VS Code + IBM Z Open Editor"]
        dev[Developer Workspace]
        zowe[Zowe Explorer Extension]
    end

    subgraph zOS["🖥️ IBM z/OS Mainframe"]
        jes2[JES2 Job Scheduler]
        irxjcl[IRXJCL — REXX Interpreter]
        rexx[COMBINE REXX Script]

        subgraph datasets["📂 Datasets"]
            exec[ZXP.PUBLIC.EXEC]
            nick[SOURCE: NICKNAME]
            brand[SOURCE: BRAND]
            artist[SOURCE: ARTIST]
            output[OUTPUT: GUITAR]
        end
    end

    dev --> zowe
    zowe -->|Submit JCL Job| jes2
    jes2 --> irxjcl
    irxjcl --> rexx
    exec --> rexx
    nick --> rexx
    brand --> rexx
    artist --> rexx
    rexx --> output

    style VSCode fill:#1a1a2e,color:#fff,stroke:#4a4a8a
    style zOS fill:#0d2137,color:#fff,stroke:#1565C0
    style datasets fill:#0a3d62,color:#fff,stroke:#1565C0
```

---

## 🔐 Security Features of IBM z/OS

| Security Feature | What It Does | Real-World Impact |
|---|---|---|
| **RACF** | Controls who can access which datasets and programs | Only authorized staff can touch transaction files |
| **Crypto Express Hardware** | On-chip encryption — keys never leave the hardware | Payment card data encrypted at silicon level |
| **Pervasive Encryption** | Encrypts all data at rest and in motion by default | No extra config needed — everything is always encrypted |
| **SMF Audit Logging** | Every action logged with timestamp and user ID | Perfect for HIPAA, PCI-DSS, SOX compliance |
| **Workload Isolation** | Jobs run in completely isolated memory spaces | One crashing job cannot corrupt another job's data |
| **DISP=SHR / EXCL** | Fine-grained file access control in JCL | Prevents two jobs writing to the same file simultaneously |

> 💡 Every `DISP=SHR` in this project ensures source files are read-only — no accidental overwrites possible.

---

## 🧰 Tools & Technologies

```mermaid
mindmap
  root((JCL2 Project))
    IBM Mainframe
      z/OS Operating System
      JES2 Scheduler
      IRXJCL Interpreter
      REXX Scripting
    Development Tools
      VS Code
      IBM Z Open Editor
      Zowe Explorer
    Concepts Mastered
      JCL Syntax
      DD Statements
      Dataset Management
      Error Code Analysis
      Batch Job Debugging
    Output
      GUITAR Dataset
      Legendary Guitars Report
```

---

## 💡 Key Concepts Explained Simply

<details>
<summary>📦 <strong>What is a Dataset?</strong></summary>

On a mainframe, files are called **datasets**. They're organized with names like `ZXP.PUBLIC.SOURCE(ARTIST)` — think of it like a folder path: the library is `ZXP.PUBLIC.SOURCE` and the specific file inside is `ARTIST`. The brackets `()` refer to members inside a library.

</details>

<details>
<summary>⚙️ <strong>What is REXX?</strong></summary>

**REXX** is a scripting language built into z/OS. The `COMBINE` script reads three input files (NICKNAME, BRAND, ARTIST) and merges them into one formatted report. Think of it as a shell script — but for mainframes, and 40+ years old.

</details>

<details>
<summary>🗂️ <strong>What is JES2?</strong></summary>

**JES2** is the mainframe's job scheduler. When you submit a JCL job, JES2 queues it, runs it, collects all output, and stores the results. It's like a highly reliable CI/CD runner — except it was invented in the 1970s and still powers critical infrastructure today.

</details>

<details>
<summary>🔎 <strong>What was the actual error?</strong></summary>

```
IRX0670E EXECIO error while trying to GET or PUT a record
```

This REXX error means: "I tried to read from a file that wasn't defined in the JCL." The COMBINE script looked for the ARTIST file, but no DD statement pointed to it — so it crashed. The fix was one line.

</details>

---

## 📊 Error Resolution Summary

| Metric | Before Fix | After Fix |
|---|---|---|
| **Condition Code** | `CC 0008` ❌ | `CC 0000` ✅ |
| **Job Status** | Failed | Completed Successfully |
| **GUITAR Output** | Not generated | Generated ✅ |
| **Error Message** | IRX0670E EXECIO failure | None |
| **DD Statements** | 6 (missing ARTIST) | 7 (complete) |
| **Lines Changed** | — | 1 line added |

---

## 🧠 What This Demonstrates About Me

| Skill | Why It Matters to Your Team |
|---|---|
| **Reading unfamiliar error logs** | I can debug systems I didn't build — critical for on-call and incidents |
| **Mainframe / JCL fluency** | Fewer than 5% of developers can work on z/OS production systems |
| **Batch architecture knowledge** | I understand enterprise job schedulers, not just web APIs |
| **Precision under pressure** | One missing line caused the failure. I found it. Spacing matters on mainframes |
| **IBM Z Xplore certified** | Validated by IBM — not self-taught guesswork |
| **End-to-end ownership** | I copied the job, debugged it, fixed it, and validated the output |

---

## 🏆 Certification

```
╔══════════════════════════════════════════════════════╗
║          IBM Z Xplore — Advanced Badge               ║
║                                                      ║
║  Challenge: JCL2 — Fix JCL and Get That Job Running  ║
║  Level:     Advanced                                 ║
║  Status:    ✅ Completed (CC 0000)                    ║
║  Platform:  IBM z/OS                                 ║
╚══════════════════════════════════════════════════════╝
```

---

## 🗂️ Repository Structure

```
ibm-zos-jcl-debugger/
│
├── README.md                      ← You are here
├── JCL/
│   ├── JES2JOB1_broken.jcl        ← Original broken JCL (CC 0008)
│   └── JES2JOB1_fixed.jcl         ← Fixed JCL with ARTIST DD (CC 0000)
├── OUTPUT/
│   └── GUITAR.txt                 ← Final Legendary Guitars report
└── docs/
    ├── JCL2_assignment.pdf        ← Original IBM Z Xplore challenge brief
    └── images/
        ├── ibm_z_hero.png
        ├── step1_copy_dataset.png
        ├── step2_paste_menu.png
        ├── step3_jcl_code.png
        ├── step4_submit_job_list.png
        ├── step5_jobs_cc0008.png
        ├── step6_jesysmsg_error.png
        ├── step7_jcl_fix.png
        ├── step8_submit_fixed.png
        └── step9_guitar_output.png
```

---

## 🤝 Let's Connect

If your team works with mainframes, cloud infrastructure, or backend systems — and you need an engineer who can **own a problem from error log to fix** — I'd love to talk.

[![LinkedIn](https://img.shields.io/badge/LinkedIn-Connect-0A66C2?style=for-the-badge&logo=linkedin)](https://linkedin.com/in/anandsundar96)
[![GitHub](https://img.shields.io/badge/GitHub-Portfolio-181717?style=for-the-badge&logo=github)](https://github.com/anandsundar)

---

<div align="center">

*Built with 🖤 on IBM z/OS | IBM Z Xplore Advanced Certification*

*"On a mainframe, one missing line can bring down a job. Finding it is the job."*

</div>
