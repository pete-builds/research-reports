# GMetrix SMS Licensing & Deployment for Cornell University

**Date:** April 15, 2026  
**Requested by:** Pete Stergion  
**Purpose:** Evaluate whether GMetrix SMS (Skills Management System) can be deployed on Cornell-managed lab computers, with specific focus on licensing, guest logon compatibility, and SCCM deployment.

---

## Executive Summary

GMetrix SMS can likely be deployed on Cornell lab computers, but several areas require direct confirmation from GMetrix sales (801-323-5800). The software supports silent installation via command-line switches (NSIS installer, not MSI), uses a per-machine installation model that should work with shared/guest logon environments, and offers both per-seat and site licensing for educational institutions. However, GMetrix does not publish SCCM-specific documentation, and guest logon compatibility is not explicitly addressed in their public materials.

---

## 1. Licensing Model

### What the license agreement says

**High confidence.** The GMetrix End-User License Agreement defines two relevant licensing tiers:

- **Seat licenses:** "The SOFTWARE PRODUCT may be used on a number of computers equal to the number of seat licenses purchased."
- **Site licenses:** "May be installed on any number of computers within the confines of the physical site."

The licensing is **per-machine, not per-user.** The agreement references computer installations as the basis for deployment rights, not individual user accounts.
[source: https://www.gmetrix.com/license-agreement/]

### How licenses work in practice

Licenses are purchased through authorized GMetrix resellers/distributors. The administrator guide states: "Licenses give an Administrator the 'right' to administer GMetrix practice tests within the terms of that license."
[source: https://www.slideserve.com/ellagonzalez/gmetrix-sms-powerpoint-ppt-presentation]

Students receive **access codes** created from the purchased licenses. Access codes are configured with expiration dates, usage limits, single-user or multi-user modes, and specific products (Word, Excel, etc.). The Certiport admin guide confirms: "you can load the gmetrix on each computer in the lab as the number of students is unlimited."
[source: https://www.certiport.com/portal/common/documentlibrary/gmetrix_admin_panel_getting_started_9_2016.pdf, via search snippet]

### Education pricing reference

SMU charges students $40 for a single product or $75 for a full suite, deployed on public Windows computers across five campus locations.
[source: https://www.smu.edu/oit/training/certification/faq]

### Recommendation

A **site license** is the most practical option for Cornell lab deployment. Contact GMetrix sales or an authorized reseller for institutional pricing. The site license allows installation on any number of machines within the physical site.

---

## 2. Guest Logon / Shared Computer Compatibility

### What is documented

**Medium confidence.** GMetrix SMS installs to **per-machine locations**, not per-user directories:

- Application binaries: `C:\Program Files (x86)\GMetrix\GMetrix SMSe`
- System-wide config: `C:\ProgramData\GMetrixSMSe\Configuration\ApplicationSettings.json`
- User-specific data: `C:\Users\[USERNAME]\AppData\Roaming`

[source: https://support.gmetrix.net/support/solutions/articles/67000509526]
[source: https://support.gmetrix.net/support/solutions/articles/67000701775]

The per-machine installation pattern (Program Files + ProgramData) means the application should be available to any user who logs into the machine, including guest/shared accounts. Students authenticate within the GMetrix application itself using their own GMetrix account credentials, not through Windows logon.

### Real-world shared computer deployments

- **SMU** deploys GMetrix on "public Windows computers" at five campus locations.
  [source: https://www.smu.edu/oit/training/certification/faq]
- **MiraCosta College** has GMetrix SMS "available at various Classroom Computer labs throughout the MiraCosta campuses."
  [source: https://miracosta.atlassian.net/wiki/spaces/KB/pages/2957574217/GMetrix+SMS]

### Potential concerns

- The `AppData\Roaming` directory stores user-specific data. On guest/shared accounts with non-persistent profiles, this data may be wiped on logoff. This could mean students need to re-enter their GMetrix credentials each session, but it should not prevent the application from functioning.
- The admin can lock down settings using the `-admin` flag on the shortcut target to configure options, then remove the flag so regular users cannot change settings.
  [source: https://support.gmetrix.net/support/solutions/articles/67000509516]
- The `ApplicationSettings.json` config file allows disabling user access to settings tabs (User Settings, System Settings, Proxy Settings), which is useful for locked-down lab environments.
  [source: https://support.gmetrix.net/support/solutions/articles/67000701775]

### Recommendation

Guest logon should work based on the per-machine install model and real-world university deployments on public computers. Confirm with GMetrix support that non-persistent user profiles do not cause licensing or activation issues.

---

## 3. SCCM / Enterprise Deployment

### Silent installation support

**High confidence.** GMetrix SMS uses an **NSIS (Nullsoft Scriptable Install System)** installer, not MSI. Silent install is supported:

**Silent install:**
```
GMetrixSMSe-v7.0.24.exe /S
```

**Silent install with custom path:**
```
GMetrixSMSe-v7.0.24.exe /S /D=C:\Program Files (x86)\GMetrix\GMetrix SMSe
```
Note: The `/D` parameter must be the last parameter and must not contain quotes, even if the path has spaces.

**Silent uninstall:**
```
"Uninstall GMetrix SMSe.exe" /S _?=C:\Program Files (x86)\GMetrix\GMetrix SMSe
```

All switches are case-sensitive.
[source: https://support.gmetrix.net/support/solutions/articles/67000660155]

### SCCM compatibility

**Medium confidence.** GMetrix does not publish SCCM-specific documentation. However:

- SCCM can deploy EXE installers with command-line arguments (not just MSI). The `/S` silent switch is compatible with SCCM application packaging.
- ManageEngine Endpoint Central (a comparable endpoint management tool) has documented GMetrix SMSe silent install profiles, confirming the `/S` switch works in automated deployment scenarios.
  [source: https://www.manageengine.com/products/desktop-central/software-installation/silent_install_GMetrix-SMSe-(7.0.27).html]
- The GMetrix mass deployment article references PDQ Deploy compatibility, another enterprise deployment tool similar to SCCM.
  [source: https://support.gmetrix.net/support/solutions/articles/67000660155]

### Post-install configuration

After silent deployment, the `ApplicationSettings.json` file at `C:\ProgramData\GMetrixSMSe\Configuration\` can be pre-configured and pushed alongside the install to:

- Disable automatic updates (`noUpdates: true`)
- Lock down user-accessible settings tabs
- Set proxy configuration
- Configure template/project save locations
- Set Adobe application paths

[source: https://support.gmetrix.net/support/solutions/articles/67000701775]

### System requirements (from search snippet)

- Processor: 1.6GHz dual core
- RAM: 2-4GB
- Disk: 550MB
- OS: Windows Server 2008 or newer (Windows 10/11 implied)
- Requires Microsoft Office or Adobe CC installed on the same machine for those practice tests
- Not compatible with macOS

[source: https://www.gmetrix.net/GetGMetrixSMS.aspx, via search snippet]

### Recommended SCCM deployment approach

1. Download the latest GMetrix SMSe installer EXE
2. Create a pre-configured `ApplicationSettings.json` with updates disabled and settings locked
3. Package in SCCM as an Application with detection rule checking for `C:\Program Files (x86)\GMetrix\GMetrix SMSe\GMetrix SMSe.exe`
4. Install command: `GMetrixSMSe-v7.x.x.exe /S`
5. Uninstall command: `"C:\Program Files (x86)\GMetrix\GMetrix SMSe\Uninstall GMetrix SMSe.exe" /S _?=C:\Program Files (x86)\GMetrix\GMetrix SMSe`
6. Deploy the `ApplicationSettings.json` to `C:\ProgramData\GMetrixSMSe\Configuration\` via a post-install script or separate SCCM package

---

## Open Questions for GMetrix Sales/Support

These items could not be confirmed from public documentation:

1. **Site license pricing for Cornell.** What is the institutional rate, and does it cover multiple buildings/campuses?
2. **Guest/non-persistent profile confirmation.** Does the application require any per-user activation or registration that would fail on wiped guest profiles?
3. **Auto-update behavior in managed environments.** Can the `noUpdates` setting in `ApplicationSettings.json` be set before first launch, or does it require the app to run first?
4. **SCCM return codes.** What exit codes does the NSIS installer return for success/failure? (Standard NSIS returns 0 for success.)
5. **Offline activation.** If lab computers have restricted internet access, does GMetrix SMS require online activation or can it run offline?
6. **.NET Framework dependency.** The offline guide references .NET Framework requirements, but the specific version is not documented. Confirm which version is needed.

**GMetrix Support:** 801-323-5800  
**Support Portal:** https://support.gmetrix.net/support/home

---

## Sources Consulted

| Source | URL | Confidence |
|--------|-----|------------|
| GMetrix License Agreement | https://www.gmetrix.com/license-agreement/ | High |
| GMetrix Mass Install/Uninstall (v7) | https://support.gmetrix.net/support/solutions/articles/67000660155 | High |
| GMetrix Configuration File Docs | https://support.gmetrix.net/support/solutions/articles/67000701775 | High |
| GMetrix SMS Removal Guide | https://support.gmetrix.net/support/solutions/articles/67000509526 | High |
| GMetrix Admin Options | https://support.gmetrix.net/support/solutions/articles/67000509516 | High |
| GMetrix Remote Desktop Guide | https://support.gmetrix.net/support/solutions/articles/67000698728 | High |
| ManageEngine Silent Install Profile | https://www.manageengine.com/products/desktop-central/software-installation/silent_install_GMetrix-SMSe-(7.0.27).html | High |
| SMU GMetrix FAQ | https://www.smu.edu/oit/training/certification/faq | High |
| GMetrix Admin Guide (SlideServe) | https://www.slideserve.com/ellagonzalez/gmetrix-sms-powerpoint-ppt-presentation | Medium |
| MiraCosta College KB | https://miracosta.atlassian.net/wiki/spaces/KB/pages/2957574217/GMetrix+SMS | Medium |
| GMetrix for School Admins | https://examprep.gmetrix.com/for-school-admins/ | Medium |
