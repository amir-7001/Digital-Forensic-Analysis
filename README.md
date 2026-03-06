Executive Summary Draft
Project: Digital Forensic Investigation - OtterCTF
Case File: 2026-MAL-3820

Investigator: Amir Bello

Tools Used: Volatility Framework 2.6, VirusTotal, Terminal Strings, Markdown

🔍 Overview
This repository documents a comprehensive memory forensic investigation of a compromised Windows 7 system. The objective was to identify the entry point of a malware infection, track its persistence mechanism, identify the attacker’s Command & Control (C2) infrastructure, and recover exfiltrated user credentials.

🎯 Key Findings
Infection Vector: The user downloaded a malicious RAR archive masquerading as "Rick and Morty" media via a web browser.

Malware Profile: The "dropper" executed a process-hollowing technique, hiding its activity inside a fake vmware-tray.exe process.

C2 Infrastructure: Active communication was established with a remote server at 77.102.199.102 over port 7575.

Evidence Recovered: Successfully extracted the plaintext password/flag CTF{...} from the lsass.exe memory space.

🛠 Step-by-Step Forensic Analysis
1. Image Identification & Environment Setup
Before beginning the analysis, I needed to identify the operating system profile of the memory dump to ensure the Volatility plugins would interpret the data structures correctly.
![](<Images/Screenshot 2026-03-05 091102.png>)

Forensic Note: The suggested profile Win7SP1x64 was selected. This step is critical to prevent data misalignment during process and network carving.

2. Identifying the Malicious Process Tree
I analyzed the process hierarchy to look for anomalies. A suspicious process named Rick And Morty season 1 download.exe was found spawning vmware-tray.exe.

![](<Images/Screenshot 2026-03-06 014035.png>)
Forensic Note: Legitimate vmware-tray.exe should not be a child of a browser download. This is a clear indicator of Process Masquerading.

3. Extracting the Malicious Payload
To perform static analysis, I extracted the suspicious executable from the memory dump back into a file on my analysis machine.
SHA-1 Hash: 314605d57b35de5ed45cc9461658ffb174d04985
![](<Images/Screenshot 2026-03-05 223907.png>)
Forensic Note: The extracted file was identified as a RAR self-extracting archive, a common wrapper for malware droppers.

5. Digital Fingerprinting (Hash Analysis)
I generated a SHA1 hash of the extracted malware to check against global threat intelligence databases like VirusTotal.
![](<Images/Screenshot 2026-03-05 230249.png>)
Forensic Note: The hash was flagged as malicious by multiple security vendors, confirming the file as a known Trojan/Dropper.

6. Network C2 Infrastructure Discovery
I conducted a network scan to find active or recently closed connections associated with the malware's PID.
![](<Images/Screenshot 2026-03-06 022742.png>)
Forensic Note: The malware established a connection to 77.102.199.102 on Port 7575. This IP acts as the Command & Control (C2) server for the attacker.

7. Credential Recovery & Flag Decryption
I targeted the lsass.exe memory space and LSA secrets to recover the user's plaintext password, which served as the primary objective (Flag) for this phase.
![](<Images/Screenshot 2026-03-06 023708.png>)
![](<Images/Screenshot 2026-03-06 025105.png>)
Forensic Note: The password CTF{...} was successfully recovered, indicating that the attacker had already achieved data exfiltration capabilities.

. Infection Vector Analysis 
I reconstructed the user's browser history to determine how the malware first entered the system.
![](<Images/Screenshot 2026-03-06 025533.png>)
Forensic Note: The user navigated to a specific download URL, confirming the infection was a result of a Drive-by Download or social engineering.

. Master File Table (MFT) Timeline 
I parsed the MFT to establish a definitive timeline of when the malicious file was written to the disk.
![](<Images/Screenshot 2026-03-06 025938.png>)
Forensic Note: The timestamp matches the network connection timing, providing a cohesive timeline of execution and compromise.
