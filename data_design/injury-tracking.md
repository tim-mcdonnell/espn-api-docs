# Injury Tracking System

This document outlines the data structure for tracking and analyzing injuries within our ESPN API data management system. The injury tracking system captures detailed information about athlete injuries, their severity, recovery timelines, and historical patterns.

## Purpose

The injury tracking system serves to:

- Record comprehensive injury data for athletes across all sports
- Track the progression of injuries and recovery timelines
- Support analysis of injury patterns and team/athlete injury histories
- Provide context for performance analysis and lineup changes

## Core Entity Relationships

```
[InjuryTypes] ←→ [Injuries] ←→ [AthleteInjuries]
      ↑                ↑               ↑
      │                │               │
      ↓                ↓               ↓
[BodyParts] ←→ [InjurySeverity] ←→ [Athletes]
                      ↑                ↑ 
                      │                │
                      ↓                ↓
[RecoveryProtocols] ←→ [InjuryUpdates] ←→ [TeamMedicalStaff]
```

## Table Definitions

### InjuryTypes

Defines categories and classifications of injuries.

| Column | Type | Description | Example |
|--------|------|-------------|---------|
| injury_type_id | INTEGER | Primary key | 1 |
| espn_id | VARCHAR | ESPN's identifier for this injury type | 12 |
| uid | VARCHAR | Universal identifier | s:40~l:41~it:12 |
| name | VARCHAR | Specific injury type name (e.g., "ACL Tear", "Concussion") | ACL Tear |
| category | VARCHAR | General category (e.g., "Ligament", "Head") | Ligament |
| description | VARCHAR | Detailed description of the injury type | Complete tear of the anterior cruciate ligament |
| typical_recovery_time_min | INTEGER | Minimum typical recovery time (days) | 180 |
| typical_recovery_time_max | INTEGER | Maximum typical recovery time (days) | 365 |
| is_surgical | BOOLEAN | Whether injury typically requires surgery | true |
| created_at | TIMESTAMP | Record creation timestamp | 2024-03-01T12:00:00Z |
| updated_at | TIMESTAMP | Record update timestamp | 2024-03-15T09:30:00Z |

### BodyParts

Catalogs body parts/locations for injury classification.

| Column | Type | Description | Example |
|--------|------|-------------|---------|
| body_part_id | INTEGER | Primary key | 1 |
| espn_id | VARCHAR | ESPN's identifier for this body part | 4 |
| uid | VARCHAR | Universal identifier | s:40~l:41~bp:4 |
| name | VARCHAR | Name of the body part (e.g., "Knee", "Ankle") | Knee |
| region | VARCHAR | Body region (e.g., "Lower Body", "Upper Body") | Lower Body |
| laterality | VARCHAR | Side indicator (e.g., "Left", "Right", "Bilateral", "N/A") | Bilateral |
| created_at | TIMESTAMP | Record creation timestamp | 2024-03-01T12:00:00Z |
| updated_at | TIMESTAMP | Record update timestamp | 2024-03-15T09:30:00Z |

### InjurySeverity

Standardized severity classifications for injuries.

| Column | Type | Description | Example |
|--------|------|-------------|---------|
| severity_id | INTEGER | Primary key | 1 |
| espn_id | VARCHAR | ESPN's identifier for this severity level | 3 |
| uid | VARCHAR | Universal identifier | s:40~l:41~is:3 |
| name | VARCHAR | Severity level name (e.g., "Minor", "Moderate", "Severe") | Severe |
| description | VARCHAR | Detailed description of this severity level | Requires surgical intervention and extended recovery |
| estimated_games_missed_min | INTEGER | Minimum games typically missed | 20 |
| estimated_games_missed_max | INTEGER | Maximum games typically missed | 82 |
| created_at | TIMESTAMP | Record creation timestamp | 2024-03-01T12:00:00Z |
| updated_at | TIMESTAMP | Record update timestamp | 2024-03-15T09:30:00Z |

### Injuries

Core table defining specific injury instances.

