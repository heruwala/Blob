# GitHub Copilot Instructions for Feature Flag Maintenance

## Repository Overview
This repository contains feature flag configurations for Production, QA, and Dev environments across multiple UI portals. The primary use case is managing monthly maintenance windows for production applications.

## Monthly Maintenance Process

### Purpose
Enable impending maintenance messages in Production UI portals when operations communicates scheduled maintenance windows. All timestamps operate in Eastern Time zone but must be stored in UTC format.

### Production Applications and Portals

| Application | File Pattern | Portals Count | Timestamp Prefix |
|-------------|--------------|---------------|------------------|
| AHP (Applicant/Healthcare Professionals) | `AHP_Feature_Flag_PRD-{on\|off}.json` | 4 | `MAINTENANCE_*_TIMESTAMP_UI_*` |
| DVI (Data Visualization) | `DVI_Feature_Flag_PRD-{on\|off}.json` | 1 | `MAINTENANCE_*_TIMESTAMP_UI_*` |
| EFM (ERAS Fellowship Documents Office) | `EFM_Feature_Flag_PRD-{on\|off}.json` | 3 | `MAINTENANCE_*_TIMESTAMP_*` |
| ESP (EPIC Secure Portal) | `ESP_Feature_Flag_PRD-{on\|off}.json` | 1 | `MAINTENANCE_MSG_*_TIMESTAMP_*` |
| PWY (Pathways) | `PWY_Feature_Flag_PRD-{on\|off}.json` | 3 | `MAINTENANCE_MSG_*_TIMESTAMP_*` |

**Total**: 10 files, 12 portals, 72 timestamp fields to update

### Timestamp Fields Structure

Each portal configuration contains three critical timestamp fields:

#### 1. IMPENDING_TIMESTAMP
- **Purpose**: When to start displaying the impending maintenance message to users
- **Value**: Current UTC time when the maintenance update is being applied
- **Critical Rule**: Must be **earlier** than START_TIMESTAMP
- **Format**: ISO 8601 with milliseconds (e.g., `2025-11-08T14:30:00.000Z`)

#### 2. START_TIMESTAMP
- **Purpose**: When the maintenance window begins
- **Value**: User-provided start time converted from Eastern Time to UTC
- **Format**: ISO 8601 with milliseconds (e.g., `2025-11-07T22:00:00.000Z`)

#### 3. END_TIMESTAMP
- **Purpose**: When the maintenance window ends
- **Value**: User-provided end time converted from Eastern Time to UTC
- **Format**: ISO 8601 with milliseconds (e.g., `2025-11-08T23:00:00.000Z`)

### Workflow for Maintenance Updates

When a user requests maintenance timestamp updates, follow these steps:

#### Step 1: Prompt for Maintenance Schedule
Ask the user:
```
Please provide the maintenance schedule in this format:
[Day] – [Month] [DD], [YYYY], starting at [H:MM AM/PM] EST through [Day] - [Month] [DD], [YYYY], [H:MM AM/PM] EST

Example: Friday – November 07, 2025, starting at 5:00PM EST through Saturday - November 08, 2025, 6:00PM EST
```

#### Step 2: Parse and Convert Timestamps

**Eastern Time to UTC Conversion**:
- **EST (Eastern Standard Time)**: Add 5 hours (UTC-5)
  - Active: First Sunday in November through second Sunday in March
- **EDT (Eastern Daylight Time)**: Add 4 hours (UTC-4)
  - Active: Second Sunday in March through first Sunday in November

**Examples**:
```
5:00 PM EST → 22:00 UTC (10:00 PM UTC)
6:00 PM EST → 23:00 UTC (11:00 PM UTC)
5:00 PM EDT → 21:00 UTC (9:00 PM UTC)
```

#### Step 3: Determine IMPENDING_TIMESTAMP
- Use current UTC time when applying the update
- **Validation**: If current time is already past START_TIMESTAMP, set IMPENDING to a reasonable time before START (e.g., 1 hour before)
- **Critical**: Ensure IMPENDING < START < END

#### Step 4: Update All Production Files

Update timestamp fields in all 10 files in the `Prod/` folder:

**AHP Files** (lowercase `name` property):
- `MAINTENANCE_IMPENDING_TIMESTAMP_UI_APPLICANT_PORTAL`
- `MAINTENANCE_START_TIMESTAMP_UI_APPLICANT_PORTAL`
- `MAINTENANCE_END_TIMESTAMP_UI_APPLICANT_PORTAL`
- `MAINTENANCE_IMPENDING_TIMESTAMP_UI_ENTITY_PORTAL`
- `MAINTENANCE_START_TIMESTAMP_UI_ENTITY_PORTAL`
- `MAINTENANCE_END_TIMESTAMP_UI_ENTITY_PORTAL`
- `MAINTENANCE_IMPENDING_TIMESTAMP_UI_STAFF_PORTAL`
- `MAINTENANCE_START_TIMESTAMP_UI_STAFF_PORTAL`
- `MAINTENANCE_END_TIMESTAMP_UI_STAFF_PORTAL`
- `MAINTENANCE_IMPENDING_TIMESTAMP_UI_INVITED_ENTITY_PORTAL`
- `MAINTENANCE_START_TIMESTAMP_UI_INVITED_ENTITY_PORTAL`
- `MAINTENANCE_END_TIMESTAMP_UI_INVITED_ENTITY_PORTAL`

**DVI Files** (uppercase `Name` property):
- `MAINTENANCE_IMPENDING_TIMESTAMP_UI_PORTAL`
- `MAINTENANCE_START_TIMESTAMP_UI_PORTAL`
- `MAINTENANCE_END_TIMESTAMP_UI_PORTAL`

