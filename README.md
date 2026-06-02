# VMware Nested Virtualization Fix (Windows 11 + AMD Ryzen)

> A real-world troubleshooting guide for resolving VMware Workstation nested virtualization failures caused by Hyper-V, VBS, Device Guard, System Guard, Secure Kernel, and modern Windows 11 security features.

---

## ⚠️ Important Notice

This guide disables certain Windows Virtualization-Based Security (VBS) components to restore AMD-V/RVI nested virtualization support in VMware Workstation.

Before proceeding:

* Create a Windows Restore Point.
* Export affected registry keys.
* Understand that disabling security features may reduce certain protections.
* Test changes in a lab environment whenever possible.
* Review organizational security policies before applying changes on corporate devices.

Validated on:

* Windows 11 Pro Build 26200
* VMware Workstation Pro 17.6.4
* AMD Ryzen AI 9 HX 370
* ASUS ROG Zephyrus G16

---

## Common Search Terms

This guide may help if you searched for:

* VMware nested virtualization not working Windows 11
* Virtualized AMD-V/RVI is not supported on this platform
* VMware Workstation does not support nested virtualization on this host
* A hypervisor has been detected
* HypervisorPresent True after disabling Hyper-V
* VMware AMD Ryzen nested virtualization fix
* EVE-NG AMD-V error VMware Workstation
* Nested ESXi VMware Workstation Windows 11
* GNS3 VM nested virtualization failure
* Module 'HV' power on failed
* Windows 11 VBS prevents VMware nested virtualization

---

## Quick Fix (TL;DR)

If:

* Hyper-V is disabled
* Virtual Machine Platform is disabled
* Windows Hypervisor Platform is disabled
* Memory Integrity is disabled

but Windows still reports:

```text
A hypervisor has been detected.
```

and VMware still reports:

```text
Virtualized AMD-V/RVI is not supported on this platform.
```

Check:

```powershell
Get-CimInstance -ClassName Win32_ComputerSystem | Select-Object HypervisorPresent
```

If:

```text
True
```

Run:

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

---

## Solves These Errors

### VMware Errors

```text
Virtualized AMD-V/RVI is not supported on this platform.
```

```text
VMware Workstation does not support nested virtualization on this host.
```

```text
Module 'HV' power on failed.
```

```text
This host supports AMD-V, but AMD-V is disabled.
```

### Windows Indicators

```text
A hypervisor has been detected.
Features required for Hyper-V will not be displayed.
```

```text
HypervisorPresent = True
```

---

## Who Is This Guide For?

This guide is intended for:

* VMware Workstation users
* EVE-NG engineers
* GNS3 users
* Cisco Modeling Labs (CML) users
* Nested ESXi lab builders
* CCNA / CCNP / CCIE candidates
* Enterprise virtualization engineers
* Home lab enthusiasts

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

Successfully validated with:

* EVE-NG
* Nested ESXi
* GNS3 VM
* Cisco Modeling Labs (CML)
* Linux KVM Guests

Successfully resolved:

* AMD-V/RVI failures
* Nested virtualization failures
* Hypervisor detected issues
* VMware startup failures
* Module HV failures

---

## Root Cause

The issue was not Hyper-V itself.

Although Hyper-V, Virtual Machine Platform, and Windows Hypervisor Platform were disabled, Windows continued loading a Secure Kernel through remaining Virtualization-Based Security components.

Investigation identified active Device Guard scenarios including:

```text
SystemGuard = Enabled
```

and

```text
SecureBiometrics = Enabled
```

These components caused Windows to continue loading a hypervisor at boot, preventing VMware from exposing AMD-V/RVI to nested guests.

---

## Investigation Summary

Initial troubleshooting included:

* Hyper-V removal
* Virtual Machine Platform removal
* Windows Hypervisor Platform removal
* Memory Integrity disabled
* Device Guard disabled
* hypervisorlaunchtype set to Off

Despite those changes:

```text
A hypervisor has been detected.
```

continued to appear.

Registry analysis revealed System Guard and Secure Biometrics remained active and continued launching the Secure Kernel.

Disabling those components resolved the issue.

---

