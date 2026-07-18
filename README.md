# Conducting Forensic Investigations on Linux Systems

<p align="center">
  <strong>Live Linux Analysis, Log Examination, File-System Repair, and E3 Drive-Image Investigation</strong>
</p>

---

## Objective

The objective of this lab was to investigate a live Linux system and a forensic image of a Linux server for evidence of unauthorized access, suspicious software installation, physical USB activity, printing, and disk imaging.

The lab combined Linux command-line analysis with Paraben E3 examination of an Ext4 drive image. The investigation included reviewing key Linux directories, examining system and authentication logs, repairing a damaged file system, identifying disguised files, analyzing successful and failed SSH logins, and locating evidence of potential data exfiltration.

## Skills Demonstrated

- Navigating the Linux file system through the command line
- Reviewing `/bin`, `/etc`, `/var`, and `/proc`
- Examining boot and kernel messages with `dmesg`
- Checking and repairing an Ext4 file system with `fsck`
- Reviewing Linux shell history
- Enumerating running processes with `ps`
- Detecting a misleading file extension with the `file` command
- Examining `kern.log` and `auth.log`
- Importing and examining a Linux drive image with Paraben E3
- Searching authentication logs for failed and successful login activity
- Identifying packages installed through `apt-get`
- Detecting a Linux keylogger and remote-access software
- Identifying USB storage activity and device serial information
- Reviewing CUPS printer logs
- Finding evidence of disk imaging with the `dd` command

## Lab Environment

| Component | Details |
|---|---|
| Linux System | `TargetLinux01` |
| Linux Distribution | Ubuntu |
| Windows Workstation | `vWorkstation` |
| Windows Operating System | Windows Server 2019 |
| Forensic Tool | Paraben E3 |
| Evidence Image | `Ubuntu20.04.image` |
| Evidence Location | `C:\Linux Forensics` |
| File System | Ext4 |
| Investigation Type | Linux intrusion, insider threat, and possible data exfiltration |

## Tools and Commands Used

| Tool or Command | Purpose |
|---|---|
| **Linux Terminal** | Performed live system and file-system analysis |
| **Paraben E3** | Imported and examined the Linux forensic image |
| `ls -l` | Displayed directory contents, permissions, owners, sizes, and timestamps |
| `dmesg` | Displayed boot, kernel, hardware, and security-related messages |
| `fsck` | Checked and repaired the Ext4 file system |
| `history` | Displayed previously executed shell commands |
| `ps -aux` | Displayed running processes and resource information |
| `file` | Identified a file’s true format regardless of its extension |
| `tail` | Displayed recent kernel-log records |
| `more` | Reviewed authentication-log records |
| E3 Advanced Search | Located login attempts, installations, USB activity, and disk-imaging commands |

## Investigation Scenario

In the first section, I examined a live Ubuntu system to identify key Linux directories, active processes, system activity, file system errors, and log records.

In the second section, I acted as a junior forensic investigator reviewing a Linux drive image from Harper and Associates. The organization suspected that an intruder had attempted to access a critical Linux server using legitimate user credentials. The analysis later identified evidence of suspicious software installation, successful unauthorized access, USB storage activity, printing, and possible disk-image exfiltration.

---

# Section 1: Hands-On Demonstration

## Part 1: Explore a Live Linux System

### Examining the `/bin` Directory

I navigated to the `/bin` directory and used a long listing to review executable files, symbolic links, permissions, owners, sizes, and timestamps.

```bash
cd /bin
ls -l
```

The `/bin` directory contains essential system commands and executable programs. Reviewing this directory can help identify altered binaries, unauthorized tools, or suspicious programs.

<p align="center">
  <img src="https://github.com/user-attachments/assets/9d8e0655-2a25-4445-acdf-161022b6eb7b"
       alt="Linux forensic investigation evidence"
       width="800">
</p>

<p align="center">
  <em>Figure 1: Contents of the `/bin` directory displayed with `ls -l`.</em>
</p>

### Examining the `/etc` Directory

I reviewed the `/etc` directory, which contains system-wide configuration files.

```bash
cd /etc
ls -l
```

