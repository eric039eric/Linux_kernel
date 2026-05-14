# Patch: docs: hwmon: sy7636a: fix temperature sysfs attribute name

## Overview

| Field | Value |
|---|---|
| Date | 2026-05-14 |
| Commit | `0ace4a32eb76fff51f53b4c077dc63001fd3fb74` |
| Patch subject | `[PATCH] docs: hwmon: sy7636a: fix temperature sysfs attribute name` |
| Message-ID | `<20260514154108.1937-1-eric039eric@gmail.com>` |
| Upstream status | Submitted via `git send-email` to Linux kernel mailing list |
| File changed | `Documentation/hwmon/sy7636a-hwmon.rst` |
| Lines changed | 1 insertion, 1 deletion |

## What was changed

Changed the documented hwmon sysfs temperature attribute name in
`Documentation/hwmon/sy7636a-hwmon.rst` from:

```
temp0_input
```

to:

```
temp1_input
```

## Why this change is needed

The hwmon subsystem defines a generic sysfs naming convention in
`Documentation/hwmon/sysfs-interface.rst`:

- Temperature channels: `temp[1-*]_input` (numbering starts from **1**)
- Voltage channels: `in[0-*]_input` (numbering starts from **0**, as an exception)

The file `Documentation/hwmon/sy7636a-hwmon.rst` previously documented
the temperature sensor attribute as `temp0_input`, which is inconsistent
with the generic hwmon naming convention.

## Validation process

This was not changed based on guesswork. The following verification steps
were performed before deciding to submit this patch:

### Step 1: Confirm the generic naming rule

```
grep -n "Numbering usually starts from 1" Documentation/hwmon/sysfs-interface.rst
# → Line 50: ...Numbering usually starts from 1, except for voltages which start from 0...

grep -n "temp\[1-\*\]_input" Documentation/hwmon/sysfs-interface.rst
# → Line 251: `temp[1-*]_input`

grep -n "in\[0-\*\]_input" Documentation/hwmon/sysfs-interface.rst
# → Line 126: `in[0-*]_input`
```

### Step 2: Confirm the driver uses standard hwmon temperature channel interface

```
grep -n "hwmon_temp\|hwmon_temp_input\|HWMON_CHANNEL_INFO(temp" drivers/hwmon/sy7636a-hwmon.c
# → Line 41:  if (type != hwmon_temp)
# → Line 44:  if (attr != hwmon_temp_input)
# → Line 57:  HWMON_CHANNEL_INFO(temp, HWMON_T_INPUT),
```

### Step 3: Confirm hwmon core generates sysfs names via standard template

```
grep -n "hwmon_temp_input" drivers/hwmon/hwmon.c
# → [hwmon_temp_input] = "temp%d_input"
```

This means for a standard hwmon temperature channel, the sysfs file
name is generated as `temp%d_input` where `%d` starts from **1**.

### Step 4: Cross-reference with other hwmon documentation files

```
git grep -n "temp0_input" Documentation/hwmon
# → Only 2 files: sy7636a-hwmon.rst and xgene-hwmon.rst

git grep -n "temp1_input" Documentation/hwmon
# → Appears in 100+ driver documentation files
```

`temp1_input` is clearly the mainstream convention across hwmon drivers.

### Step 5: Checkpatch verification

```
./scripts/checkpatch.pl --strict --codespell \
    0001-docs-hwmon-sy7636a-fix-temperature-sysfs-attribute-n.patch

# total: 0 errors, 0 warnings, 0 checks, 6 lines checked
# ...has no obvious style problems and is ready for submission.
```

## Recipients (from get_maintainer.pl)

```
./scripts/get_maintainer.pl -f Documentation/hwmon/sy7636a-hwmon.rst
```

| Role | Name / Address |
|---|---|
| Maintainer (hwmon) | Guenter Roeck `<linux@roeck-us.net>` |
| Maintainer (docs) | Jonathan Corbet `<corbet@lwn.net>` |
| Reviewer (docs) | Shuah Khan `<skhan@linuxfoundation.org>` |
| Mailing list | `linux-hwmon@vger.kernel.org` |
| Mailing list | `linux-doc@vger.kernel.org` |
| Mailing list | `linux-kernel@vger.kernel.org` |

## Upstream submission log

```
Server: smtp.gmail.com
RCPT TO: linux@roeck-us.net
RCPT TO: corbet@lwn.net
RCPT TO: skhan@linuxfoundation.org
RCPT TO: linux-hwmon@vger.kernel.org
RCPT TO: linux-doc@vger.kernel.org
RCPT TO: linux-kernel@vger.kernel.org
Result: 250
```

## Key learning from this patch

This patch was not discovered by accident or guessing. It was found through
a systematic verification process:

1. **Generic rule first** — read `sysfs-interface.rst` to understand the
   naming convention before touching any specific file.
2. **Document vs. rule comparison** — spotted the inconsistency between
   `temp0_input` in the driver doc and `temp[1-*]_input` in the generic rule.
3. **Driver code verification** — confirmed the driver uses standard hwmon
   temperature channel interface, not a custom naming scheme.
4. **Core template verification** — confirmed hwmon core generates `temp%d_input`,
   not `temp%d_input` starting from 0.
5. **Cross-reference** — compared with 100+ other hwmon docs to confirm
   `temp1_input` is the convention.

This process of **rule → specific doc → driver code → core → cross-reference**
is the correct approach for any hwmon documentation review.
