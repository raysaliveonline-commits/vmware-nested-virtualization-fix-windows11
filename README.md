# VMware Nested Virtualization Fix (Windows 11 + AMD Ryzen)

> Resolving VMware Workstation nested virtualization failures caused by Hyper-V, VBS, Device Guard, System Guard, Secure Kernel, and Windows 11 security components.

---

## ⚠️ Important Notice

This guide disables certain Windows Virtualization-Based Security (VBS) components in order to restore full AMD-V/RVI nested virtualization support for VMware Workstation.

Before making any changes:

* Export the affected registry keys.
* Create a Windows Restore Point.
* Understand that disabling security features may reduce certain Windows protections.
* Test changes in a lab environment whenever possible.
* Review your organization's security policies before applying these changes on corporate devices.

This guide was validated on:

* Windows 11 Pro Build 26200
* VMware Workstation Pro 17.6.4
* AMD Ryzen AI 9 HX 370
* ASUS ROG Zephyrus G16

---

## Common Search Terms

## Quick Fix (TL;DR)

If you have already:

* Disabled Hyper-V
* Disabled Virtual Machine Platform
* Disabled Windows Hypervisor Platform
* Disabled Memory Integrity
* Rebooted

but Windows still reports:

```text
A hypervisor has been detected.
```

and VMware still reports:

```text
Virtualized AMD-V/RVI is not supported on this platform.
```

run:

```powershell
Get-CimInstance -ClassName Win32_ComputerSystem | Select-Object HypervisorPresent
```

If the result is:

```text
True
```

check Device Guard scenarios:

```cmd
reg query "HKLM\SYSTEM\CurrentControlSet\Control\DeviceGuard\Scenarios" /s
```

If you find:

```text
SystemGuard Enabled = 1
```

or

```text
SecureBiometrics Enabled = 1
```

disable them:

```cmd
reg add "HKLM\SYSTEM\CurrentControlSet\Control\DeviceGuard\Scenarios\SystemGuard" /v Enabled /t REG_DWORD /d 0 /f

reg add "HKLM\SYSTEM\CurrentControlSet\Control\DeviceGuard\Scenarios\SecureBiometrics" /v Enabled /t REG_DWORD /d 0 /f
```

Reboot:

```cmd
shutdown /r /t 0
```

Validate:

```powershell
Get-CimInstance -ClassName Win32_ComputerSystem | Select-Object HypervisorPresent
```

Expected:

```text
False
```

If successful, VMware nested virtualization should function normally again.

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

```text
Module 'HV' power on failed.
```

### Windows

```text
A hypervisor has been detected.
Features required for Hyper-V will not be displayed.
```

---

## Who Is This Guide For?

This guide is intended for users running:

* VMware Workstation Pro
* EVE-NG
* GNS3 VM
* Nested ESXi Labs
* Cisco Modeling Labs (CML)
* KVM-based Linux Guests
* CCNA, CCNP, and CCIE Lab Environments
* Home Labs and Enterprise Test Labs

---

## Symptoms

You have already:

* Disabled Hyper-V
* Disabled Virtual Machine Platform
* Disabled Windows Hypervisor Platform
* Disabled Memory Integrity
* Rebooted multiple times

Yet VMware still reports:

```text
Virtualized AMD-V/RVI is not supported on this platform.
```

or

```text
A hypervisor has been detected.
```

and nested virtualization continues to fail.

---

## Environment Used During Troubleshooting

### Hardware

* ASUS ROG Zephyrus G16 GA605WI
* AMD Ryzen AI 9 HX 370
* 32 GB RAM

### Software

* Windows 11 Pro Build 26200
* VMware Workstation Pro 17.6.4

---

## Verified Working For

### Tested Nested Virtualization Workloads

* EVE-NG
* Nested ESXi
* GNS3 VM
* Cisco Modeling Labs (CML)
* KVM-based Linux Guests

### Successfully Resolved Errors

* Virtualized AMD-V/RVI is not supported on this platform
* VMware Workstation does not support nested virtualization on this host
* Module 'HV' power on failed
* HypervisorPresent = True after Hyper-V was disabled
* A hypervisor has been detected

---

## Root Cause

Although Hyper-V was disabled, Windows continued loading a Secure Kernel through virtualization-based security components.

Investigation revealed:

```text
HypervisorPresent = True
```

and:

```text
Virtualization-based security: Running
```

Even after:

* Hyper-V disabled
* Virtual Machine Platform disabled
* Windows Hypervisor Platform disabled
* Memory Integrity disabled
* Hypervisor launch disabled

Further investigation identified:

```text
SystemGuard Enabled = 1
```

and:

```text
SecureBiometrics Enabled = 1
```

