# RD Gateway BlueGate (CVE-2020-0610) – Reproducible Lab

This lab reproduces the RD Gateway BlueGate (CVE-2020-0610) pre-auth UDP/DTLS issue using a minimal, non-destructive check.

## Scope & Safety
- Pre-auth DTLS handshake on UDP 3391 + a single tiny fragment (BlueGate “check”).
- Non-destructive; no DoS flood.
- Use in an isolated lab only.

## Requirements
- Hypervisor: Hyper-V / VMware / VirtualBox
- Windows Server: 2012 / 2012 R2 / 2016 / 2019 (unpatched for Jan 2020 RDG fixes)
- Admin rights on the VM
- Internet access for tooling (optional)

## Steps
1) Install RD Gateway role
- Server Manager → Add Roles and Features → Remote Desktop Services → RD Gateway

2) Enable UDP Transport
- RD Gateway Manager → <ServerName> → Properties → Transport Settings → Check “Allow users to connect by using UDP” → OK

3) Open UDP/3391
Run PowerShell as Administrator:
```powershell
powershell -ExecutionPolicy Bypass -File .\scripts\add-udp-3391-firewall.ps1
```

4) (Optional) Disable updates during validation window
- Keep the server on a snapshot in unpatched state for the test period.

5) Sanity-check
```powershell
powershell -ExecutionPolicy Bypass -File .\scripts\sanity-check.ps1
```
Expected: RDG role present; UDP/3391 firewall rule enabled; port reachable from a peer.

6) Validate with nuclei
- On your scan box:
```bash
nuclei -t network/cves/2020/CVE-2020-0610.yaml -u <rdg_host> -var rdg_port=3391 -var dtls_timeout=6 -debug
```
- Expected:
  - Patched: DEBUG_HEX ends with ffff0080 (0x8000ffff), NOT_VULNERABLE
  - Vulnerable: no such trailer, VULNERABLE

## Nice-to-have Artifacts
- `samples/nuclei-debug-vulnerable.txt` – sanitized nuclei -debug from vulnerable
- `samples/nuclei-debug-patched.txt` – sanitized nuclei -debug from patched
- Optional pcap: DTLS handshake + single fragment

## Cleanup / Revert
- Revert VM snapshot to pre-validation state.

## References
- Kryptos Logic: https://www.kryptoslogic.com/blog/2020/01/rdp-to-rce-when-fragmentation-goes-wrong/
- BlueGate PoC: https://gitlab.com/ind3p3nd3nt/BlueGate
- NVD: https://nvd.nist.gov/vuln/detail/CVE-2020-0610
