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
[INJURY_TYPES] ←→ [INJURIES] ←→ [ATHLETE_INJURIES]
      ↑                ↑               ↑
      │                │               │
      ↓                ↓               ↓
[BODY_PARTS] ←→ [INJURY_SEVERITY] ←→ [ATHLETES]
                      ↑                ↑ 
                      │                │
                      ↓                ↓
[RECOVERY_PROTOCOLS] ←→ [INJURY_UPDATES] ←→ [TEAM_MEDICAL_STAFF]
```

## Table Definitions

### INJURY_TYPES

Defines categories and classifications of injuries.

| Column | Data Type | Description |
|--------|-----------|-------------|
| injury_type_id | INTEGER | Primary key |
| name | VARCHAR | Specific injury type name (e.g., "ACL Tear", "Concussion") |
| category | VARCHAR | General category (e.g., "Ligament", "Head") |
| description | VARCHAR | Detailed description of the injury type |
| typical_recovery_time_min | INTEGER | Minimum typical recovery time (days) |
| typical_recovery_time_max | INTEGER | Maximum typical recovery time (days) |
| is_surgical | BOOLEAN | Whether injury typically requires surgery |
| created_at | TIMESTAMP | Record creation timestamp |
| updated_at | TIMESTAMP | Record update timestamp |

### BODY_PARTS

Catalogs body parts/locations for injury classification.

| Column | Data Type | Description |
|--------|-----------|-------------|
| body_part_id | INTEGER | Primary key |
| name | VARCHAR | Name of the body part (e.g., "Knee", "Ankle") |
| region | VARCHAR | Body region (e.g., "Lower Body", "Upper Body") |
| laterality | VARCHAR | Side indicator (e.g., "Left", "Right", "Bilateral", "N/A") |
| created_at | TIMESTAMP | Record creation timestamp |
| updated_at | TIMESTAMP | Record update timestamp |

### INJURY_SEVERITY

Standardized severity classifications for injuries.

| Column | Data Type | Description |
|--------|-----------|-------------|
| severity_id | INTEGER | Primary key |
| name | VARCHAR | Severity level name (e.g., "Minor", "Moderate", "Severe") |
| description | VARCHAR | Detailed description of this severity level |
| estimated_games_missed_min | INTEGER | Minimum games typically missed |
| estimated_games_missed_max | INTEGER | Maximum games typically missed |
| created_at | TIMESTAMP | Record creation timestamp |
| updated_at | TIMESTAMP | Record update timestamp |

### INJURIES

Core table defining specific injury instances.

| Column | Data Type | Description |
|--------|-----------|-------------|
| injury_id | INTEGER | Primary key |
| injury_type_id | INTEGER | Foreign key to INJURY_TYPES |
| body_part_id | INTEGER | Foreign key to BODY_PARTS |
| severity_id | INTEGER | Foreign key to INJURY_SEVERITY |
| espn_injury_id | VARCHAR | ESPN's identifier for this injury |
| description | VARCHAR | Detailed description of this specific injury |
| is_chronic | BOOLEAN | Whether this is a chronic/recurring condition |
| created_at | TIMESTAMP | Record creation timestamp |
| updated_at | TIMESTAMP | Record update timestamp |

### ATHLETE_INJURIES

Links athletes to specific injuries and tracks their status.

| Column | Data Type | Description |
|--------|-----------|-------------|
| athlete_injury_id | INTEGER | Primary key |
| athlete_id | INTEGER | Foreign key to ATHLETES |
| injury_id | INTEGER | Foreign key to INJURIES |
| team_id | INTEGER | Foreign key to TEAMS (team at time of injury) |
| event_id | INTEGER | Foreign key to EVENTS (null if not game-related) |
| injury_date | TIMESTAMP | When the injury occurred |
| discovery_date | TIMESTAMP | When the injury was discovered/diagnosed |
| initial_status | VARCHAR | Initial game status (e.g., "Questionable", "Out") |
| current_status | VARCHAR | Current game status |
| status_last_updated | TIMESTAMP | When status was last changed |
| projected_return_date | TIMESTAMP | Estimated return date |
| actual_return_date | TIMESTAMP | Actual return date (null if not yet returned) |
| is_season_ending | BOOLEAN | Whether injury ended player's season |
| is_career_ending | BOOLEAN | Whether injury ended player's career |
| surgery_date | TIMESTAMP | Date of surgery (null if no surgery) |
| placed_on_ir_date | TIMESTAMP | Date placed on injured reserve (null if not applicable) |
| reinjury_of | INTEGER | Self-reference to previous injury instance (null if new) |
| created_at | TIMESTAMP | Record creation timestamp |
| updated_at | TIMESTAMP | Record update timestamp |

### INJURY_UPDATES

Tracks the progression and updates for athlete injuries.

| Column | Data Type | Description |
|--------|-----------|-------------|
| update_id | INTEGER | Primary key |
| athlete_injury_id | INTEGER | Foreign key to ATHLETE_INJURIES |
| update_date | TIMESTAMP | When this update was reported |
| status | VARCHAR | Status at time of update |
| description | VARCHAR | Details of the update |
| source | VARCHAR | Source of the update information |
| return_timeline_min | INTEGER | Updated minimum days to return |
| return_timeline_max | INTEGER | Updated maximum days to return |
| setback_reported | BOOLEAN | Whether this update reports a setback |
| created_at | TIMESTAMP | Record creation timestamp |
| updated_at | TIMESTAMP | Record update timestamp |

### RECOVERY_PROTOCOLS

Documents recovery approaches for injuries.

| Column | Data Type | Description |
|--------|-----------|-------------|
| protocol_id | INTEGER | Primary key |
| injury_type_id | INTEGER | Foreign key to INJURY_TYPES |
| name | VARCHAR | Name of the protocol |
| description | VARCHAR | Detailed description |
| typical_duration | INTEGER | Typical duration in days |
| is_surgical | BOOLEAN | Whether this protocol involves surgery |
| rehab_phases | INTEGER | Number of rehabilitation phases |
| created_at | TIMESTAMP | Record creation timestamp |
| updated_at | TIMESTAMP | Record update timestamp |

### TEAM_MEDICAL_STAFF

Tracks medical personnel associated with teams.

| Column | Data Type | Description |
|--------|-----------|-------------|
| staff_id | INTEGER | Primary key |
| team_id | INTEGER | Foreign key to TEAMS |
| name | VARCHAR | Staff member's name |
| role | VARCHAR | Role (e.g., "Head Trainer", "Team Physician") |
| start_date | TIMESTAMP | When they started with the team |
| end_date | TIMESTAMP | When they left the team (null if current) |
| created_at | TIMESTAMP | Record creation timestamp |
| updated_at | TIMESTAMP | Record update timestamp |

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
FROM ATHLETE_INJURIES ai
JOIN ATHLETES a ON ai.athlete_id = a.athlete_id
JOIN INJURIES i ON ai.injury_id = i.injury_id
WHERE ai.team_id = ? AND ai.actual_return_date IS NULL
ORDER BY ai.projected_return_date ASC;
```

**Injury history for a specific athlete:**
```sql
SELECT i.description, bp.name AS body_part, bp.laterality,
       ai.injury_date, ai.actual_return_date,
       DATEDIFF('day', ai.injury_date, COALESCE(ai.actual_return_date, CURRENT_DATE)) AS days_out
FROM ATHLETE_INJURIES ai
JOIN INJURIES i ON ai.injury_id = i.injury_id
JOIN BODY_PARTS bp ON i.body_part_id = bp.body_part_id
WHERE ai.athlete_id = ?
ORDER BY ai.injury_date DESC;
```

**Teams most affected by injuries in a season:**
```sql
SELECT t.display_name AS team, COUNT(ai.athlete_injury_id) AS injury_count,
       SUM(DATEDIFF('day', ai.injury_date, COALESCE(ai.actual_return_date, CURRENT_DATE))) AS total_days_missed
FROM ATHLETE_INJURIES ai
JOIN TEAMS t ON ai.team_id = t.team_id
WHERE ai.injury_date BETWEEN ? AND ? -- Season start and end dates
GROUP BY t.team_id, t.display_name
ORDER BY total_days_missed DESC;
``` 