| Column | Type | Description | Example |
|--------|------|-------------|---------|
| injury_id | INTEGER | Primary key | 1 |
| injury_type_id | INTEGER | Foreign key to InjuryTypes | 1 |
| body_part_id | INTEGER | Foreign key to BodyParts | 1 |
| severity_id | INTEGER | Foreign key to InjurySeverity | 1 |
| espn_id | VARCHAR | ESPN's identifier for this injury | 87652 |
| uid | VARCHAR | Universal identifier | s:40~l:41~i:87652 |
| description | VARCHAR | Detailed description of this specific injury | Complete ACL tear with meniscus damage |
| is_chronic | BOOLEAN | Whether this is a chronic/recurring condition | false |
| created_at | TIMESTAMP | Record creation timestamp | 2024-03-01T12:00:00Z |
| updated_at | TIMESTAMP | Record update timestamp | 2024-03-15T09:30:00Z |

### AthleteInjuries

Links athletes to specific injuries and tracks their status.

| Column | Type | Description | Example |
|--------|------|-------------|---------|
| athlete_injury_id | INTEGER | Primary key | 1 |
| athlete_id | INTEGER | Foreign key to Athletes | 4433137 |
| injury_id | INTEGER | Foreign key to Injuries | 1 |
| team_id | INTEGER | Foreign key to Teams (team at time of injury) | 2509 |
| event_id | INTEGER | Foreign key to Events (null if not game-related) | 401524661 |
| injury_date | TIMESTAMP | When the injury occurred | 2024-01-15T19:42:30Z |
| discovery_date | TIMESTAMP | When the injury was discovered/diagnosed | 2024-01-16T14:00:00Z |
| initial_status | VARCHAR | Initial game status (e.g., "Questionable", "Out") | Out |
| current_status | VARCHAR | Current game status | Out |
| status_last_updated | TIMESTAMP | When status was last changed | 2024-01-16T14:00:00Z |
| projected_return_date | TIMESTAMP | Estimated return date | 2024-08-15T00:00:00Z |
| actual_return_date | TIMESTAMP | Actual return date (null if not yet returned) | null |
| is_season_ending | BOOLEAN | Whether injury ended player's season | true |
| is_career_ending | BOOLEAN | Whether injury ended player's career | false |
| surgery_date | TIMESTAMP | Date of surgery (null if no surgery) | 2024-01-25T08:30:00Z |
| placed_on_ir_date | TIMESTAMP | Date placed on injured reserve (null if not applicable) | 2024-01-17T16:00:00Z |
| reinjury_of | INTEGER | Self-reference to previous injury instance (null if new) | null |
| created_at | TIMESTAMP | Record creation timestamp | 2024-03-01T12:00:00Z |
| updated_at | TIMESTAMP | Record update timestamp | 2024-03-15T09:30:00Z |

### InjuryUpdates

Tracks the progression and updates for athlete injuries.

| Column | Type | Description | Example |
|--------|------|-------------|---------|
| update_id | INTEGER | Primary key | 1 |
| athlete_injury_id | INTEGER | Foreign key to AthleteInjuries | 1 |
| update_date | TIMESTAMP | When this update was reported | 2024-02-01T15:30:00Z |
| status | VARCHAR | Status at time of update | Out |
| description | VARCHAR | Details of the update | Surgery successful, beginning initial rehab phase |
| source | VARCHAR | Source of the update information | Team PR |
| return_timeline_min | INTEGER | Updated minimum days to return | 180 |
| return_timeline_max | INTEGER | Updated maximum days to return | 220 |
| setback_reported | BOOLEAN | Whether this update reports a setback | false |
| created_at | TIMESTAMP | Record creation timestamp | 2024-03-01T12:00:00Z |
| updated_at | TIMESTAMP | Record update timestamp | 2024-03-15T09:30:00Z |

### RecoveryProtocols

Documents recovery approaches for injuries.