**EFM Files** (uppercase `Name` property):
- `MAINTENANCE_IMPENDING_TIMESTAMP_APPLICANTPORTAL`
- `MAINTENANCE_START_TIMESTAMP_APPLICANTPORTAL`
- `MAINTENANCE_END_TIMESTAMP_APPLICANTPORTAL`
- `MAINTENANCE_IMPENDING_TIMESTAMP_MIDUSPORTAL`
- `MAINTENANCE_START_TIMESTAMP_MIDUSPORTAL`
- `MAINTENANCE_END_TIMESTAMP_MIDUSPORTAL`
- `MAINTENANCE_IMPENDING_TIMESTAMP_ADMINPORTAL`
- `MAINTENANCE_START_TIMESTAMP_ADMINPORTAL`
- `MAINTENANCE_END_TIMESTAMP_ADMINPORTAL`

**ESP Files** (uppercase `Name` property):
- `MAINTENANCE_MSG_IMPENDING_TIMESTAMP_SECURE_REPORTS`
- `MAINTENANCE_MSG_START_TIMESTAMP_SECURE_REPORTS`
- `MAINTENANCE_MSG_END_TIMESTAMP_SECURE_REPORTS`

**PWY Files** (uppercase `Name` property):
- `MAINTENANCE_MSG_IMPENDING_TIMESTAMP_AUTHORITY`
- `MAINTENANCE_MSG_START_TIMESTAMP_AUTHORITY`
- `MAINTENANCE_MSG_END_TIMESTAMP_AUTHORITY`
- `MAINTENANCE_MSG_IMPENDING_TIMESTAMP_CASEMANAGEMENT`
- `MAINTENANCE_MSG_START_TIMESTAMP_CASEMANAGEMENT`
- `MAINTENANCE_MSG_END_TIMESTAMP_CASEMANAGEMENT`
- `MAINTENANCE_MSG_IMPENDING_TIMESTAMP_APPLICANT`
- `MAINTENANCE_MSG_START_TIMESTAMP_APPLICANT`
- `MAINTENANCE_MSG_END_TIMESTAMP_APPLICANT`

#### Step 5: Validation Checklist

After updating, verify:
- ✅ All 72 timestamp values updated across 10 files
- ✅ IMPENDING_TIMESTAMP = current UTC time
- ✅ START_TIMESTAMP = converted from EST/EDT to UTC
- ✅ END_TIMESTAMP = converted from EST/EDT to UTC
- ✅ All timestamps in ISO 8601 format: `YYYY-MM-DDTHH:mm:ss.000Z`
- ✅ JSON files remain valid (no syntax errors)
- ✅ **IMPENDING_TIMESTAMP < START_TIMESTAMP** (CRITICAL)
- ✅ **START_TIMESTAMP < END_TIMESTAMP**
- ✅ Chronological order maintained: IMPENDING < START < END

### JSON File Structure Notes

- **AHP files**: Use lowercase `"name"` property for feature flag names
- **DVI, EFM, ESP, PWY files**: Use uppercase `"Name"` property for feature flag names
- All files follow the same structure with `value` properties containing the timestamp strings
- Maintain consistent formatting and indentation when editing

### Common Scenarios

#### Scenario 1: Standard Maintenance Update
User provides future maintenance window → Update all 72 timestamps with current UTC time for IMPENDING and converted times for START/END.

#### Scenario 2: Late Update (Current Time Past START)
If current UTC time is already past the START_TIMESTAMP, set IMPENDING to 1 hour before START to maintain chronological validity.

#### Scenario 3: Daylight Saving Time Transitions
Always verify whether the maintenance date falls under EST or EDT before converting:
- March through early November (before first Sunday) → EDT (UTC-4)
- November (after first Sunday) through March (before second Sunday) → EST (UTC-5)

### Best Practices

1. **Always update all 10 files**: Both `-on` and `-off` variants must be synchronized
2. **Use multi_replace_string_in_file**: For efficiency when updating multiple timestamps
3. **Verify timezone**: Confirm EST vs EDT based on the maintenance date
4. **Validate chronology**: Always check IMPENDING < START < END before committing
5. **Preserve JSON structure**: Maintain existing formatting, indentation, and property names
6. **Double-check conversions**: Time zone math errors can cause incorrect maintenance windows

### Example Update

**Input**:
```
Friday – November 07, 2025, starting at 5:00PM EST through Saturday - November 08, 2025, 6:00PM EST
```

**Calculated Values**:
- Current UTC: `2025-11-06T18:00:00.000Z` (example: updated on Nov 6 at 6 PM UTC)
- IMPENDING: `2025-11-06T18:00:00.000Z`
- START: `2025-11-07T22:00:00.000Z` (Nov 7, 5 PM EST + 5 hours)
- END: `2025-11-08T23:00:00.000Z` (Nov 8, 6 PM EST + 5 hours)

**Validation**: ✅ `2025-11-06T18:00:00.000Z` < `2025-11-07T22:00:00.000Z` < `2025-11-08T23:00:00.000Z`

### Files to Update

Production files (10 total):
- `Prod/AHP_Feature_Flag_PRD-off.json`
- `Prod/AHP_Feature_Flag_PRD-on.json`
- `Prod/DVI_Feature_Flag_PRD-off.json`
- `Prod/DVI_Feature_Flag_PRD-on.json`
- `Prod/EFM_Feature_Flag_PRD-off.json`
- `Prod/EFM_Feature_Flag_PRD-on.json`
- `Prod/ESP_Feature_Flag_PRD-off.json`
- `Prod/ESP_Feature_Flag_PRD-on.json`
- `Prod/PWY_Feature_Flag_PRD-off.json`
- `Prod/PWY_Feature_Flag_PRD-on.json`

QA and Dev environments have similar structures but are updated less frequently.
