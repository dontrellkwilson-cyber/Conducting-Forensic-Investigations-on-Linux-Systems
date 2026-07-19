# Conducting Forensic Investigations on Linux Systems

<p align="center">
  <strong>Live Linux Analysis, Log Examination, File-System Repair, and E3 Drive-Image Investigation</strong>
</p>

---

## Objective

The objective of this lab was to investigate a live Linux system and a forensic image of a Linux server for evidence related to unauthorized access attempts, software installation, USB storage activity, printing, and disk imaging.

The lab combined Linux command-line analysis with Paraben E3 examination of an ext4 drive image. The investigation included reviewing key Linux directories, examining system and authentication logs, repairing a damaged test file system, identifying a disguised file, analyzing successful and failed SSH authentication events, and locating evidence consistent with possible data exfiltration.

## Skills Demonstrated

- Navigating the Linux file system through the command line
- Reviewing `/bin`, `/etc`, `/var`, and `/proc`
- Examining kernel and boot-related messages with `dmesg`
- Checking and repairing an ext4 test file system with `fsck`
- Reviewing Linux shell history and its limitations
- Enumerating running processes with `ps aux`
- Detecting a misleading file extension with the `file` command
- Examining `kern.log` and `auth.log`
- Importing and examining a Linux drive image with Paraben E3
- Searching authentication logs for failed and successful SSH activity
- Identifying evidence of package-installation commands
- Identifying commands associated with a Linux keylogger and remote-access service
- Identifying USB storage activity and device serial information
- Reviewing CUPS printer logs
- Finding evidence of raw disk imaging with the `dd` command
- Correlating multiple artifacts without overstating their meaning

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
| File System | ext4 |
| Investigation Type | Linux intrusion, insider-threat, and possible data-exfiltration investigation |

## Tools and Commands Used

| Tool or Command | Purpose |
|---|---|
| **Linux Terminal** | Performed live system and file-system analysis |
| **Paraben E3** | Imported and examined the Linux forensic image |
| `ls -l` | Displayed directory contents, permissions, owners, sizes, and timestamps |
| `dmesg` | Displayed messages from the kernel ring buffer |
| `fsck` | Checked and repaired the disposable ext4 test file system |
| `history` | Displayed commands retained in the current shell history |
| `ps aux` | Displayed running processes and resource information |
| `file` | Identified a file's format based on its contents rather than its extension |
| `tail` | Displayed recent kernel-log records |
| `less` | Reviewed authentication-log records |
| **E3 Advanced Search** | Located authentication events, installation commands, USB activity, and disk-imaging commands |

## Investigation Scenario

In the first section, I examined a live Ubuntu system to identify key Linux directories, active processes, system activity, file-system errors, and log records.

In the second section, I acted as a junior forensic investigator reviewing a Linux drive image from Harper and Associates. The organization suspected that someone had attempted to access a critical Linux server using legitimate usernames. The analysis identified repeated authentication failures, a successful SSH authentication event, package-installation commands, USB storage activity, printing, and the creation of a raw disk image on externally mounted storage.

> **Forensic handling note:** Live-system commands can change access times, shell history, caches, logs, and other system state. Repair tools such as `fsck` can make extensive changes to a file system. These actions were performed only in an authorized educational lab and should not be run against original evidence. In an actual investigation, the examiner should document each live-response action and perform repairs only on a verified working copy.

---

## Section 1: Hands-On Demonstration

### Part 1: Explore a Live Linux System

#### Examining the `/bin` Directory

I navigated to `/bin` and used a long listing to review executable files, symbolic links, permissions, owners, sizes, and timestamps.

```bash
cd /bin
ls -l
```

The `/bin` path provides access to essential system commands. On modern Ubuntu systems, `/bin` may be a symbolic link to `/usr/bin`. Reviewing this location can help identify unexpected executables or changes that require further validation against trusted package information and known-good hashes.

<p align="center">
  <img src="https://github.com/user-attachments/assets/9d8e0655-2a25-4445-acdf-161022b6eb7b"
       alt="Linux terminal showing a long listing of files and symbolic links under the bin directory"
       width="800">
</p>

<p align="center">
  <em>Figure 1: Contents of the `/bin` directory displayed with `ls -l`.</em>
</p>

#### Examining the `/etc` Directory