| Column | Type | Description | Example |
|--------|------|-------------|---------|
| protocol_id | INTEGER | Primary key | 1 |
| injury_type_id | INTEGER | Foreign key to InjuryTypes | 1 |
| espn_id | VARCHAR | ESPN's identifier for this protocol | 53 |
| uid | VARCHAR | Universal identifier | s:40~l:41~rp:53 |
| name | VARCHAR | Name of the protocol | Standard ACL Reconstruction Protocol |
| description | VARCHAR | Detailed description | Progressive rehabilitation protocol following ACL reconstruction |
| typical_duration | INTEGER | Typical duration in days | 270 |
| is_surgical | BOOLEAN | Whether this protocol involves surgery | true |
| rehab_phases | INTEGER | Number of rehabilitation phases | 4 |
| created_at | TIMESTAMP | Record creation timestamp | 2024-03-01T12:00:00Z |
| updated_at | TIMESTAMP | Record update timestamp | 2024-03-15T09:30:00Z |

### TeamMedicalStaff

Tracks medical personnel associated with teams.

| Column | Type | Description | Example |
|--------|------|-------------|---------|
| staff_id | INTEGER | Primary key | 1 |
| team_id | INTEGER | Foreign key to Teams | 2509 |
| espn_id | VARCHAR | ESPN's identifier for this staff member | 782 |
| uid | VARCHAR | Universal identifier | s:40~l:41~t:2509~ms:782 |
| name | VARCHAR | Staff member's name | Dr. Sarah Johnson |
| role | VARCHAR | Role (e.g., "Head Trainer", "Team Physician") | Team Physician |
| start_date | TIMESTAMP | When they started with the team | 2020-06-01T00:00:00Z |
| end_date | TIMESTAMP | When they left the team (null if current) | null |
| created_at | TIMESTAMP | Record creation timestamp | 2024-03-01T12:00:00Z |
| updated_at | TIMESTAMP | Record update timestamp | 2024-03-15T09:30:00Z |

## Implementation Guidelines

The injury tracking system should follow these implementation guidelines:

1. **Data Sourcing**:
   - Primary injury data will be collected from ESPN API's injury reports
   - Supplemental data will be gathered from team announcements and ESPN news articles
   - Historical data will require backfilling from multiple sources

2. **Data Loading Frequency**:
   - Injury status should be updated daily during active seasons
   - Complete refresh of injury data before each game day
   - Historical updates may be made as new information becomes available

3. **Data Quality Considerations**:
   - Injury descriptions may vary in specificity across different sports and leagues
   - Return dates are often estimates and require regular updates
   - Reinjuries must be properly linked to original injuries

4. **Analytical Use Cases**:
   - Tracking injury impact on team performance
   - Analyzing injury patterns by position, team, or playing surface
   - Evaluating the accuracy of projected return dates
   - Identifying injury trends across seasons or careers

## Example Queries

**Athletes currently injured on a specific team:**
```sql
SELECT a.full_name, i.description, ai.current_status, ai.projected_return_date
FROM AthleteInjuries ai
JOIN Athletes a ON ai.athlete_id = a.athlete_id
JOIN Injuries i ON ai.injury_id = i.injury_id
WHERE ai.team_id = ? AND ai.actual_return_date IS NULL
ORDER BY ai.projected_return_date ASC;
```

**Injury history for a specific athlete:**
```sql
SELECT i.description, bp.name AS body_part, bp.laterality,
       ai.injury_date, ai.actual_return_date,
       DATEDIFF('day', ai.injury_date, COALESCE(ai.actual_return_date, CURRENT_DATE)) AS days_out
FROM AthleteInjuries ai
JOIN Injuries i ON ai.injury_id = i.injury_id
JOIN BodyParts bp ON i.body_part_id = bp.body_part_id
WHERE ai.athlete_id = ?
ORDER BY ai.injury_date DESC;
```

**Teams most affected by injuries in a season:**
```sql
SELECT t.display_name AS team, COUNT(ai.athlete_injury_id) AS injury_count,
       SUM(DATEDIFF('day', ai.injury_date, COALESCE(ai.actual_return_date, CURRENT_DATE))) AS total_days_missed
FROM AthleteInjuries ai
JOIN Teams t ON ai.team_id = t.team_id
WHERE ai.injury_date BETWEEN ? AND ? -- Season start and end dates
GROUP BY t.team_id, t.display_name
ORDER BY total_days_missed DESC;
``` 