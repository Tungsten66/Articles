# Microsoft Defender for Endpoint on iOS — Test Plan

*Last updated: March 26, 2026*

**Deployment Model:** Supervised device | Zero Touch (Silent) Control Filter | Apple Business Manager | DisableSignOut = true

**References:**
- [Deploy MDE on iOS with Microsoft Intune](https://learn.microsoft.com/en-us/defender-endpoint/ios-install)
- [Configure MDE on iOS features](https://learn.microsoft.com/en-us/defender-endpoint/ios-configure-features)

## Prerequisites

| # | Requirement | Details |
|---|-------------|---------|
| 1 | iOS/iPadOS device running **iOS 16.0+** | Physical device enrolled as **Supervised** via Apple Business Manager (ABM) |
| 2 | Apple Business Manager configured | Device assigned in ABM with an Automated Device Enrollment (ADE) program token linked to Intune as the MDM server |
| 3 | Microsoft Intune license assigned to test user | Required for device enrollment and policy delivery |
| 4 | Microsoft Defender for Endpoint **Plan 1 or Plan 2** license | Assigned to the test user |
| 5 | Microsoft Defender app deployed via Intune | Added as an iOS store app (or Volume Purchase Program (VPP) app) and assigned as **Required** |
| 6 | Intune Company Portal installed | Deployed via ABM/VPP |
| 7 | Defender and Intune connector enabled | **Settings → Endpoints → Advanced Features → Microsoft Intune Connection** in the Defender portal |
| 8 | Network connectivity to MDE service URLs | [Firewall/proxy configured](https://learn.microsoft.com/en-us/defender-endpoint/configure-environment) |
| 9 | `issupervised` app configuration policy deployed | Key: `issupervised`, Value: `{{issupervised}}`, Type: String — targeted to all managed iOS devices |
| 10 | Zero Touch (Silent) Control Filter profile deployed | Custom configuration profile using `ControlFilterZeroTouch.mobileconfig` deployed via Intune |
| 11 | `DisableSignOut = true` app configuration policy deployed | Key: `DisableSignOut`, Value: `true`, Type: String — targeted to all managed iOS devices |

---

## Intune Configuration Summary

The following Intune policies must be in place before testing begins. All other Defender for Endpoint settings not explicitly listed below use their default values.

### App Configuration Policy — Managed Devices

| Key | Value Type | Value | Purpose |
|-----|-----------|-------|---------|
| `issupervised` | String | `{{issupervised}}` | Enables supervised-mode features in the Defender app |
| `DisableSignOut` | String | `true` | Hides the Sign Out button in the Defender app |

### Device Configuration Profile — Custom (Control Filter)

| Setting | Value |
|---------|-------|
| Profile Type | Templates → Custom |
| `.mobileconfig` file | [ControlFilterZeroTouch.mobileconfig](https://download.microsoft.com/download/f/8/e/f8ed3484-b665-4c3c-9ae9-272c8a04159b/Microsoft_Defender_for_Endpoint_Control_Filter_Zerotouch.mobileconfig) |
| Assignment | All managed iOS devices |

**Note:** The Zero Touch Control Filter profile provides Web Protection without a local loopback VPN and enables silent onboarding — users do not need to open the app to activate protection.

### Apple Business Manager

| Setting | Value |
|---------|-------|
| MDM Server | Microsoft Intune |
| Device Assignment | Assigned to Intune with Supervised enrollment profile |
| Enrollment Type | Automated Device Enrollment (ADE) — Supervised |

---

## Test Cases

### TC-01: ABM Enrollment & Supervised Mode Verification

| Step | Action | Expected Result |
|------|--------|-----------------|
| 1 | Factory reset (or unbox) the iOS device assigned in Apple Business Manager | Device powers on and displays the Remote Management enrollment screen |
| 2 | Complete the Setup Assistant — the device auto-enrolls into Intune as a supervised device | Enrollment completes; the device appears in the **Intune admin center → Devices → iOS/iPadOS** with Supervised = Yes |
| 3 | Verify the device is marked as **Supervised** in **Settings → General → About** | "This iPhone is supervised and managed by [Organization]" is displayed |
| 4 | Confirm the Microsoft Defender app is automatically installed via Intune | The Defender app icon appears on the home screen without user action |
| 5 | Confirm the `issupervised` app config policy and Control Filter profile are applied | In Intune: **Device → Device configuration** shows the profiles as successfully applied |

**Portal validation:** Intune admin center → **Devices** → select device → **Device configuration** → verify profiles installed

---

### TC-02: Zero Touch (Silent) Onboarding

| Step | Action | Expected Result |
|------|--------|-----------------|
| 1 | After enrollment and policy sync, wait up to 5 minutes for silent onboarding to complete | The Defender app activates automatically — no user interaction required |
| 2 | Verify the device appears in the **Microsoft Defender portal → Device Inventory** | Device is listed with a heartbeat, iOS version, and onboarding timestamp |
| 3 | On the device, open the Defender app | The dashboard shows **"Device is protected"** with no setup wizard prompts |
| 4 | Verify that **no VPN profile** is installed on the device (Settings → General → VPN & Device Management → VPN) | No Defender VPN profile is present — Web Protection is delivered via the Control Filter |
| 5 | Check the Control Filter status in Defender app settings | Web Protection is active and using the Control Filter (not VPN) |

**Note:** For first-time onboarding, a silent notification may be sent. If MFA or password changes are required, the user may need to sign in manually once.

**Portal validation:** Security.microsoft.com → **Assets → Devices** → search for the device name

---

### TC-03: DisableSignOut Verification

| Step | Action | Expected Result |
|------|--------|-----------------|
| 1 | Confirm `DisableSignOut = true` policy is deployed via Intune app config | Policy shows as applied in Intune device configuration |
| 2 | Open the Defender app and navigate to the app menu/settings | The **Sign Out** button is **not visible** |
| 3 | Verify the user cannot sign out through any in-app method | No sign-out option is available anywhere in the app |

---

### TC-04: Web Protection — Anti-Phishing

| Step | Action | Expected Result |
|------|--------|-----------------|
| 1 | Open Safari (or any browser) on the iOS device | Browser launches normally |
| 2 | Browse to `https://smartscreentestratings2.net/` | Defender **blocks** the page and displays a warning that the site is unsafe/phishing |
| 3 | Attempt to browse to a known-safe site (e.g., `https://microsoft.com`) | Page loads without any block or warning |
| 4 | Check the Defender app on the device | A **Web Protection alert** is shown for the blocked phishing URL |
| 5 | Check the **Microsoft Defender portal → Alerts** | An alert is generated for the phishing URL detection |

**Additional phishing test URLs:**

| URL | Type | Expected |
|-----|------|----------|
| `https://smartscreentestratings2.net/` | Phishing test page | Blocked |
| `http://smartscreentestratings2.net/` | Phishing test page (HTTP) | Blocked |
| `https://demo.smartscreen.msft.net/` | Microsoft SmartScreen demo page | Use the phishing/malware demo links on this page |

**Note:** On supervised devices with the Control Filter profile, Defender has access to the entire URL. If the URL is found to be phishing, it is blocked.

---

### TC-05: Web Protection — Custom Indicators (URL/Domain)

**Note:** IP-based custom indicators are **not supported** on iOS. Only URL and domain indicators are supported. On supervised devices, the URL is included in alerts (not hidden for privacy).

| Step | Action | Expected Result |
|------|--------|-----------------|
| 1 | In the Defender portal, go to **Settings → Endpoints → Indicators → URLs/Domains** | Indicator management page opens |
| 2 | Add a custom **Block** indicator for a test domain (e.g., `testdomain.example.com`) | Indicator is saved successfully |
| 3 | Wait for the indicator to sync to the device (~15–30 min or force sync via Company Portal) | Policy syncs |
| 4 | On the iOS device, browse to the blocked domain | Defender blocks the page and shows a warning |
| 5 | Add a custom **Warn** indicator for a different test domain | Indicator is saved |
| 6 | Browse to the warned domain on the device | Defender shows a warning but allows the user to proceed |
| 7 | Remove the custom indicators after testing | Indicators are deleted successfully |

---

### TC-06: Network Protection — Open Wi-Fi Detection

| Step | Action | Expected Result |
|------|--------|-----------------|
| 1 | Verify Network Protection and Open Network Detection are using defaults (both enabled) — no app config policy changes needed | No custom policy required |
| 2 | Grant **Location permission** ("Always") to the Defender app | Permission granted |
| 3 | Connect the device to an **open (unsecured) Wi-Fi network** | Defender detects the open network and generates an event in the device timeline |
| 4 | Check the **Microsoft Defender portal → Device Timeline** | An event is logged for the open network connection |
| 5 | Disconnect from the open network and connect to a secured network | The event reflects the disconnection |

**Note:** Starting May 2025, open Wi-Fi connections generate **timeline events** instead of alerts. Connection/disconnection events within the same 24-hour period are deduplicated.

---

### TC-07: Network Protection — Rogue Certificate Detection

| Step | Action | Expected Result |
|------|--------|-----------------|
| 1 | Ensure **Certificate Detection** is not disabled (enabled by default) | Default config in place |
| 2 | Install a **self-signed or untrusted certificate** on the device via a profile | Defender detects the suspicious certificate and alerts the user |
| 3 | Check the Defender app on the device | A certificate-related alert or notification is shown |
| 4 | Check the **Microsoft Defender portal → Device Timeline** | An event is logged for the certificate detection |
| 5 | Remove the untrusted certificate profile from the device | The alert is resolved |

---

### TC-08: Jailbreak Detection

| Step | Action | Expected Result |
|------|--------|-----------------|
| 1 | Use a **known jailbroken device** (or simulate jailbreak detection, if available in a test environment) | Device is detected as jailbroken |
| 2 | Open the Defender app on the jailbroken device | Defender reports a **high-risk** alert for jailbreak detection |
| 3 | Check the **Microsoft Defender portal → Alerts** | A high-risk alert is generated |
| 4 | Verify device compliance in the **Intune admin center** | Device is marked as **non-compliant** (if jailbreak compliance policy is configured with `Block` for jailbroken devices) |
| 5 | Verify Conditional Access blocks corporate resource access (if configured) | Access to corporate resources (e.g., Exchange Online, SharePoint) is denied |

**Note:** When jailbreak is detected, user data on the Defender app is cleared and the VPN profile (if any) is deleted. Intune-delivered VPN profiles are not removed.

---

### TC-09: Device Compliance & Conditional Access

| Step | Action | Expected Result |
|------|--------|-----------------|
| 1 | Configure a **device compliance policy** in Intune that requires Defender threat level at or below Low | Policy is deployed |
| 2 | Configure a **Conditional Access policy** in Entra ID that requires device compliance for resource access | Policy is active |
| 3 | With no active threats, access a corporate resource (e.g., Outlook, Teams) | Access is **granted** |
| 4 | Trigger a threat (e.g., browse to `https://smartscreentestratings2.net/`) | Defender raises the device risk level |
| 5 | Attempt to access the same corporate resource | Access is **blocked** due to non-compliance |
| 6 | Remediate the threat (dismiss/resolve the alert in Defender) | Device risk level returns to normal, access is restored |

---

### TC-10: Vulnerability Assessment of Apps (Supervised)

| Step | Action | Expected Result |
|------|--------|-----------------|
| 1 | In the Intune admin center, go to **Endpoint Security → Microsoft Defender for Endpoint** and enable **App sync for iOS/iPadOS devices** | Toggle is enabled |
| 2 | Ensure the device is onboarded and the Defender app is active | Device shows in the portal |
| 3 | In the Defender portal, go to **Vulnerability Management → Software Inventory** | Installed apps from the supervised device are listed |
| 4 | Install an app with a **known vulnerability** on the device | The vulnerable app appears in the software inventory |
| 5 | Check **Vulnerability Management → Weaknesses** | The CVE for the vulnerable app is listed with remediation recommendations |
| 6 | Update or remove the vulnerable app | The vulnerability is resolved in the portal |

**Note:** For supervised corporate devices, the admin does **not** need to enable "Send full application inventory data on personally owned iOS/iPadOS devices." Full app inventory is collected by default for corporate-supervised devices.

---

### TC-11: Privacy Controls (Supervised)

**Note:** On supervised devices, end-user privacy controls are **not visible** — the admin fully controls privacy settings.

| Step | Action | Expected Result |
|------|--------|-----------------|
| 1 | Verify the default setting: `DefenderExcludeURLInReport` is **not configured** (defaults to `false`) | Phishing alerts include the full URL/domain in the Defender portal |
| 2 | Browse to a phishing URL (e.g., `https://smartscreentestratings2.net/`) | The alert in the portal **includes** the domain name |
| 3 | Open the Defender app → Settings | No end-user privacy toggle is displayed (supervised device) |

---

### TC-12: Control Filter Validation (No VPN)

| Step | Action | Expected Result |
|------|--------|-----------------|
| 1 | On the device, go to **Settings → General → VPN & Device Management → VPN** | No Defender for Endpoint VPN profile is installed |
| 2 | Browse to a phishing URL (e.g., `https://smartscreentestratings2.net/`) | Defender blocks the page via the Control Filter — **not** via VPN |
| 3 | Browse to a safe URL (e.g., `https://microsoft.com`) | Page loads normally with no interference |
| 4 | Verify in the Defender app that Web Protection is active | Status shows Web Protection enabled via Control Filter |
| 5 | Attempt to use a corporate VPN (if configured) alongside Defender | Both the corporate VPN and the Control Filter coexist without conflict |

**Note:** The Control Filter profile does **not** work with Always-On VPN (AOVPN) due to platform restrictions.

---

## Test Results Summary

| Test Case | Description | Status | Tester | Date | Notes |
|-----------|-------------|--------|--------|------|-------|
| TC-01 | ABM Enrollment & Supervised Mode | [ ] Pass [ ] Fail | | | |
| TC-02 | Zero Touch (Silent) Onboarding | [ ] Pass [ ] Fail | | | |
| TC-03 | DisableSignOut Verification | [ ] Pass [ ] Fail | | | |
| TC-04 | Web Protection — Anti-Phishing | [ ] Pass [ ] Fail | | | |
| TC-05 | Custom Indicators (URL/Domain) | [ ] Pass [ ] Fail | | | |
| TC-06 | Network Protection — Open Wi-Fi | [ ] Pass [ ] Fail | | | |
| TC-07 | Network Protection — Certificate Detection | [ ] Pass [ ] Fail | | | |
| TC-08 | Jailbreak Detection | [ ] Pass [ ] Fail | | | |
| TC-09 | Device Compliance & Conditional Access | [ ] Pass [ ] Fail | | | |
| TC-10 | Vulnerability Assessment (TVM) | [ ] Pass [ ] Fail | | | |
| TC-11 | Privacy Controls (Supervised) | [ ] Pass [ ] Fail | | | |
| TC-12 | Control Filter Validation (No VPN) | [ ] Pass [ ] Fail | | | |

## References

- [Deploy MDE on iOS with Microsoft Intune](https://learn.microsoft.com/en-us/defender-endpoint/ios-install)
- [Configure MDE on iOS features](https://learn.microsoft.com/en-us/defender-endpoint/ios-configure-features)
- [Web Protection overview](https://learn.microsoft.com/en-us/defender-endpoint/web-protection-overview)
- [Custom Indicators overview](https://learn.microsoft.com/en-us/defender-endpoint/indicators-overview)
- [Microsoft Defender Vulnerability Management](https://learn.microsoft.com/en-us/defender-vulnerability-management/defender-vulnerability-management)
- [Apple Business Manager](https://business.apple.com/)
- [Intune iOS/iPadOS enrollment](https://learn.microsoft.com/en-us/intune/intune-service/fundamentals/deployment-guide-enrollment-ios-ipados)