Configuration files can provide evidence of altered services, user settings, network configuration, persistence mechanisms, or application tampering.

<p align="center">
  <img src="https://github.com/user-attachments/assets/30ef6aee-d0b5-4724-95b9-2a4acfc62531"
       alt="Linux forensic investigation evidence"
       width="800">
</p>

<p align="center">
  <em>Figure 2: System configuration files and directories located under `/etc`.</em>
</p>

### Examining the `/var` Directory

I reviewed the `/var` directory, which contains changing system data such as logs, caches, temporary data, and application records.

```bash
cd /var
ls -l
```

<p align="center">
  <img src="https://github.com/user-attachments/assets/a6b82280-4897-4f84-a0ce-b0cdaef22a2c"
       alt="Linux forensic investigation evidence"
       width="800">
</p>

<p align="center">
  <em>Figure 3: Variable system data and log-related directories located under `/var`.</em>
</p>

### Examining the `/proc` Directory

I reviewed the `/proc` directory, which is a virtual file system containing live process and kernel information stored in memory.

```bash
cd /proc
ls -l
```

The numbered directories under `/proc` correspond to active process IDs, while other files expose system and kernel information.

<p align="center">
  <img src="https://github.com/user-attachments/assets/757297ee-0be9-4d32-8c27-afab090901f2"
       alt="Linux forensic investigation evidence"
       width="800">
</p>

<p align="center">
  <em>Figure 4: Live process and system information available through the `/proc` virtual file system.</em>
</p>

---

## Part 2: Use Linux Shell Commands for Forensic Investigations

### Reviewing Kernel and Boot Messages

I used `dmesg` to review messages generated during system startup and hardware initialization.

```bash
dmesg
```

The output included kernel, AppArmor, service, and device-related records that could help identify failed services, suspicious startup activity, or hardware events.

<p align="center">
  <img src="https://github.com/user-attachments/assets/245085fb-b7e4-45b3-9077-4f5d5321e5be"
       alt="Linux forensic investigation evidence"
       width="800">
</p>

<p align="center">
  <em>Figure 5: Kernel and boot-related messages displayed with the `dmesg` command.</em>
</p>

### Checking and Repairing the Ext4 File System

I checked the Ext4 file system on `/dev/sdb5`:

```bash
sudo fsck /dev/sdb5
```

The initial check identified an invalid journal superblock. I stopped the interactive check and used preen mode to automatically repair issues that could be safely corrected:

```bash
sudo fsck /dev/sdb5 -p
```

I then ran the standard check again:

```bash
sudo fsck /dev/sdb5
```

The final result showed the file system as clean after the journal was recreated.

<p align="center">
  <img src="https://github.com/user-attachments/assets/f6d443cf-002f-4ca7-809e-f42a2713dcfd"
       alt="Linux forensic investigation evidence"
       width="800">
</p>

<p align="center">
  <em>Figure 6: `fsck` results showing repair of the damaged journal and successful file-system verification.</em>
</p>

### Reviewing Shell Command History

I reviewed commands that had previously been executed on the Linux system:

```bash
history
```

Command history can reveal file deletion, copying, package installation, disk manipulation, or other activity relevant to an investigation.

<p align="center">
  <img src="https://github.com/user-attachments/assets/2e383c1f-0972-4d13-93fa-6f09590223db"
       alt="Linux forensic investigation evidence in Paraben's E3"
       width="800">
</p>

<p align="center">
  <em>Figure 7: Shell history showing commands previously executed on the Linux system.</em>
</p>

### Reviewing Running Processes

I displayed active processes with:

```bash
ps -aux
```

The output included each process owner, PID, CPU use, memory use, start time, state, and command.

<p align="center">
  <img src="https://github.com/user-attachments/assets/247e93b4-f045-4490-b4c3-23ed6d9ad851"
       alt="Linux forensic investigation evidence in Paraben's E3"
       width="800">
</p>

<p align="center">
  <em>Figure 8: Active Linux processes displayed with `ps -aux`.</em>
</p>

### Identifying a Disguised File

I navigated to the user’s Documents directory and examined `MyScheduler.txt`:

```bash
cd /home/user/Documents
ls
file MyScheduler.txt
```

The `file` command determined that `MyScheduler.txt` was actually JPEG image data rather than a text document. This demonstrated how an extension can be changed to conceal a file’s true format.

<p align="center">
  <img src="https://github.com/user-attachments/assets/a23e57de-b799-4812-9066-db5ec5b045bb"
       alt="Linux forensic investigation evidence in Paraben's E3"
       width="800">
</p>

<p align="center">
  <em>Figure 9: The `file` command identifying `MyScheduler.txt` as JPEG image data.</em>
</p>

---

## Part 3: Retrieve Log Files on a Live Linux System

### Reviewing `kern.log`

I navigated to the Linux log directory and reviewed recent kernel records:

```bash
cd /var/log
ls
sudo tail -f kern.log
```

The kernel log can contain hardware, driver, AppArmor, startup, and external-device activity.

<p align="center">
  <img width="900"
       alt="Linux kern log records displayed in the terminal"
       src="PASTE-KERN-LOG-IMAGE-URL-HERE" />
</p>

<p align="center">
  <em>Figure 10: Recent kernel records displayed from `kern.log`.</em>
</p>

### Reviewing `auth.log`

I reviewed authorization and authentication activity with:

```bash
sudo more -f auth.log
```

The authentication log records login attempts, `sudo` activity, opened and closed sessions, and commands executed with elevated privileges.

<p align="center">
  <img width="900"
       alt="Linux auth log authentication and sudo records displayed in the terminal"
       src="PASTE-AUTH-LOG-IMAGE-URL-HERE" />
</p>

<p align="center">
  <em>Figure 11: Authentication, session, and elevated-command records displayed from `auth.log`.</em>
</p>

---

# Section 2: Applied Learning

## Part 1: Identify Login Attempts on a Linux Drive Image

I created an E3 case and imported:

```text
C:\Linux Forensics\Ubuntu20.04.image
```

I navigated to the Linux log directory in the Ext4 partition:

```text
Ubuntu20.04
└── Partition Parser
    └── Partition1
        └── EXT4
            └── Root
                └── var
                    └── log
```

I examined `auth.log` and used E3 Advanced Search to identify failed and successful login activity.

### Failed Login Findings

| User | Failed Attempts | Source IP | Source Ports | Time Ranges |
|---|---:|---|---|---|
| `noel` | 12 | `192.168.78.1` | `14444`, `3521` | Jun 11, 00:57:08-00:57:34 and 05:06:33-05:06:50 |
| `dominic` | 11 | `192.168.78.1` | `4663`, `3417` | Jun 11, 05:07:29-05:07:57 and 05:36:44-05:38:32 |

The variable intervals between the failed attempts were more consistent with manual activity than a rapid automated brute-force attack.

### Successful Login Finding

The most recent successful login identified for `dominic` occurred on:

```text
Jun 11 at 04:48:18
```

No successful session was found for `noel`.

---

## Part 2: Identify Software Installations on a Linux Drive Image

I searched `auth.log` for:

```text
apt-get install
```

The records showed installation commands executed by the user `dominic`.

| Package | Assessment |
|---|---|
| `logkeys` | Suspicious Linux keylogger capable of recording keyboard input |
| `openssh-server` | Provided remote SSH access and was relevant because remote login attempts were present |
| `autotools-dev` | Development support package |
| `build-essential` | Compilation toolchain commonly used to build software |
| `autoconf` | Used to configure software for compilation |
| `kbd` | Keyboard and console utility package |
| `acct` | Process and login accounting package |

The strongest suspicious finding was `logkeys`, which could have been used to capture legitimate credentials. The installation of supporting development tools may indicate that software was compiled or configured directly on the server.

---

## Part 3: Identify External Drive Attachments

I searched `kern.log` for the phrase:

```text
USB mass storage device detected
```

The investigation identified the following USB device:

| Field | Finding |
|---|---|
| Connection Time | Jun 10 at 10:24:12 |
| Device Name | `USB DISK` |
| Serial Number | `FBI1405291710` |

The USB connection occurred before the suspicious login activity and supported the possibility of physical access to the server.

---