I reviewed `/etc`, which contains system-wide configuration files and subdirectories.

```bash
cd /etc
ls -l
```

Configuration files can provide evidence of service changes, user and authentication settings, network configuration, scheduled activity, persistence mechanisms, or application tampering. A directory listing alone does not establish that a file was altered, so suspicious entries should be compared with package records, timestamps, backups, and known-good configurations.

<p align="center">
  <img src="https://github.com/user-attachments/assets/30ef6aee-d0b5-4724-95b9-2a4acfc62531"
       alt="Linux terminal showing system configuration files and directories under etc"
       width="800">
</p>

<p align="center">
  <em>Figure 2: System configuration files and directories located under `/etc`.</em>
</p>

#### Examining the `/var` Directory

I reviewed `/var`, which contains changing system data such as logs, caches, spool data, temporary data, and application records.

```bash
cd /var
ls -l
```

<p align="center">
  <img src="https://github.com/user-attachments/assets/a6b82280-4897-4f84-a0ce-b0cdaef22a2c"
       alt="Linux terminal showing variable system data and log-related directories under var"
       width="800">
</p>

<p align="center">
  <em>Figure 3: Variable system data and log-related directories located under `/var`.</em>
</p>

#### Examining the `/proc` Directory

I reviewed `/proc`, which is a virtual file system generated by the kernel to expose current process, hardware, and system information.

```bash
cd /proc
ls -l
```

The numbered directories under `/proc` correspond to active process IDs, while other files expose current kernel and system information. Because `/proc` reflects the live state of the system, its contents can change while the examination is in progress.

<p align="center">
  <img src="https://github.com/user-attachments/assets/757297ee-0be9-4d32-8c27-afab090901f2"
       alt="Linux terminal showing process ID directories and system entries in the proc virtual file system"
       width="800">
</p>

<p align="center">
  <em>Figure 4: Live process and system information available through the `/proc` virtual file system.</em>
</p>

---

### Part 2: Use Linux Shell Commands for Forensic Investigations

#### Reviewing Kernel and Boot Messages

I used `dmesg` to review messages currently available in the kernel ring buffer.

```bash
dmesg
```

The output included kernel, AppArmor, service, and device-related records that could help identify failed services, startup activity, hardware events, or security-control messages. Because the ring buffer is limited and can be overwritten, `dmesg` should be correlated with persistent logs when available.

<p align="center">
  <img src="https://github.com/user-attachments/assets/245085fb-b7e4-45b3-9077-4f5d5321e5be"
       alt="Linux terminal displaying kernel, AppArmor, service, and hardware messages from dmesg"
       width="800">
</p>

<p align="center">
  <em>Figure 5: Kernel and boot-related messages displayed with the `dmesg` command.</em>
</p>

#### Checking and Repairing the ext4 File System

Before running `fsck`, I confirmed that the disposable test partition was not mounted. I then checked the ext4 file system on `/dev/sdb5`:

```bash
sudo fsck /dev/sdb5
```

The initial check reported an invalid journal superblock. I stopped the interactive check and used preen mode to automatically correct issues that `fsck` classified as safe to repair:

```bash
sudo fsck -p /dev/sdb5
```

I then ran the standard check again to verify the result:

```bash
sudo fsck /dev/sdb5
```

The final output reported the file system as clean after the journal-related repair completed.

> **Caution:** `fsck` changes file-system metadata and should not be run on a mounted partition or original forensic evidence. Repairs should be performed only on a verified working copy after documenting its hash.

<p align="center">
  <img src="https://github.com/user-attachments/assets/f6d443cf-002f-4ca7-809e-f42a2713dcfd"
       alt="Linux terminal showing fsck journal repair and final clean status for the ext4 test partition"
       width="800">
</p>

<p align="center">
  <em>Figure 6: `fsck` output showing the journal-related repair and final file-system verification.</em>
</p>

#### Reviewing Shell Command History

I reviewed commands retained in the current user's shell history:

```bash
history
```

Shell history can reveal file deletion, copying, package installation, disk manipulation, or other activity relevant to an investigation. However, it may be incomplete, disabled, cleared, edited, or limited to a particular user and shell session, so it should not be treated as a complete activity record.

