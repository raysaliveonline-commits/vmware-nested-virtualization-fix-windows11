## ⚠️ Important Notice

This guide disables certain Windows Virtualization-Based Security (VBS) components in order to restore full AMD-V/RVI nested virtualization support for VMware Workstation.

Before making any changes:

- Export the affected registry keys.
- Create a Windows Restore Point.
- Understand that disabling security features may reduce certain Windows protections.
- This guide was validated on Windows 11 Build 26200 and AMD Ryzen AI hardware.
- Test changes in a lab environment whenever possible.

If this is a production, corporate, or security-managed device, consult your organization's security policy before applying these changes.

---

# VMware Nested Virtualization Fix (Windows 11 + AMD Ryzen)

## Solves These Errors

### VMware Workstation

```text
Virtualized AMD-V/RVI is not supported on this platform.
```

```text
VMware Workstation does not support nested virtualization on this host.
```

```text
This host supports AMD-V, but AMD-V is disabled.
```

### Windows

```text
A hypervisor has been detected. Features required for Hyper-V will not be displayed.
```

---

# Who Is This For?

This guide is intended for users running:

- VMware Workstation Pro
- EVE-NG
- GNS3 VM
- ESXi Nested Labs
- Cisco Modeling Labs (CML)
- KVM on Linux Guests
- CCNP / CCIE Lab Environments

especially on:

- Windows 11
- AMD Ryzen Systems
- AMD Ryzen AI Systems
- ASUS ROG Laptops
- VMware Workstation 17.x

---

# Symptoms

You have already:

- Disabled Hyper-V
- Disabled Virtual Machine Platform
- Disabled Windows Hypervisor Platform
- Disabled Memory Integrity
- Rebooted

But VMware still reports:

```text
Virtualized AMD-V/RVI is not supported on this platform.
```

or

```text
A hypervisor has been detected.
```

---

# Environment Used During Troubleshooting

Hardware:

```text
ASUS ROG Zephyrus G16
AMD Ryzen AI 9 HX 370
32 GB RAM
```

Software:

```text
Windows 11 Pro Build 26200
VMware Workstation Pro 17.6.4
```

---

# Root Cause

Although Hyper-V was disabled, Windows was still loading a Secure Kernel through:

```text
System Guard
```

and

```text
Secure Biometrics
```

This caused Windows to continue reporting:

```text
HypervisorPresent = True
```

which prevented VMware from exposing AMD-V/RVI to nested guests.

---

# Step 1 - Verify Hypervisor Is Still Active

Open PowerShell as Administrator:

```powershell
Get-CimInstance -ClassName Win32_ComputerSystem | Select-Object HypervisorPresent
```

If you see:

```text
True
```

continue with this guide.

---

# Step 2 - Verify Hyper-V Components Are Disabled

Run:

```powershell
Get-WindowsOptionalFeature -Online -FeatureName Microsoft-Hyper-V-All
```

Expected:

```text
State : Disabled
```

Run:

```powershell
Get-WindowsOptionalFeature -Online -FeatureName VirtualMachinePlatform
```

Expected:

```text
State : Disabled
```

Run:

```powershell
Get-WindowsOptionalFeature -Online -FeatureName HypervisorPlatform
```

Expected:

```text
State : Disabled
```

---

# Step 3 - Disable Hypervisor Launch

Open CMD as Administrator:

```cmd
bcdedit /set hypervisorlaunchtype off
```

Verify:

```cmd
bcdedit /enum | findstr hypervisor
```

Expected:

```text
hypervisorlaunchtype Off
```

Reboot.

---

# Step 4 - Check Device Guard Scenarios

Open PowerShell:

```cmd
reg query "HKLM\SYSTEM\CurrentControlSet\Control\DeviceGuard\Scenarios" /s
```

If you discover:

```text
SystemGuard Enabled = 1
```

or

```text
SecureBiometrics Enabled = 1
```

continue.

---

# Step 5 - Disable Remaining Secure Kernel Components

Open CMD or PowerShell as Administrator.

Disable System Guard:

```cmd
reg add "HKLM\SYSTEM\CurrentControlSet\Control\DeviceGuard\Scenarios\SystemGuard" /v Enabled /t REG_DWORD /d 0 /f
```

Disable Secure Biometrics:

```cmd
reg add "HKLM\SYSTEM\CurrentControlSet\Control\DeviceGuard\Scenarios\SecureBiometrics" /v Enabled /t REG_DWORD /d 0 /f
```

---

# Step 6 - Reboot Immediately

```cmd
shutdown /r /t 0
```

---

# Step 7 - Validate Fix

Check:

```powershell
Get-CimInstance -ClassName Win32_ComputerSystem | Select-Object HypervisorPresent
```

Expected:

```text
False
```

Check:

```cmd
systeminfo | findstr /i hypervisor
```

Expected:

```text
No hypervisor detected
```

---

# Step 8 - Launch VMware

Open VMware Workstation.

For nested virtualization workloads:

- EVE-NG
- ESXi
- GNS3 VM
- Cisco Modeling Labs
- KVM Guests

verify that:

```text
Virtualize AMD-V/RVI
```

is enabled under:

```text
VM Settings
→ Processors
```

Power on the VM.

Expected result:

```text
VM boots successfully.
Nested virtualization operational.
```

---

# Troubleshooting Notes

If the issue persists:

1. Upgrade VMware Workstation to the latest 17.6.x release.
2. Verify SVM Mode is enabled in BIOS.
3. Verify HypervisorPresent returns False.
4. Verify Hyper-V, Virtual Machine Platform, and Hypervisor Platform remain disabled.
5. Check BIOS for security features that may re-enable virtualization security.

---

# TAC Summary

Observation:

```text
Hyper-V disabled
Hypervisor still detected
VMware nested virtualization failing
```

Technical Interpretation:

```text
Secure Kernel still running
System Guard still active
Hypervisor remains loaded
```

Root Cause:

```text
Windows Secure Kernel launched through System Guard
```

Resolution:

```text
Disable SystemGuard
Disable SecureBiometrics
Reboot
Verify HypervisorPresent = False
```

---

# Keywords

VMware Workstation, AMD-V, RVI, Hyper-V, Nested Virtualization, Windows 11, Ryzen AI, ASUS ROG, EVE-NG, ESXi, GNS3, Cisco Modeling Labs, Device Guard, System Guard, Secure Kernel, Hypervisor Detected, VMware AMD-V Fix
