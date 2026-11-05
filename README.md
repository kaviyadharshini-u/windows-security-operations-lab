# Windows Security Operations Lab

A controlled Windows lab combining Microsoft Defender alert investigation, security configuration changes, Nessus vulnerability assessment, and post-change verification.

**Analyst:** Kaviyadharshini  
**Dates:** 4–5 November 2026  
**Environment:** Windows 11 Enterprise Evaluation in VirtualBox

## Scope and safety

Testing was restricted to my own Windows VM, SOC-Windows-Lab
(hostname DESKTOP-JRSGF7A, target IP 10.0.2.15).

The physical laptop, NAT gateway, accommodation network, other
devices and public targets were excluded. Defender and Windows
Firewall remained enabled. No real malware or denial-of-service
testing was used.

## 1. Defender detection and response

I used the EICAR antivirus test file to test detection and
remediation. EICAR is a controlled test file, not real malware.
The file was not executed.

Defender reported Virus:DOS/EICAR_Test_File in the VM's Downloads
folder.

| Evidence | Result |
|---|---|
| Event 1116, record 451 | Detection recorded at 17:30:48 UTC on 4 November |
| Event 1117, record 452 | Successful quarantine recorded at 17:30:53 UTC |
| Remediation error code | 0x00000000 |
| Correlation | Both events shared the same Detection ID |

I exported the selected events and calculated a SHA256 hash.
After copying the EVTX to my physical laptop, I recalculated
the hash and confirmed it matched.

**Evidence file:** `EICAR-Detection-and-Action.evtx`

**SHA256:**
`A4C40C3BF70C8739C6944FE267B777BFC867ECC0411A1BD0BB4AFC524FF795E4`

The matching hash supports transfer integrity. It does not
independently prove the authenticity or completeness of the logs.

## 2. Security baseline changes

| Change | Verification |
|---|---|
| Installed Windows Security platform update KB5007651 | Update history reported successful installation |
| Enabled potentially unwanted app blocking | Both Block apps and Block downloads were enabled |
| Updated Microsoft Edge from 156.0.4314.50 to 156.0.4314.58 | After restart, About reported up to date |

Defender real-time protection, cloud-delivered protection,
automatic sample submission and tamper protection were observed
enabled.

These were configuration and update observations—not findings
from Nessus. No specific CVE remediation was established.
PUA blocking behaviour was not separately tested.

## 3. Vulnerability assessment

- Scanner: Nessus Essentials 10.12.6, build R20073
- Recorded baseline plugin feed: 202610061918
- Scan: Windows-Lab-Baseline
- Policy: Basic Network Scan
- Target: 10.0.2.15 only
- Coverage: Common ports; no credentials supplied
- Baseline: 5 November 2026, 21:48–21:55

### V01 — SSL Certificate Cannot Be Trusted

| Field | Baseline result |
|---|---|
| Plugin ID | 51192 |
| Endpoint | 10.0.2.15:8834/tcp |
| Service | Nessus HTTPS management interface |
| Scanner severity | Medium |
| CVSS v3.0 | 6.5 |

The certificate chain terminated at an unrecognised Nessus
Certification Authority. This concerned the scanner's own
management interface, not an identified missing Windows patch.

### Treatment decision

I only needed to access Nessus from inside the VM. I therefore
restricted the management interface to localhost instead of
leaving it bound to all interfaces.

Before the change, port 8834 listened on `0.0.0.0` and `::`.
The owning process was confirmed as `nessusd`.

I changed `listen_address` from `0.0.0.0` to `127.0.0.1`
and restarted the Nessus service.

### Verification

- Port 8834 subsequently listened only on `127.0.0.1`.
- The Nessus interface remained accessible locally.
- A repeat scan against 10.0.2.15 completed on
  5 November 2026, 22:36–22:40.
- The original certificate finding was not reported in that run.

**Outcome: Network exposure mitigated; certificate trust not repaired.**

The certificate remains untrusted. Its absence from the
network-address rescan does not demonstrate a repaired
certificate chain. Remote management is intentionally unavailable.

The original wildcard bindings alone did not establish that the
interface was reachable from outside the VM's NAT environment.

## Limitations

- This was a single-host personal lab, not a production assessment.
- The scans were unauthenticated and limited to common ports.
- The recorded plugin feed predated the assessment.
- Scan results do not establish Windows patch status or prove
  the absence of other vulnerabilities.
- A change in displayed result counts is not a count of
  vulnerabilities fixed.
- EICAR detection does not demonstrate protection against
  every malware family or attack technique.
- Screenshots and notes were retained because this Essentials
  installation did not permit report exports.

## Supporting records

The following investigation records and screenshots are included:

- [EICAR investigation](EICAR-Investigation.txt)
- [Security baseline and changes](Security-Baseline-and-Changes.txt)
- [Assessment scope](Vulnerability-Assessment-Scope.txt)
- [Findings register](Vulnerability-Findings-Register.txt)
- [Vulnerability management procedure](Vulnerability-Management-Procedure.txt)

### Scan evidence

- [Baseline certificate finding](12-Nessus-Certificate-Finding.png)
- [Certificate output and affected endpoint](13-Nessus-Certificate-Output.png)
- [Plugin details and severity](14-Nessus-Certificate-Score.png)
- [Baseline scan configuration](15-Nessus-Scan-Information.png)
- [Completed post-change scan](16-Nessus-Rescan-Completed.png)
- [Original finding not reported in rescan](17-Nessus-Rescan-Certificate-Not-Reported.png)
- [Event-log transfer hash verification](18-EICAR-Transfer-Hash-Verified.png)

The original EVTX and uncropped evidence are retained privately.
The hash screenshot is cropped to omit the personal file path.

## Skills demonstrated

- Windows Defender event investigation and correlation
- Evidence export and file-integrity verification
- Security configuration and update verification
- Scoped vulnerability scanning
- Scanner evidence review and contextual treatment decisions
- Management-interface hardening
- Post-change verification and residual-risk documentation
