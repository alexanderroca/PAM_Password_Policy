# Linux Password Complexity Enforcement (`pam_pwquality`)

This repository provides a production-ready configuration for enforcing corporate password complexity rules across Linux systems using `pam_pwquality`. 

In enterprise environments, implementing password complexity via PAM (Pluggable Authentication Modules) serves as a **Technical Control** that enforces an organization's **Managerial Password Policy**.

---

## 📦 Package Requirements by Distribution

Before configuring `pwquality.conf`, the `pam_pwquality` module must be installed on your Linux distribution:

| Distribution Family | Package Name | Installation Command |
| :--- | :--- | :--- |
| **Debian / Ubuntu / Kali / Mint** | `libpam-pwquality` | `sudo apt update && sudo apt install libpam-pwquality` |
| **RHEL / CentOS / Fedora / Rocky** | `pam_pwquality` | `sudo dnf install pam_pwquality` |
| **Arch Linux / Manjaro** | `libpwquality` | `sudo pacman -S libpwquality` |
| **openSUSE / SLES** | `pam_pwquality` | `sudo zypper install pam_pwquality` |

---

## ⚙️ Configuration (`/etc/security/pwquality.conf`)

Place the following configuration into `/etc/security/pwquality.conf`:

```ini
# Configuration for systemwide password quality limits
# Location: /etc/security/pwquality.conf

# Minimum number of unique characters between old and new passwords
difok = 4

# Minimum acceptable length for new passwords
minlen = 14

# Mandatory character requirements (Negative values = required count)
dcredit = -1    # Requires at least 1 digit (0-9)
ucredit = -1    # Requires at least 1 uppercase letter (A-Z)
lcredit = -1    # Requires at least 1 lowercase letter (a-z)
ocredit = -1    # Requires at least 1 special character (!@#$, etc.)
```

## 🔍 Parameter Breakdown & Mechanics

Key Rule in `pwquality.conf`: Setting a credit value (`dcredit`, `ucredit`, `lcredit`, `ocredit`) to a negative integer converts it from an optional bonus credit into a mandatory minimum requirement.

| Parameter | Value | Functional Description | Security Impact |
| :--- | :--- | :--- | :--- |
| `minlen` | `14` | Sets the minimum password length to 14 characters. | Mitigates offline brute-force and dictionary attacks. |
| `dcredit` | `-1` | Mandates at least 1 numeric digit. | Expands the character search space for cracking tools. |
| `ucredit` | `-1` | Mandates at least 1 uppercase letter. | Prevents simple all-lowercase passphrase patterns. |
| `lcredit` | `-1` | Mandates at least 1 lowercase letter. | Enforces mixed-case character distribution. |
| `ocredit` | `-1` | Mandates at least 1 special character/symbol. | Defeats basic alphanumeric wordlists. |
| `difork` | `4` | Requires 4 characters in the new password to differ from the old one. | Prevents users from making minor single-character changes during password resets. |

## 🛠️ PAM Stack Integration

Installing `libpam-pwquality` on Debian/Ubuntu/Kali automatically registers the module in `/etc/pam.d/common-password`.
Verify that your PAM password stack includes the `pam_pwquality.so` directive:

```ini
# /etc/pam.d/common-password
password    requisite    pam_pwquality.so retry=3
```

(On RHEL/Fedora systems, manage PAM integration using `sudo authselect enable-feature with-pwquality`).

## 🧪 Testing & Verification
To verify that PAM is actively enforcing these rules:
1. Attempt to change a standard (non-root) user password: `passwd <username>`

1. Test a short or weak password (e.g., Password1!).
2. **Expected Output**:

```ini
BAD PASSWORD: The password is shorter than 14 characters
Authentication token manipulation error
```
