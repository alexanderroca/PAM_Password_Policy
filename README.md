# Linux Password Complexity & Account Lockout Enforcement (`PAM`)

This repository provides a production-ready configuration for enforcing corporate password complexity rules via `pam_pwquality` and automated brute-force protection via `pam_faillock` across Linux systems using Pluggable Authentication Modules (PAM).

In enterprise environments, implementing these controls acts as a critical **Technical Control** that enforces an organization's **Managerial Security Policy**.

---

## 📦 Package Requirements by Distribution

Before configuring `pwquality.conf`, the `pam_pwquality` module must be installed on your Linux distribution:

| Distribution Family | Package Name | Installation Command |
| :--- | :--- | :--- |
| **Debian / Ubuntu / Kali / Mint** | `libpam-pwquality` | `sudo apt update && sudo apt install libpam-pwquality` |
| **RHEL / CentOS / Fedora / Rocky** | `pam_pwquality` | `sudo dnf install pam_pwquality` |

---

## ⚙️ Password Complexity Configuration (`/etc/security/pwquality.conf`)

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

### 🔍 Parameter Breakdown & Mechanics

*Key Rule*: Setting credit values to a negative integer converts them from optional bonus credits into mandatory minimum requirements.

| Parameter | Value | Functional Description | Security Impact |
| :--- | :--- | :--- | :--- |
| `minlen` | `14` | Sets the minimum password length to 14 characters. | Mitigates offline brute-force and dictionary attacks. |
| `dcredit` | `-1` | Mandates at least 1 numeric digit. | Expands the character search space for cracking tools. |
| `ucredit` | `-1` | Mandates at least 1 uppercase letter. | Prevents simple all-lowercase passphrase patterns. |
| `lcredit` | `-1` | Mandates at least 1 lowercase letter. | Enforces mixed-case character distribution. |
| `ocredit` | `-1` | Mandates at least 1 special character/symbol. | Defeats basic alphanumeric wordlists. |
| `difork` | `4` | Requires 4 characters in the new password to differ from the old one. | Prevents users from making minor single-character changes during password resets. |

## Account Lockout Configuration (`/etc/security/faillock.conf`)

To protect accounts against brute-force and credential-stuffing attacks without manually corrupting low-level stack files, configure `pam_faillock` globally.
1. Open or create `/etc/security/faillock.conf`:
```Bash
sudo vim /etc/security/faillock.conf
```

2. Add the following production-ready parameters:
```Plaintext
# Configuration for locking the user after multiple failed
# authentication attempts.
#
# The directory where the user files with the failure records are kept.
# The default is /var/run/faillock.
dir = /var/lib/faillock
#
# Will log the user name into the system log if the user is not found.
# Enabled if option is present.
# audit
#
# Don't print informative messages.
# Enabled if option is present.
# silent
#
# Don't log informative messages via syslog.
# Enabled if option is present.
# no_log_info
#
# Only track failed user authentications attempts for local users
# in /etc/passwd and ignore centralized (AD, IdM, LDAP, etc.) users.
# The `faillock` command will also no longer track user failed
# authentication attempts. Enabling this option will prevent a
# double-lockout scenario where a user is locked out locally and
# in the centralized mechanism.
# Enabled if option is present.
# local_users_only
#
# Deny access if the number of consecutive authentication failures
# for this user during the recent interval exceeds n tries.
# The default is 3.
deny = 3
#
# The length of the interval during which the consecutive
# authentication failures must happen for the user account
# lock out is <replaceable>n</replaceable> seconds.
# The default is 900 (15 minutes).
fail_interval = 900
#
# The access will be re-enabled after n seconds after the lock out.
# The value 0 has the same meaning as value `never` - the access
# will not be re-enabled without resetting the faillock
# entries by the `faillock` command.
# The default is 600 (10 minutes).
unlock_time = 600
#
# Root account can become locked as well as regular accounts.
# Enabled if option is present.
even_deny_root
#
# This option implies the `even_deny_root` option.
# Allow access after n seconds to root account after the
# account is locked. In case the option is not specified
# the value is the same as of the `unlock_time` option.
root_unlock_time = 900
#
# If a group name is specified with this option, members
# of the group will be handled by this module the same as
# the root account (the options `even_deny_root>` and
# `root_unlock_time` will apply to them.
# By default, the option is not set.
# admin_group = <admin_group_name>
```

## 🛠️ Managing via pam-auth-update

Operational Warning: Avoid manual text edits of core shared files like `/etc/pam.d/common-auth` or `/etc/pam.d/common-account`. Manual injection mistakes (such as mixing auth blocks into account files) will corrupt the PAM chain and cause unexpected permission lockouts. Always use Debian's native utility to manage modules.

### Managing via `pam-auth-update`

To safely enable and register account lockout handling across all system services:
```Bash
sudo pam-auth-update --enable faillock
```

This safely wires the required `preauth` and `authfail` logic into `common-auth` and appends `account required pam_faillock.so` into `common-account` under the hood.

## 🧪 Testing & Verification

1. **Verify Password Complexity Rules**:

    1. Attempt to change a user password: `passwd <username>`
    2. Test a short or weak password (e.g., Password1!).
    3. **Expected Output**:
    ```ini
    BAD PASSWORD: The password is shorter than 14 characters
    Authentication token manipulation error
    ```

2. **Verify Account Lockout Status**:

    1. Check failure counts for a specific user:
    ```Bash
    sudo faillock --user <username>
    ```
    2. Manually clear lockout records:
    ```Bash
    sudo faillock --user <username> --reset
    ```