<p align="center">
  <img src="https://github.com/user-attachments/assets/2e383c1f-0972-4d13-93fa-6f09590223db"
       alt="Linux terminal displaying commands retained in the current user's shell history"
       width="800">
</p>

<p align="center">
  <em>Figure 7: Shell history showing commands retained for the current user.</em>
</p>

#### Reviewing Running Processes

I displayed active processes with:

```bash
ps aux
```

The output included each process owner, PID, CPU use, memory use, start information, state, and command. This represented a point-in-time view and could change immediately as processes started or stopped.

<p align="center">
  <img src="https://github.com/user-attachments/assets/247e93b4-f045-4490-b4c3-23ed6d9ad851"
       alt="Linux terminal displaying active processes, users, process IDs, and command lines with ps aux"
       width="800">
</p>

<p align="center">
  <em>Figure 8: Active Linux processes displayed with `ps aux`.</em>
</p>

#### Identifying a Disguised File

I navigated to the user's Documents directory and examined `MyScheduler.txt`:

```bash
cd /home/user/Documents
ls
file MyScheduler.txt
```

The `file` command identified `MyScheduler.txt` as JPEG image data rather than plain text. This demonstrated that a filename extension can be misleading and that the file's contents should be examined before assigning a format or purpose.

<p align="center">
  <img src="https://github.com/user-attachments/assets/a23e57de-b799-4812-9066-db5ec5b045bb"
       alt="Linux terminal showing the file command identifying MyScheduler.txt as JPEG image data"
       width="800">
</p>

<p align="center">
  <em>Figure 9: The `file` command identifying `MyScheduler.txt` as JPEG image data.</em>
</p>

---

### Part 3: Retrieve Log Files on a Live Linux System

#### Reviewing `kern.log`

I navigated to the Linux log directory and reviewed recent kernel records:

```bash
cd /var/log
ls
sudo tail -n 50 kern.log
```

The kernel log can contain hardware, driver, AppArmor, startup, and external-device activity. Log availability depends on the Ubuntu version and logging configuration, so equivalent records may also appear in `journalctl`.

<p align="center">
  <img src="https://github.com/user-attachments/assets/ff40d78c-4d81-46af-9119-5ed0335be193"
       alt="Linux terminal displaying recent kernel log records from var log kern.log"
       width="800">
</p>

<p align="center">
  <em>Figure 10: Recent kernel records displayed from `kern.log`.</em>
</p>

#### Reviewing `auth.log`

I reviewed authentication and authorization activity with:

```bash
sudo less /var/log/auth.log
```

The authentication log can contain SSH login attempts, `sudo` activity, opened and closed sessions, and commands invoked through `sudo`. The exact fields and retained events depend on system configuration and log rotation.

<p align="center">
  <img src="https://github.com/user-attachments/assets/54ee299d-e5fe-4de1-a094-42c570f56a64"
       alt="Linux terminal displaying SSH, session, and sudo records from auth.log"
       width="800">
</p>

<p align="center">
  <em>Figure 11: Authentication, session, and elevated-command records displayed from `auth.log`.</em>
</p>

---

## Section 2: Applied Learning

### Part 1: Identify Login Attempts in a Linux Drive Image

I created an E3 case and imported:

```text
C:\Linux Forensics\Ubuntu20.04.image
```

I navigated to the Linux log directory in the ext4 partition:

```text
Ubuntu20.04
└── Partition Parser
    └── Partition1
        └── EXT4
            └── Root
                └── var
                    └── log
```

I examined `auth.log` and used E3 Advanced Search to identify failed and successful SSH authentication activity.

#### Failed Login Findings

| User | Failed Attempts | Source IP | Observed Source Ports | Time Ranges |
|---|---:|---|---|---|
| `noel` | 12 | `192.168.78.1` | `14444`, `3521` | Jun 11, 00:57:08–00:57:34 and 05:06:33–05:06:50 |
| `dominic` | 11 | `192.168.78.1` | `4663`, `3417` | Jun 11, 05:07:29–05:07:57 and 05:36:44–05:38:32 |

The attempts occurred in short groups with pauses between them. This did not resemble a continuous high-speed brute-force sequence, but timing alone was not sufficient to determine whether the activity was manual or automated.

