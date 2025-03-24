# Coaching Staff Data Structure

This section documents the tables used for storing coaching staff data in our ESPN API data structure. These tables track coaching personnel, their roles, history, and relationships with teams.

## Coaches

Information about coaching personnel.

| Column | Type | Description | Example |
|--------|------|-------------|---------|
| coach_id | INTEGER | Primary key (auto-increment) | 1 |
| espn_id | VARCHAR | ESPN's identifier | 6427 |
| first_name | VARCHAR | First name | Jon |
| last_name | VARCHAR | Last name | Scheyer |
| full_name | VARCHAR | Full name | Jon Scheyer |
| display_name | VARCHAR | Display name | Jon Scheyer |
| short_name | VARCHAR | Short display name | J. Scheyer |
| birth_date | TIMESTAMP | Birth date | 1987-08-24T00:00Z |
| alma_mater | VARCHAR | College attended | Duke |
| alma_mater_id | INTEGER | Foreign key to franchises | 150 |
| biography | VARCHAR | Biographical information | Former Duke player who took over from Coach K |
| headshot_url | VARCHAR | URL to headshot image | https://a.espncdn.com/i/headshots/mens-college-basketball/players/full/6427.png |
| status | VARCHAR | Current status | active |
| birth_place | VARCHAR | Place of birth | Northbrook, IL |
| nationality | VARCHAR | Nationality | USA |
| height | INTEGER | Height in inches | 76 |
| display_height | VARCHAR | Formatted height | 6' 4" |
| start_year | INTEGER | First year coaching | 2014 |
| career_win_pct | DECIMAL | Career winning percentage | 0.774 |
| career_wins | INTEGER | Career wins | 24 |
| career_losses | INTEGER | Career losses | 7 |
| slug | VARCHAR | URL identifier | jon-scheyer |

**Indexing:**
- Create explicit index on `espn_id` for API integration:
  ```sql
  CREATE INDEX idx_coaches_espn_id ON Coaches(espn_id);
  ```
- Create index on `last_name, first_name` for name-based searches:
  ```sql
  CREATE INDEX idx_coaches_name ON Coaches(last_name, first_name);
  ```
- Order by `last_name, first_name` to optimize name-based lookups via zonemaps

## CoachRoles

Defines different coaching positions and roles.

| Column | Type | Description | Example |
|--------|------|-------------|---------|
| role_id | INTEGER | Primary key (auto-increment) | 1 |
| name | VARCHAR | Role name | Head Coach |
| display_name | VARCHAR | Display name | Head Coach |
| short_name | VARCHAR | Short name | HC |
| description | VARCHAR | Role description | Primary coach responsible for team operations |
| rank_order | INTEGER | Ordering/hierarchy | 1 |
| is_assistant | BOOLEAN | If assistant coach | false |
| is_staff | BOOLEAN | If part of staff (not coaches) | false |

**Indexing:**
- No explicit indexes needed due to small table size
- Order by `rank_order` to maintain hierarchy via zonemaps

## TeamCoaches

Links coaches to teams with their roles.

| Column | Type | Description | Example |
|--------|------|-------------|---------|
| team_coach_id | INTEGER | Primary key (auto-increment) | 1 |
| coach_id | INTEGER | Foreign key to coaches | 1 |
| team_id | INTEGER | Foreign key to teams | 1 |
| franchise_id | INTEGER | Foreign key to franchises | 2 |
| role_id | INTEGER | Foreign key to coach roles | 1 |
| season_id | INTEGER | Foreign key to seasons | 1 |
| is_interim | BOOLEAN | If serving in interim capacity | false |
| is_active | BOOLEAN | If currently active | true |
| start_date | TIMESTAMP | Start date with team | 2022-04-01T00:00Z |
| end_date | TIMESTAMP | End date with team (NULL if current) | NULL |
| order_num | INTEGER | Hierarchy order | 1 |
| salary | INTEGER | Annual salary (if available) | 9200000 |
| contract_start | TIMESTAMP | Contract start date | 2022-04-01T00:00Z |
| contract_end | TIMESTAMP | Contract end date | 2029-04-01T00:00Z |

**Indexing:**
- Order by `team_id, season_id, order_num` to optimize team coaching staff lookup
- Create index for coach history and tenure analysis:
  ```sql
  CREATE INDEX idx_team_coaches_coach ON TeamCoaches(coach_id, start_date);
  ```
- Create index for role-based queries:
  ```sql
  CREATE INDEX idx_team_coaches_role ON TeamCoaches(role_id, team_id, season_id);
  ```

## CoachingStaff

Non-coaching staff members associated with teams.

| Column | Type | Description | Example |
|--------|------|-------------|---------|
| staff_id | INTEGER | Primary key (auto-increment) | 1 |
| first_name | VARCHAR | First name | Alan |
| last_name | VARCHAR | Last name | Stein |
| full_name | VARCHAR | Full name | Alan Stein |
| display_name | VARCHAR | Display name | Alan Stein Jr. |
| role_id | INTEGER | Foreign key to coach roles | 4 |
| team_id | INTEGER | Foreign key to teams | 1 |
| franchise_id | INTEGER | Foreign key to franchises | 2 |
| season_id | INTEGER | Foreign key to seasons | 1 |
| is_active | BOOLEAN | If currently active | true |
| start_date | TIMESTAMP | Start date with team | 2023-06-15T00:00Z |
| end_date | TIMESTAMP | End date with team (NULL if current) | NULL |
| order_num | INTEGER | Hierarchy order | 1 |
| department | VARCHAR | Department name | Strength and Conditioning |

