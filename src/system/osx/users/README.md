# Users <img style="margin: 6px 13px 0px 0px" align="left" src="../../../data/images/logo_36x36.png" />

### Quick links
- [.. up dir](..)

## User Account

### Account Policy Data
User account policy data can be listed
```bash
$ dscl . -read $HOME accountPolicyData
```

### User Expiration Date
Uses the GNU version of date from brew

```bash
$ date -d @$(dscl . -read $HOME accountPolicyData | grep -A1 passwordLastSetTime | grep -o -E '[0-9]+' | head -n 1)
Mon Aug 21 10:26:49 MDT 2023
```