<p align="center">
  <img src="https://github.com/user-attachments/assets/82bd1ed8-83d7-4e2d-ad11-80e93698d7df"
       alt="Paraben E3 showing failed SSH authentication attempts against noel from 192.168.78.1"
       width="800">
</p>

<p align="center">
  <em>Figure 12A: Failed SSH authentication attempts against `noel` from `192.168.78.1`, including records with source port `14444`.</em>
</p>

<p align="center">
  <img src="https://github.com/user-attachments/assets/10d01b9f-f038-40ff-ab8b-217ea65cff66"
       alt="Paraben E3 showing failed SSH authentication attempts against dominic from 192.168.78.1"
       width="800">
</p>

<p align="center">
  <em>Figure 12B: Failed SSH authentication attempts against `dominic` from `192.168.78.1`, including records with source port `4663`.</em>
</p>

#### Successful Login Finding

The most recent successful SSH authentication identified for `dominic` occurred at:

```text
Jun 11 at 04:48:18
```

No successful SSH authentication for `noel` was identified in the records reviewed. The successful event established that valid credentials were accepted, but the log entry alone did not establish who used them or whether the access was authorized.

---

### Part 2: Identify Software-Installation Commands in a Linux Drive Image

I searched `auth.log` for:

```text
apt-get install
```

The records showed `sudo` commands in which the user `dominic` invoked `apt-get install` for several packages.

| Package | Forensic Assessment |
|---|---|
| `logkeys` | Keylogging software capable of recording keyboard input; highly relevant to a credential-theft investigation |
| `openssh-server` | Provides remote SSH access; legitimate in many environments but relevant when correlated with remote authentication activity |
| `autotools-dev` | Development support package |
| `build-essential` | Compilation toolchain commonly used to build software |
| `autoconf` | Used to configure software for compilation |
| `kbd` | Keyboard and console utility package |
| `acct` | Process and login accounting package |

The `logkeys` command was the strongest concern because the software can capture keyboard input. The development packages were consistent with preparing to compile or configure software, but they were not suspicious by themselves. The `auth.log` records showed that installation commands were invoked; package-manager logs such as `/var/log/apt/history.log` or `/var/log/dpkg.log` would provide stronger confirmation that each installation completed successfully.

<p align="center">
  <img src="https://github.com/user-attachments/assets/89580e2d-a1e2-47d4-9b3a-264a23738572"
       alt="Paraben E3 showing apt-get install commands associated with logkeys and supporting packages"
       width="49%">
  <img src="https://github.com/user-attachments/assets/5c91394a-178e-4869-aa42-fe1b7d1d8802"
       alt="Paraben E3 showing additional apt-get installation commands invoked by dominic"
       width="49%">
</p>

<p align="center">
  <em>Figure 13: `apt-get install` commands invoked by `dominic`, including a command involving the `logkeys` keylogger.</em>
</p>

---

### Part 3: Identify External Drive Attachments

I searched `kern.log` for the phrase:

```text
USB mass storage device detected
```

The investigation identified the following USB storage device:

| Field | Finding |
|---|---|
| Connection Time | Jun 10 at 10:24:12 |
| Device Name | `USB DISK` |
| Serial Number | `FBI1405291710` |

The USB event occurred before the later authentication activity. It showed that the operating system detected a USB mass-storage device, but the event alone did not establish who connected it, whether it was authorized, or what data was transferred.

<p align="center">
  <img src="https://github.com/user-attachments/assets/3e82cc89-3402-47b8-83e1-6ecf46c3497a"
       alt="Paraben E3 showing kernel log records for USB DISK device serial FBI1405291710"
       width="800">
</p>

<p align="center">
  <em>Figure 14: Kernel log records identifying the connected USB storage device and its serial number.</em>
</p>

---

## Section 3: Challenge and Analysis

### Part 1: Identify Recently Printed Files

I researched Linux printing records and examined the CUPS log directory in E3:

```text
/var/log/cups
```

I opened `page_log` in Document View. The record showed a print job associated with the user `dominic` on June 10, 2021, at approximately 10:30:36.

<p align="center">
  <img src="https://github.com/user-attachments/assets/fa8c1d3f-6414-4c5e-aa76-02165cfc1fed"
       alt="Paraben E3 showing a CUPS page_log print record associated with dominic"
       width="800">
</p>

