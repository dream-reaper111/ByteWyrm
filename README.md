# ByteWyrm
https://giphy.com/gifs/Rageguy-hacker-rageguy-rage-guy-onqm2IiY4fX7Zsghp9
https://giphy.com/gifs/14kdiJUblbWBXy
https://giphy.com/gifs/loop-glitch-matrix-3og0IAIsfBiw1eQ2R2
Legacy Python Malware Analysis Sample
Static analysis / archival research. This repository contains destructive file-handling code and a remote-control server. Do not execute these samples on a personal, production, or shared system.

Overview
This repository contains three legacy Python scripts: a TCP controller (server.py), a remote shell client (shell.py), and a file-encryption/payload-download sample (worm.py). The code illustrates older remote-control and ransomware-like patterns, including embedded secrets, custom encrypted messaging, local copying, and destructive file processing.
This README describes the supplied source. It does not claim that either script works successfully in its current state or that the three scripts form a complete working system.
Repository Contents
File	Role	Observed behavior
server.py	Interactive TCP controller	Accepts connections, tracks clients, sends command strings, and implements upload/download requests using AES-CFB and Base64.
shell.py	Remote shell client	Connects to a controller, executes shell commands, transfers files, and exposes an optional Windows persistence routine.
worm.py	Destructive file-processing sample	Attempts local copying, recursively encrypts files, removes originals after encryption, and attempts to download and execute an external executable.


The executable referenced by worm.py is not included. The client source was supplied separately as pasted shell.py text; no evidence establishes that the referenced executable was built from it.
Source-Level Behavior
server.py
- Binds an IPv4 TCP listener to all local interfaces on port 6000.
- Stores accepted sockets and client addresses for interactive selection.
- Sends an activation message when a client session is selected.
- Provides controller-side command dispatch and file-transfer handling.
- Uses AES-CFB with a hardcoded shared secret and fixed initialization vector.
- Base64-encodes encrypted messages and uses a custom end-of-message marker.
The supplied shell.py uses matching protocol constants and appears intended as the corresponding client; interoperability has not been tested. A listener backlog of 128 is not evidence of 128 concurrently managed interactive sessions.
shell.py
- Attempts an outbound TCP connection to a placeholder controller address on port 6000.
- Waits for an activation message and then accepts command strings.
- Executes received commands through a shell and returns captured standard output and standard error.
- Supports changing its working directory, bidirectional file transfer, and downloading a file from a URL. The URL-download handler writes a file; it does not itself launch that file.
- Implements a Windows persistence routine that writes vbscript.vbs and attempts to register it in a machine-wide Run key. The generated script monitors the executable, attempts hidden relaunch, and optionally re-downloads it if absent. This routine is invoked only when requested through command dispatch, not automatically at startup.
- Retries connections after socket errors.
The persistence routine reports completion without establishing that the registry operation succeeded; permissions and platform dependencies can prevent it from working. No keylogging, credential dumping, or privilege-escalation implementation is shown in this client.
worm.py
The main() function calls these routines in order:
1. copy() attempts to create four local directories and copy the script into them.
2. action() walks the current working directory recursively, encrypts eligible files, and deletes each original after its encryption routine returns.
3. downloadBackdoor() attempts to fetch an executable over HTTP and pass its local path to a shell-based process launcher.
This is the intended call sequence, not a verified execution trace. Unhandled errors in earlier operations can prevent later steps from running.
File encryption uses AES-CBC with a key derived from an embedded password through SHA-256. Each output contains a 16-byte textual original-size field, a 16-byte IV, and encrypted content. Output filenames receive the (encrypted) prefix. A decryption function exists, but main() does not call it; reliable recovery has not been established.
Defined but disabled behavior
Calls to hide() and propagate() are commented out in main().
- hide() would mark Python files hidden, mark text files read-only, and attempt to delete other entries in the current directory. It is destructive, not merely cosmetic.
- propagate() selects a local drive or user-directory destination, copies a file named worm.py, and attempts to launch that copy.
Despite the filename, the supplied source does not demonstrate network scanning or autonomous network propagation. Local copies alone do not establish a working network worm.
Indicators for Defensive Review
These values are extracted from source and are not proof of execution or attribution.
Indicator	Source context
TCP port 6000	Controller listener
EOFEOFEOFEOFEOFX	Custom message terminator
(encrypted) filename prefix	Encrypted output naming
copy0 through copy3	Local directories created by copy()
hxxp://172[.]16[.]190[.]175/security/dist/shell.exe	Embedded executable-download destination, defanged
shell.exe	Downloaded payload filename
vbscript.vbs	Script written by the client persistence routine
Machine-wide Windows CurrentVersion\Run key, value blah	Attempted client startup registration


The embedded destination uses a private IPv4 address. Its presence does not establish that the endpoint exists, is reachable, or was used in an incident. Embedded cryptographic secrets are intentionally not reproduced here.
Technical Limitations
- Legacy runtime: Python 2 print statements, raw_input, and urllib2 prevent direct execution under Python 3.
- Platform mismatch: Unconditional Windows-specific imports can prevent worm.py from loading on Linux despite its Linux path branches.
- Missing components: The downloaded executable is absent; the relationship between it and the separately supplied client source is unverified.
- Unsafe cryptography: Embedded secrets, a fixed controller IV, unauthenticated ciphertext, and a non-cryptographic IV generator in the file-encryption routine undermine security.
- Fragile transport: TCP receive boundaries are treated as independently decodable encrypted messages, although TCP provides a byte stream. Custom delimiter handling does not establish reliable framing.
- Unsafe input: Python 2 input() evaluates expressions in the controller's client-selection prompt.
- Fragile file handling: Relative source paths, existing directories, permission failures, path parsing, and incomplete error handling can interrupt processing.
- Recovery uncertainty: Self-exclusion and decryption path construction are brittle; the presence of a decrypt function does not guarantee safe restoration.
These limitations are analysis findings, not a repair or modernization guide.
Defensive Analysis Workflow
1. Preserve original files and record cryptographic hashes before analysis.
2. Review source as text without importing or executing it.
3. Record enabled entry points separately from unused or disabled functions.
4. Compare source indicators with available process, filesystem, and network evidence.
5. If execution is necessary for an authorized investigation, use a disposable, isolated malware-analysis environment without shared folders, mounted personal storage, credentials, or external connectivity.
Useful defensive evidence includes rapid file creation and deletion, (encrypted) filenames, Python-launched shell processes, unexpected executable downloads, file-attribute changes, and unauthorized listeners on TCP port 6000. Correlate multiple signals; individual filenames or ports are not conclusive.
Research Roadmap
- Add a static-analysis report with file hashes and annotated source references.
- Document dependencies and missing components without restoring operational capability.
- Build a harmless simulation using synthetic files and recorded events, with encryption, deletion, replication, and payload execution removed.
- Develop defensive detection examples and evaluate false positives against benign activity.
- Add an evidence-based behavior timeline if sandbox observations become available.
Status and Attribution
Status: Legacy source sample; reviewed statically only. None of the scripts were executed to prepare this README.
Author Credit
Originally written by Tristan Swickard (deadbyte.dll) at age 15.
Preserved as an early programming and cybersecurity project, documenting the author’s learning at that age. Authorship and age are credited according to the author’s account; an exact creation date has not been established.
A license was not supplied. Preserve any third-party notices and verify redistribution rights before publishing.