These components were still causing Windows to launch the Secure Kernel, resulting in a hypervisor being loaded at boot.

VMware therefore could not expose AMD-V/RVI to nested guests.

---

## Troubleshooting Workflow

### Step 1 – Verify Hypervisor Is Still Active

Open PowerShell as Administrator:

```powershell
Get-CimInstance -ClassName Win32_ComputerSystem | Select-Object HypervisorPresent
```

If the result is:

```text
True
```

continue with this guide.

---

### Step 2 – Verify Hyper-V Components Are Disabled

Check Hyper-V:

```powershell
Get-WindowsOptionalFeature -Online -FeatureName Microsoft-Hyper-V-All
```

Expected:

```text
State : Disabled
```

Check Virtual Machine Platform:

```powershell
Get-WindowsOptionalFeature -Online -FeatureName VirtualMachinePlatform
```

Expected:

```text
State : Disabled
```

Check Windows Hypervisor Platform:

```powershell
Get-WindowsOptionalFeature -Online -FeatureName HypervisorPlatform
```

Expected:

```text
State : Disabled
```

---

### Step 3 – Disable Hypervisor Launch

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

### Step 4 – Investigate Device Guard Scenarios

Run:

```cmd
reg query "HKLM\SYSTEM\CurrentControlSet\Control\DeviceGuard\Scenarios" /s
```

Look for:

```text
SystemGuard Enabled = 1
```

or

```text
SecureBiometrics Enabled = 1
```

---

### Step 5 – Disable Remaining Secure Kernel Components

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

### Step 6 – Reboot

```cmd
shutdown /r /t 0
```

---

### Step 7 – Validate

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

### Step 8 – Launch VMware

Open VMware Workstation.

Verify:

```text
VM Settings
→ Processors
→ Virtualize AMD-V/RVI
```

is enabled.

Power on:

* EVE-NG
* Nested ESXi
* GNS3 VM
* Cisco Modeling Labs

Expected result:

```text
VM boots successfully
Nested virtualization operational
```

---

## TAC-Style Analysis

### Observation

```text
Hyper-V disabled
Hypervisor still detected
VMware nested virtualization failing
```

### Technical Interpretation

```text
Secure Kernel still running
System Guard active
Hypervisor remains loaded
```

### Root Cause

```text
Windows Secure Kernel launched through System Guard
```

### Resolution

```text
Disable SystemGuard
Disable SecureBiometrics
Reboot
Verify HypervisorPresent = False
```

---

## Investigation Summary

This issue was investigated on a Windows 11 Pro Build 26200 system running VMware Workstation Pro 17.6.4 on AMD Ryzen AI hardware.

Initial troubleshooting included:

* Hyper-V removal
* Virtual Machine Platform removal
* Windows Hypervisor Platform removal
* Memory Integrity disabled
* Device Guard disabled
* hypervisorlaunchtype set to Off

Despite these actions, Windows continued reporting:

```text
A hypervisor has been detected
```

Further analysis revealed that System Guard and Secure Biometrics were still allowing the Secure Kernel to remain active.

Disabling those components and rebooting restored full AMD-V/RVI nested virtualization support for VMware Workstation.


## References

Relevant technologies involved:

* VMware Workstation Pro
* Microsoft Hyper-V
* Virtualization-Based Security (VBS)
* Device Guard
* Credential Guard
* System Guard
* Secure Kernel (VTL1)

Useful vendor documentation:

* Microsoft Learn: Virtualization-Based Security
* Microsoft Learn: Hyper-V Troubleshooting
* VMware Workstation Documentation
* VMware Nested Virtualization Documentation
* Windows Device Guard Documentation

---

## Contributing

If this guide helped resolve a similar issue on different hardware, feel free to open an Issue or Pull Request.

Please include:

* CPU model
* Laptop or motherboard model
* Windows build number
* VMware version
* Exact error message
* Resolution steps

Community validation helps improve troubleshooting accuracy for everyone.

---

## Repository Purpose

This repository documents real-world VMware nested virtualization troubleshooting scenarios and provides repeatable fixes for engineers, students, lab builders, and enterprise virtualization administrators.

The goal is to reduce time spent troubleshooting Hyper-V, VBS, Device Guard, System Guard, and AMD-V/RVI conflicts that prevent EVE-NG, ESXi, GNS3, and other nested virtualization workloads from running successfully on modern Windows 11 systems.

---

## Keywords

VMware, VMware Workstation, Nested Virtualization, Windows 11, AMD Ryzen, AMD-V, RVI, Hyper-V, Device Guard, Credential Guard, System Guard, Secure Kernel, VBS, EVE-NG, ESXi, GNS3, Cisco Modeling Labs, CML, Home Lab, CCNP, CCIE, Virtualization, Hypervisor Detected