<p align="center">
  <em>Figure 15: CUPS `page_log` record showing printing activity associated with the user `dominic`.</em>
</p>

---

### Part 2: Identify Disk Imaging

I searched the `/var/log` folder for records containing:

```text
/usr/bin/dd
```

The search located evidence of the following disk-copy command:

```bash
sudo dd if=/dev/sda of=/media/dominic/data/Ubuntu20.04.image bs=4096 status=progress
```

The command instructed `dd` to read the system disk `/dev/sda` and write a raw image named `Ubuntu20.04.image` to storage mounted at `/media/dominic/data`.

This command was consistent with creating a full disk image on externally mounted storage. When correlated with the USB device record, it supported a possible data-exfiltration scenario, but the command and mount path alone did not prove that the destination was the same USB device or that the image later left the organization's control.

<p align="center">
  <img src="https://github.com/user-attachments/assets/857826b2-945d-4fa2-bb47-6f9263709314"
       alt="Paraben E3 showing a logged dd command that copied dev sda to Ubuntu20.04.image on mounted storage"
       width="800">
</p>

<p align="center">
  <em>Figure 16: Log record showing `dd` used to create a raw image of `/dev/sda` on mounted storage.</em>
</p>

---

## Key Findings

- The ext4 journal on `/dev/sdb5` required repair, and the final `fsck` check reported the disposable test file system as clean.
- `MyScheduler.txt` contained JPEG image data despite its `.txt` extension.
- Linux shell history and process listings provided useful live-system information but represented incomplete, changeable views of activity.
- `kern.log` and `auth.log` contained hardware, authentication, session, and `sudo` command records.
- The usernames `noel` and `dominic` appeared in repeated failed SSH authentication attempts from `192.168.78.1`.
- A successful SSH authentication for `dominic` was recorded on Jun 11 at 04:48:18, although the record alone did not identify the person using the credentials.
- `auth.log` contained a command invoking installation of `logkeys`, a Linux keylogger.
- An `openssh-server` installation command was also present, providing relevant context for remote-access analysis without proving malicious use.
- A USB storage device with serial number `FBI1405291710` was detected on Jun 10 at 10:24:12.
- CUPS logs showed printing activity associated with `dominic` shortly after the USB connection.
- A logged `dd` command copied `/dev/sda` to `Ubuntu20.04.image` under `/media/dominic/data`.
- Correlation of the USB, authentication, installation-command, printing, and disk-imaging artifacts supported a possible insider-assisted compromise and data-exfiltration hypothesis, but additional evidence would be required to confirm it.

## Evidence Timeline

| Date and Time | Event |
|---|---|
| Jun 10, 10:24:12 | USB storage device `FBI1405291710` detected |
| Jun 10, 2021, approximately 10:30:36 | CUPS print activity recorded for `dominic` |
| Jun 11, 00:57:08–00:57:34 | Failed SSH authentication attempts against `noel` |
| Jun 11, 04:48:18 | Successful SSH authentication recorded for `dominic` |
| Jun 11, 05:06:33–05:06:50 | Additional failed SSH authentication attempts against `noel` |
| Jun 11, 05:07:29–05:07:57 | Failed SSH authentication attempts against `dominic` |
| Jun 11, 05:36:44–05:38:32 | Additional failed SSH authentication attempts against `dominic` |
| Time not established from the cited record | `dd` command instructed the system to copy `/dev/sda` to mounted storage |

## Conclusion

This lab demonstrated how live Linux analysis and forensic drive-image examination can be combined to reconstruct potentially suspicious activity.

The live-system portion provided experience with Linux directories, processes, file identification, file-system repair, and log examination. The E3 investigation then identified repeated authentication failures, a successful SSH authentication event, commands involving a keylogger and an SSH service, USB storage activity, printing, and the creation of a raw disk image on mounted storage.

The findings did not rely on a single artifact. Authentication logs, package-installation commands, kernel logs, CUPS records, and the `dd` command created a stronger timeline when examined together. The combined evidence supported a possible insider-assisted compromise and data-exfiltration hypothesis, while still requiring additional validation before reaching a final attribution or intent determination.

---

> **Note:** This repository documents work completed in an authorized educational lab environment. The evidence, users, devices, and investigative scenario were provided for cybersecurity and digital forensics training.
