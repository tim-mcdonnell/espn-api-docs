# Core Relationship Tables

This section documents the relationship tables that connect our core dimension entities. These tables define how entities like teams, athletes, and groups relate to each other in our data model.

## TeamGroups

Maps teams to groups/conferences.

| Column | Type | Description | Example |
|--------|------|-------------|---------|
| team_group_id | INTEGER | Primary key (auto-increment) | 1 |
| team_id | INTEGER | Foreign key to teams | 1 |
| group_id | INTEGER | Foreign key to groups | 1 |

**Indexing:**
- Order by `group_id, team_id` to optimize grouping operations
- No explicit indexes needed beyond the primary key

## AthleteTeams

Links athletes to their teams by season.

| Column | Type | Description | Example |
|--------|------|-------------|---------|
| athlete_team_id | INTEGER | Primary key (auto-increment) | 1 |
| athlete_id | INTEGER | Foreign key to athletes | 1 |
| team_id | INTEGER | Foreign key to teams | 1 |
| season_id | INTEGER | Foreign key to seasons | 1 |
| active | BOOLEAN | If currently active | true |
| jersey | VARCHAR | Jersey number with team | 15 |

**Indexing:**
- Create composite index for team roster lookups:
  ```sql
  CREATE INDEX idx_athlete_teams_team ON AthleteTeams(team_id, season_id);
  ```
- Create index for athlete history lookups:
  ```sql
  CREATE INDEX idx_athlete_teams_athlete ON AthleteTeams(athlete_id, season_id);
  ``` 