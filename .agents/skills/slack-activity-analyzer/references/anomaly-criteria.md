# Anomaly Detection Criteria

## High Severity

### Bulk file operations outside business hours
- 3 or more file_downloaded or file_uploaded events between 00:00-06:00 UTC for the same user
- Indicates potential data exfiltration

### Mass channel deletions
- 3 or more channel_deleted events within a 30-minute window
- Indicates potential sabotage or compromised account

### Privilege escalation
- permission_changed events where a user's role changes from member to admin
  without a corresponding approval pattern (e.g., no prior admin activity from another admin)
- Especially suspicious if combined with other anomalous activity from the same user

### Login from anomalous location
- A user logging in from a source_location that differs from their established pattern
- Look for switches between known office locations and external/unknown locations
- Especially suspicious when combined with a user_agent change (e.g., browser to python-requests)

## Medium Severity

### App installation spike
- 3 or more app_installed events within a 1-hour window
- May indicate unauthorized integration setup or compromised OAuth flow

### Off-hours messaging volume
- More than 5 message_sent events between 00:00-06:00 UTC from a single user in one day
- Not necessarily malicious but worth flagging for review

### Rapid member removal
- 3 or more member_removed events within a 1-hour window
- Could indicate organizational disruption or unauthorized access

## Low Severity

### Unusual user agent strings
- Login events with user_agent values that differ from the user's historical pattern
- May indicate automation or credential sharing

### Single off-hours login
- A login event between 00:00-06:00 UTC without corresponding off-hours activity
- Low risk on its own but worth noting in context of other findings