**Indexing:**
- Order by `team_id, season_id, order_num` for staff roster lookup
- Create index for department-based queries:
  ```sql
  CREATE INDEX idx_coaching_staff_department ON CoachingStaff(department, team_id, season_id);
  ```

## CoachCareerHistory

Historical record of coaching careers across teams and roles.

| Column | Type | Description | Example |
|--------|------|-------------|---------|
| history_id | INTEGER | Primary key (auto-increment) | 1 |
| coach_id | INTEGER | Foreign key to coaches | 1 |
| franchise_id | INTEGER | Foreign key to franchises | 2 |
| role_id | INTEGER | Foreign key to coach roles | 1 |
| start_year | INTEGER | Start year | 2014 |
| end_year | INTEGER | End year (NULL if current) | 2021 |
| win_count | INTEGER | Total wins | 115 |
| loss_count | INTEGER | Total losses | 35 |
| tie_count | INTEGER | Total ties | 0 |
| win_pct | DECIMAL | Winning percentage | 0.767 |
| championships | INTEGER | Championships won | 0 |
| post_season_appearances | INTEGER | Post-season appearances | 4 |
| order_num | INTEGER | Chronological order of tenure | 1 |
| notes | VARCHAR | Additional information | Served as associate head coach under Mike Krzyzewski |

**Indexing:**
- Order by `coach_id, start_year, end_year` to optimize career timeline queries
- Create index for franchise coaching history:
  ```sql
  CREATE INDEX idx_coach_career_history_franchise ON CoachCareerHistory(franchise_id, end_year);
  ```

## CoachSeasonStatistics

Season-level statistics for coaches.

| Column | Type | Description | Example |
|--------|------|-------------|---------|
| coach_stat_id | INTEGER | Primary key (auto-increment) | 1 |
| coach_id | INTEGER | Foreign key to coaches | 1 |
| team_id | INTEGER | Foreign key to teams | 1 |
| season_id | INTEGER | Foreign key to seasons | 1 |
| wins | INTEGER | Season wins | 24 |
| losses | INTEGER | Season losses | 7 |
| win_pct | DECIMAL | Season winning percentage | 0.774 |
| conference_wins | INTEGER | Conference wins | 15 |
| conference_losses | INTEGER | Conference losses | 3 |
| home_wins | INTEGER | Home wins | 15 |
| home_losses | INTEGER | Home losses | 1 |
| away_wins | INTEGER | Away wins | 7 |
| away_losses | INTEGER | Away losses | 5 |
| neutral_wins | INTEGER | Neutral site wins | 2 |
| neutral_losses | INTEGER | Neutral site losses | 1 |
| post_season_wins | INTEGER | Post-season wins | 0 |
| post_season_losses | INTEGER | Post-season losses | 1 |
| ranked_wins | INTEGER | Wins vs. ranked teams | 4 |
| ranked_losses | INTEGER | Losses vs. ranked teams | 3 |
| tournament_appearances | INTEGER | Tournament appearances | 1 |
| final_rank | INTEGER | Final ranking | 16 |
| group_finish | INTEGER | Conference finish position | 2 |

**Indexing:**
- Order by `coach_id, season_id` to optimize coach performance lookup by season
- Create index for comparative analysis:
  ```sql
  CREATE INDEX idx_coach_season_statistics_wins ON CoachSeasonStatistics(season_id, wins DESC);
  ```

## CoachingTrees

Tracks coaching mentorship relationships.

| Column | Type | Description | Example |
|--------|------|-------------|---------|
| tree_id | INTEGER | Primary key (auto-increment) | 1 |
| name | VARCHAR | Tree name | Coach K Tree |
| root_coach_id | INTEGER | Foreign key to coaches (tree founder) | 25 |
| description | VARCHAR | Tree description | Coaches who worked under Mike Krzyzewski |
| start_year | INTEGER | Tree start year | 1980 |
| branch_count | INTEGER | Number of branches | 47 |
| active_coaches | INTEGER | Currently active coaches in tree | 23 |

**Indexing:**
- No explicit indexes needed due to small table size

## CoachingTreeMembers

Links coaches to their coaching trees.

| Column | Type | Description | Example |
|--------|------|-------------|---------|
| tree_member_id | INTEGER | Primary key (auto-increment) | 1 |
| tree_id | INTEGER | Foreign key to coaching trees | 1 |
| coach_id | INTEGER | Foreign key to coaches | 1 |
| mentor_coach_id | INTEGER | Foreign key to coaches (mentor) | 25 |
| generation | INTEGER | Tree generation (1=direct disciple) | 1 |
| years_under_mentor | INTEGER | Years coached under mentor | 8 |
| relationship_start | INTEGER | Start year of relationship | 2014 |
| relationship_end | INTEGER | End year of relationship | 2022 |
| relationship_type | VARCHAR | Type of relationship | Assistant Coach |
| notes | VARCHAR | Additional information | Played under Coach K from 2006-2010 |

**Indexing:**
- Order by `tree_id, generation, coach_id` to optimize tree hierarchy queries
- Create index for mentor relationships:
  ```sql
  CREATE INDEX idx_coaching_tree_members_mentor ON CoachingTreeMembers(mentor_coach_id, coach_id);
  ``` 