# Section 3: Challenge and Analysis

## Part 1: Identify Recently Printed Files

I researched Linux printing records and examined the CUPS log directory in E3:

```text
/var/log/cups
```

I opened `page_log` in Document View. The record showed that the user `dominic` generated a print job on June 10, 2021, at approximately 10:30:36.

<p align="center">
  <img width="900"
       alt="Linux CUPS page log displayed in Paraben E3 Document View"
       src="PASTE-PRINTER-LOG-IMAGE-URL-HERE" />
</p>

<p align="center">
  <em>Figure 12: CUPS `page_log` record showing printing activity associated with the user `dominic`.</em>
</p>

---

## Part 2: Identify Disk Imaging

I searched the `/var/log` folder for records containing:

```text
/usr/bin/dd
```

The search located evidence of the following disk-copy command:

```bash
sudo dd if=/dev/sda of=/media/dominic/data/Ubuntu20.04.image bs=4096 status=progress
```

The command used `dd` to copy the system disk `/dev/sda` into an image file named `Ubuntu20.04.image` on storage mounted under `/media/dominic/data`.

This finding supports the possibility that a full disk image was created and written to the connected USB storage device for exfiltration.

<p align="center">
  <img width="900"
       alt="Record of the Linux dd disk imaging command displayed in Paraben E3 Text View"
       src="PASTE-DD-COMMAND-IMAGE-URL-HERE" />
</p>

<p align="center">
  <em>Figure 13: Log record showing `dd` used to create a forensic-style image of `/dev/sda` on externally mounted storage.</em>
</p>

---

# Key Findings

- The Ext4 journal on `/dev/sdb5` was damaged but was successfully regenerated with `fsck`.
- `MyScheduler.txt` was not a text file; it was JPEG image data with a misleading extension.
- Linux command history and process listings provided valuable live-system evidence.
- `kern.log` and `auth.log` contained hardware, login, session, and privileged-command records.
- The accounts `noel` and `dominic` were targeted in repeated failed login attempts from `192.168.78.1`.
- `dominic` successfully logged in on Jun 11 at 04:48:18.
- `logkeys`, a Linux keylogger, was installed through `apt-get`.
- `openssh-server` was present and provided remote-access capability.
- A USB storage device with serial number `FBI1405291710` was connected on Jun 10 at 10:24:12.
- CUPS logs showed printing activity by `dominic` shortly after the USB connection.
- A `dd` command copied `/dev/sda` to `Ubuntu20.04.image` on storage mounted under `/media/dominic/data`.
- The combined evidence supports a possible insider-assisted intrusion and disk-image exfiltration.

# Evidence Timeline

| Date and Time | Event |
|---|---|
| Jun 10, 10:24:12 | USB storage device `FBI1405291710` connected |
| Jun 10, approximately 10:30:36 | CUPS print activity recorded for `dominic` |
| Jun 11, 00:57:08-00:57:34 | Failed login attempts against `noel` |
| Jun 11, 04:48:18 | Successful login session for `dominic` |
| Jun 11, 05:06:33-05:06:50 | Additional failed login attempts against `noel` |
| Jun 11, 05:07:29-05:07:57 | Failed login attempts against `dominic` |
| Jun 11, 05:36:44-05:38:32 | Additional failed login attempts against `dominic` |
| Evidence log record | `dd` used to copy `/dev/sda` to externally mounted storage |

# Conclusion

This lab demonstrated how live Linux analysis and forensic drive-image examination can be combined to reconstruct suspicious activity.

The live-system portion provided experience with Linux directories, processes, file identification, file-system repair, and log examination. The E3 investigation then revealed repeated authentication failures, a successful login, installation of a keylogger and remote-access software, USB storage activity, printing, and the creation of a full disk image on externally mounted storage.

The evidence did not rely on a single artifact. Together, authentication logs, package-installation records, kernel logs, CUPS records, and the `dd` command created a stronger timeline of a possible insider-assisted compromise and data-exfiltration event.

---

> **Note:** This repository documents work completed in an authorized educational lab environment. The evidence, users, devices, and investigative scenario were provided for cybersecurity and digital forensics training.
