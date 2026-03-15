# SOC-Home-Lab

🛡️ SOC Home Lab
A hands-on Security Operations Center (SOC) home lab built to simulate real-world attack and detection scenarios. This project demonstrates practical skills in threat detection, log analysis, SIEM configuration, and incident response documentation.

🧰 Lab Environment
Component	Tool / Platform
Host Machine	MacBook Pro (Apple M3 Pro)
Virtualization	UTM
Attacker VM	Kali Linux (aarch64)
Vulnerable Target VM	Basic Pentesting 1 (VulnHub)
SIEM	Splunk Enterprise (host)
Log Forwarder	Splunk Universal Forwarder 10.2.1 (Kali) / rsyslog UDP 514 (Target)
Network Mode	Host-only / Internal (isolated)

🎯 Objectives
	• Simulate common attacker techniques (reconnaissance, exploitation, privilege escalation)
	• Ingest and analyze logs in Splunk to identify indicators of compromise (IOCs)
	• Build and tune detection rules and alerts
	• Practice the full SOC analyst workflow: detect → investigate → document → respond

🏗️ Lab Setup Progress
Step	Task	Status
1	Splunk Enterprise installed on host Mac	✅ Complete
2	Splunk receiving port 9997 configured	✅ Complete
3	Splunk Universal Forwarder installed on Kali Linux	✅ Complete
4	Kali Linux logs forwarding to Splunk	✅ Complete
5	Basic Pentesting VM logs forwarding to Splunk via rsyslog	✅ Complete
6	Both hosts verified visible in Splunk	✅ Complete
7	Attack simulations & detection	✅ Complete

🔌 Log Sources Configured
Kali Linux (Attacker)
Log Path	Method	Purpose
/var/log/auth.log	Splunk Universal Forwarder	Login attempts, SSH activity, sudo commands
/var/log/dpkg.log	Splunk Universal Forwarder	Software installs, tool deployment tracking
/var/log/apache2	Splunk Universal Forwarder	Web traffic and web-based attack activity
Basic Pentesting 1 (Target)
Method	Details	Purpose
rsyslog UDP 514	*.* @SIEM_IP:514	All system logs forwarded to Splunk
	💡 The target VM runs Ubuntu 16.04 (GLIBC 2.23) which is incompatible with modern Splunk Universal Forwarder versions. rsyslog forwarding was implemented instead — a common real-world approach for legacy systems in enterprise SOC environments.

🔬 Exercises Completed
#	Exercise	Techniques	Detection Method
01	Network Reconnaissance	Nmap -A -T4 scan, service enumeration	Splunk alert on Nmap signatures in syslog — count > 3 triggers High severity alert
02	Exploitation	ProFTPD 1.3.3c backdoor (CVE-2010-4221), Metasploit proftpd_133c_backdoor, reverse shell	Splunk alert on FTP connection spike > 5 per minute — Critical severity; attack timeline reconstructed across 50 min window
03	Post-Exploitation & Privilege Escalation	Reverse shell persistence, sudo enumeration, passwd/shadow file access	Splunk alert on TTY=unknown sudo activity — Critical severity; single occurrence sufficient to trigger

📊 Splunk Detection Highlights
Exercise 01 — Nmap Scan Detection
index=main host="192.168.64.7"
| search "Nmap" OR "no such user found" OR "Unable to negotiate" OR "Protocol major versions differ"
| stats count by host
| where count > 3
	Detects Nmap reconnaissance by identifying known scan signatures in syslog. Triggers a High severity alert when more than 3 matching events are found within the hour.
Exercise 02 — ProFTPD Exploitation Detection
index=main host="192.168.64.7"
| search "proftpd" "192.168.64.3"
| bucket _time span=1m
| stats count as FTP_Connections by _time, host
| where FTP_Connections > 5
	Detects ProFTPD backdoor exploitation by monitoring for abnormal FTP connection volume. Triggers a Critical severity alert when more than 5 FTP connections occur within a single minute.
Exercise 02 — Attack Timeline Reconstruction
index=main host="192.168.64.7"
| search "proftpd" "192.168.64.3"
| stats count as connections, earliest(_time) as first_seen, latest(_time) as last_seen by host
| eval duration_minutes=round((last_seen-first_seen)/60,2)
| eval first_seen=strftime(first_seen,"%Y-%m-%d %H:%M:%S")
| eval last_seen=strftime(last_seen,"%Y-%m-%d %H:%M:%S")
| table host, connections, first_seen, last_seen, duration_minutes
	Reconstructs full attack timeline showing attacker was active for 50+ minutes across reconnaissance and exploitation phases.
Exercise 03 — Reverse Shell / Privilege Escalation Detection
index=main host="192.168.64.7"
| search "TTY=unknown"
| stats count as suspicious_commands by host
| where suspicious_commands > 0
	Detects reverse shell activity by identifying sudo commands executed from non-interactive shells (TTY=unknown). Any occurrence triggers a Critical severity alert — no threshold needed as legitimate admin activity always has a TTY assigned.

🧠 Key Skills Demonstrated
	• SIEM Operations — Splunk log ingestion, indexing, search & reporting
	• Threat Detection — Writing SPL queries to identify malicious behavior
	• Offensive Awareness — Understanding attacker tools (Nmap, Metasploit) to build better defenses
	• Incident Documentation — Structured write-ups following SOC analyst workflows
	• Virtualization & Lab Design — Isolated network environment using UTM on Apple Silicon (aarch64)
	• Log Forwarding Architecture — Configured Splunk Universal Forwarder and rsyslog across multiple Linux hosts
	• Adaptability — Diagnosed GLIBC incompatibility on legacy system and implemented rsyslog as enterprise-grade alternative
	• Alert Engineering — Built and tuned automated Splunk alerts with appropriate severity classification
	• Attack Timeline Reconstruction — Used SPL to reconstruct attacker activity across a 50+ minute window
	• Noise Reduction — Distinguished malicious activity from legitimate background traffic in every exercise

📝 Methodology
Each exercise follows this structure:
	1. Objective — What attack technique is being simulated
	2. Attack Execution — Commands run on Kali Linux against the target VM
	3. Log Evidence — Raw logs captured in Splunk
	4. Detection — SPL query that identifies the activity
	5. Response Notes — What a SOC analyst would do next

⚠️ Disclaimer
This lab is conducted entirely within an isolated virtual environment on a personal machine. No external systems, networks, or third-party infrastructure are targeted. All activity is for educational purposes only.
