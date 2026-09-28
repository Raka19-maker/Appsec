# Android Application Assessment: InsecureBankv2

## Target / lab
- **Application:** InsecureBankv2 (Android APK with Python back-end server)
- **Type:** Android app
- **Setup:** Android emulator with the APK installed, back-end server run locally

## Scope
- **In scope:** InsecureBankv2 APK and its traffic to the local back end
- **Out of scope:** Emulator host
- **Approach:** Static and dynamic testing aligned with OWASP MASVS / Mobile Top 10
- **Tools:** MobSF, jadx, adb, Burp Suite

## Summary of findings

| ID | Finding | Severity | Status |
|----|---------|----------|--------|
| F-01 | Login bypass through exported activity | 🟠 High | Validated |
| F-02 | Credentials stored in SharedPreferences with a hardcoded key | 🟠 High | Validated |
| F-03 | Sensitive data written to logs | 🟡 Medium | Validated |
| F-04 | Debuggable and backup-enabled build | 🟡 Medium | Validated |

## Findings

### F-01: Login bypass through exported activity
- **Severity:** 🟠 High
- **Location:** `com.android.insecurebankv2.PostLogin` (exported in AndroidManifest.xml)
- **Description:** The post-login activity is exported and can be launched directly.
- **Impact:** An attacker or malicious app can open the logged-in screens without credentials.

**Validation steps**
1. MobSF flags `PostLogin` as exported.
2. Run `adb shell am start -n com.android.insecurebankv2/.PostLogin`.
3. The logged-in screen opens without authentication.

**Remediation guidance**
- Set `android:exported="false"` on internal activities.
- Check for a valid session in every protected activity.

### F-02: Credentials stored with a hardcoded key
- **Severity:** 🟠 High
- **Location:** `shared_prefs/mySharedPreferences.xml`; key in `CryptoClass`
- **Description:** Username and password are saved in SharedPreferences, encrypted with an AES key hardcoded in the app.
- **Impact:** Anyone with the device or a backup can decrypt the stored credentials.

**Validation steps**
1. Pull the prefs file with adb.
2. Decompile with jadx and find the hardcoded key in `CryptoClass`.
3. Decrypt the stored password with that key.

**Remediation guidance**
- Don't store passwords on the device; store a session token instead.
- Protect any secrets with Android Keystore and EncryptedSharedPreferences.

### F-03: Sensitive data in logs
- **Severity:** 🟡 Medium
- **Location:** Login and transfer flows
- **Description:** Credentials and transaction details are written to logcat.
- **Impact:** Other apps with log access or anyone with adb can read them.

**Validation steps**
1. Run `adb logcat` while logging in and making a transfer.
2. The username, password and transfer details appear in the log.

**Remediation guidance**
- Remove sensitive logging from release builds (e.g. strip `Log` calls with ProGuard/R8).

### F-04: Debuggable and backup-enabled build
- **Severity:** 🟡 Medium
- **Location:** AndroidManifest.xml (`android:debuggable="true"`, `android:allowBackup="true"`)
- **Description:** The release APK is debuggable and allows adb backups.
- **Impact:** App data can be extracted and the app can be attached to a debugger.

**Validation steps**
1. MobSF reports both flags in the manifest analysis.
2. `adb backup` extracts app data including the prefs file.

**Remediation guidance**
- Set `debuggable="false"` and `allowBackup="false"` for release builds.

## Key takeaways
- Combined static analysis (MobSF, jadx) with dynamic testing (adb, logcat, Burp) to go from automated flags to validated, exploitable issues.