## Full Troubleshooting Workflow

### Verify Hypervisor Status

```powershell
Get-CimInstance -ClassName Win32_ComputerSystem | Select-Object HypervisorPresent
```

Expected problematic result:

```text
True
```

---

### Verify Hyper-V Components

```powershell
Get-WindowsOptionalFeature -Online -FeatureName Microsoft-Hyper-V-All
```

```powershell
Get-WindowsOptionalFeature -Online -FeatureName VirtualMachinePlatform
```

```powershell
Get-WindowsOptionalFeature -Online -FeatureName HypervisorPlatform
```

Expected:

```text
State : Disabled
```

---

### Disable Hypervisor Launch

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

---

### Check Device Guard Scenarios

```cmd
reg query "HKLM\SYSTEM\CurrentControlSet\Control\DeviceGuard\Scenarios" /s
```

Look for:

```text
SystemGuard
SecureBiometrics
```

---

### Disable Remaining Secure Kernel Components

```cmd
reg add "HKLM\SYSTEM\CurrentControlSet\Control\DeviceGuard\Scenarios\SystemGuard" /v Enabled /t REG_DWORD /d 0 /f
```

```cmd
reg add "HKLM\SYSTEM\CurrentControlSet\Control\DeviceGuard\Scenarios\SecureBiometrics" /v Enabled /t REG_DWORD /d 0 /f
```

---

### Reboot

```cmd
shutdown /r /t 0
```

---

### Validate Success

```powershell
Get-CimInstance -ClassName Win32_ComputerSystem | Select-Object HypervisorPresent
```

Expected:

```text
False
```

---

### Launch VMware

Verify:

```text
VM Settings
→ Processors
→ Virtualize AMD-V/RVI
```

is enabled.

Start:

* EVE-NG
* ESXi
* GNS3 VM
* Cisco Modeling Labs

Nested virtualization should now function normally.

---

## Screenshots

Create an `images` folder and add:

* VMware AMD-V/RVI error
* HypervisorPresent = True
* systeminfo showing hypervisor detected
* Registry showing SystemGuard enabled
* Successful EVE-NG or ESXi boot

Example:

```markdown
![VMware Error](images/vmware-error.png)

![Hypervisor Present](images/hypervisor-present.png)

![Successful Boot](images/success.png)
```

---

## TAC-Style Analysis

### Observation

```text
Hyper-V disabled
VMware nested virtualization failing
Hypervisor still detected
```

### Technical Interpretation

```text
Secure Kernel still running
System Guard still active
Hypervisor still loaded at boot
```

### Root Cause

```text
Windows Secure Kernel launched through remaining Device Guard scenarios.
```

### Resolution

```text
Disable SystemGuard
Disable SecureBiometrics
Reboot
Verify HypervisorPresent = False
```

---

## References

Relevant technologies:

* VMware Workstation Pro
* Hyper-V
* Virtualization-Based Security (VBS)
* Device Guard
* Credential Guard
* System Guard
* Secure Kernel

Useful documentation:

* Microsoft Learn: Hyper-V
* Microsoft Learn: VBS
* VMware Workstation Documentation
* VMware Nested Virtualization Documentation

---

## Contributing

Found a similar issue?

Please open an Issue or Pull Request and include:

* CPU model
* Laptop or motherboard model
* Windows build
* VMware version
* Exact error
* Resolution steps

Community contributions improve troubleshooting accuracy.

---

## Repository Purpose

This repository documents real-world VMware nested virtualization troubleshooting and provides repeatable fixes for engineers, students, lab builders, and enterprise administrators.

The goal is to reduce time spent troubleshooting Hyper-V, VBS, Device Guard, System Guard, Secure Kernel, and AMD-V/RVI conflicts on modern Windows 11 systems.

---

## License

MIT License

---

## Keywords

VMware, VMware Workstation, Nested Virtualization, Windows 11, AMD Ryzen, AMD-V, RVI, Hyper-V, Device Guard, Credential Guard, System Guard, VBS, EVE-NG, ESXi, GNS3, Cisco Modeling Labs, CML, Home Lab, CCNP, CCIE, Virtualization, Hypervisor Detected
