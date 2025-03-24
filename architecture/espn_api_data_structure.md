# ESPN API Data Normalization Structure

## Overview

This document outlines a normalized database structure design for storing data from ESPN's NCAA Men's Basketball API. The purpose is to create an efficient, normalized structure that maps all API endpoint data into well-organized database tables for analytics purposes.

We're using DuckDB as the database engine for this project, which provides SQL capabilities with an analytics focus.

## Database Structure

Our normalized structure will use the following principles:
- Separate dimension and fact tables
- Consistent use of surrogate keys
- Proper foreign key relationships
- Appropriate handling of historical data
- Efficient storage of nested/hierarchical data

## Indexing Strategy

DuckDB primarily uses two indexing mechanisms:

1. **Zonemaps (min-max indexes)**
   - Created automatically for all general-purpose data types
   - Effectiveness heavily depends on data ordering
   - Critical for filtering operations

2. **ART Indexes**
   - Must be explicitly created
   - Useful for highly selective queries
   - Consume memory that isn't automatically managed

Our indexing strategy focuses on:
- **Ordering data appropriately** to maximize zonemap effectiveness
- Creating **explicit indexes** only where necessary for high-value queries
- Managing **memory consumption** through selective indexing
- Leveraging **materialized views** for frequent query patterns

## Entity Relationship Overview

```
[LEAGUES] ←→ [SEASONS] ←→ [SEASON_TYPES] ←→ [WEEKS]
     ↑          ↑              ↑
     │          │              │
     ↓          ↓              ↓
[TEAMS] ←→ [TEAM_SEASONS] ←→ [GAMES/EVENTS] ←→ [GAME_STATISTICS]
     ↑                           ↑               ↑
     │                           │               │
     ↓                           ↓               ↓
[ATHLETES] ←→ [ATHLETE_TEAMS] ←→ [GAME_ATHLETES] ←→ [ATHLETE_GAME_STATISTICS]
     ↑                                             ↑
     │                                             │
     ↓                                             ↓
[ATHLETE_STATISTICS] ←→ [STAT_CATEGORIES] ←→ [STAT_TYPES]
```

## Core Dimension Tables

### Leagues
Stores the basic league information.

| Column | Type | Description | Example |
|--------|------|-------------|---------|
| league_id | INTEGER | Primary key | 41 |
| uid | VARCHAR | Universal identifier | s:40~l:41 |
| name | VARCHAR | League name | NCAA Men's Basketball |
| abbreviation | VARCHAR | League abbreviation | NCAAM |
| short_name | VARCHAR | Short name | NCAAM |
| slug | VARCHAR | URL identifier | mens-college-basketball |

**Indexing:** No explicit indexes needed as this is a small table with an integer primary key.

### Seasons
Stores information about seasons.

| Column | Type | Description | Example |
|--------|------|-------------|---------|
| season_id | INTEGER | Primary key (auto-increment) | 1 |
| year | INTEGER | Season year | 2025 |
| display_name | VARCHAR | Display name | 2024-25 |
| start_date | TIMESTAMP | Season start date | 2024-07-13T07:00Z |
| end_date | TIMESTAMP | Season end date | 2025-04-09T06:59Z |
| league_id | INTEGER | Foreign key to leagues | 41 |

**Indexing:** 
- Order by `year` to optimize zonemap effectiveness for year-based filters
- No explicit indexes needed due to small table size and integer primary key

### SeasonTypes
Stores types of seasons (preseason, regular season, etc.).

| Column | Type | Description | Example |
|--------|------|-------------|---------|
| season_type_id | INTEGER | Primary key (auto-increment) | 1 |
| espn_id | INTEGER | ESPN's ID | 2 |
| season_id | INTEGER | Foreign key to seasons | 1 |
| name | VARCHAR | Type name | Regular Season |
| abbreviation | VARCHAR | Short abbreviation | reg |
| start_date | TIMESTAMP | Type start date | 2024-11-04T08:00Z |
| end_date | TIMESTAMP | Type end date | 2025-03-18T06:59Z |
| has_groups | BOOLEAN | If has group info | true |
| has_standings | BOOLEAN | If has standings | true |
| slug | VARCHAR | URL identifier | regular-season |

**Indexing:** No explicit indexes needed as this is a small table with few distinct values.

### Weeks
Weekly divisions within a season type.

| Column | Type | Description | Example |
|--------|------|-------------|---------|
| week_id | INTEGER | Primary key (auto-increment) | 1 |
| season_type_id | INTEGER | Foreign key to season types | 1 |
| number | INTEGER | Week number | 1 |
| text | VARCHAR | Display text | Week 1 |
| start_date | TIMESTAMP | Week start date | 2024-11-04T08:00Z |
| end_date | TIMESTAMP | Week end date | 2024-11-11T07:59Z |

**Indexing:**
- Order by `season_type_id, start_date` to improve filtering by date ranges within a season type
- No explicit indexes needed due to small cardinality

### Franchises (Programs)
Long-term athletic programs.

| Column | Type | Description | Example |
|--------|------|-------------|---------|
| franchise_id | INTEGER | Primary key (auto-increment) | 1 |
| espn_id | VARCHAR | ESPN's ID | 153 |
| uid | VARCHAR | Universal identifier | s:40~l:41~f:153 |
| location | VARCHAR | Geographic location | North Carolina |
| name | VARCHAR | Team name | Tar Heels |
| abbreviation | VARCHAR | Short abbreviation | UNC |
| display_name | VARCHAR | Full display name | North Carolina Tar Heels |
| short_display_name | VARCHAR | Short display name | UNC |
| color | VARCHAR | Primary color | 7BAFD4 |
| alternate_color | VARCHAR | Alternate color | 13294B |
| is_active | BOOLEAN | If franchise is active | true |
| slug | VARCHAR | URL identifier | north-carolina-tar-heels |

**Indexing:**
- Create explicit index on `espn_id` for API integration queries: 
  ```sql
  CREATE INDEX idx_franchises_espn_id ON Franchises(espn_id);
  ```
- Consider index on `slug` if URL-based lookups are frequent

### Teams
Season-specific team instances.

| Column | Type | Description | Example |
|--------|------|-------------|---------|
| team_id | INTEGER | Primary key (auto-increment) | 1 |
| espn_id | VARCHAR | ESPN's ID | 52 |
| uid | VARCHAR | Universal identifier | s:40~l:41~t:52 |
| season_id | INTEGER | Foreign key to seasons | 1 |
| franchise_id | INTEGER | Foreign key to franchises | 1 |
| is_active | BOOLEAN | If team is active | true |
| is_all_star | BOOLEAN | If all-star team | false |

**Indexing:**
- Create explicit index on `espn_id` for API queries:
  ```sql
  CREATE INDEX idx_teams_espn_id ON Teams(espn_id);
  ```
- Order table by `season_id, franchise_id` to optimize filtering by both dimensions

### Groups (Conferences/Divisions)
Conference and division structures.

| Column | Type | Description | Example |
|--------|------|-------------|---------|
| group_id | INTEGER | Primary key (auto-increment) | 1 |
| espn_id | VARCHAR | ESPN's ID | 1 |
| season_id | INTEGER | Foreign key to seasons | 1 |
| season_type_id | INTEGER | Foreign key to season types | 1 |
| name | VARCHAR | Group name | Atlantic Coast |
| abbreviation | VARCHAR | Abbreviation | ACC |
| short_name | VARCHAR | Short name | ACC |
| is_conference | BOOLEAN | If it's a conference | true |
| parent_group_id | INTEGER | Self-reference to parent group | NULL |
| slug | VARCHAR | URL identifier | atlantic-coast |

**Indexing:**
- Order by `season_id, season_type_id` to optimize filtering on these frequently queried dimensions
- No explicit indexes needed due to moderate size

### Venues
Game locations.

| Column | Type | Description | Example |
|--------|------|-------------|---------|
| venue_id | INTEGER | Primary key (auto-increment) | 1 |
| espn_id | VARCHAR | ESPN's ID | 4103 |
| name | VARCHAR | Venue name | Mackey Arena |
| capacity | INTEGER | Seating capacity | 14804 |
| indoor | BOOLEAN | If indoor venue | true |
| city | VARCHAR | City | West Lafayette |
| state | VARCHAR | State | IN |
| country | VARCHAR | Country | USA |

**Indexing:**
- Create index on `espn_id` for API integration:
  ```sql
  CREATE INDEX idx_venues_espn_id ON Venues(espn_id);
  ```
- Consider index on `state, city` if geographic filtering is common

### Athletes
Player information.

| Column | Type | Description | Example |
|--------|------|-------------|---------|
| athlete_id | INTEGER | Primary key (auto-increment) | 1 |
| espn_id | VARCHAR | ESPN's ID | 4433137 |
| uid | VARCHAR | Universal identifier | s:40~l:41~a:4433137 |
| guid | VARCHAR | Global unique ID | 9a0fb84c-85b5-47b2-b877-d44ff3a5c93f |
| first_name | VARCHAR | First name | Zach |
| last_name | VARCHAR | Last name | Edey |
| full_name | VARCHAR | Full name | Zach Edey |
| display_name | VARCHAR | Display name | Zach Edey |
| short_name | VARCHAR | Short name | Z. Edey |
| weight | INTEGER | Weight in pounds | 305 |
| height | INTEGER | Height in inches | 87 |
| display_height | VARCHAR | Formatted height | 7' 3" |
| date_of_birth | TIMESTAMP | Birth date | 2002-05-14T04:00Z |
| position_id | INTEGER | Foreign key to positions | 1 |
| jersey | VARCHAR | Jersey number | 15 |
| slug | VARCHAR | URL identifier | zach-edey |

**Indexing:**
- Create explicit indexes for common search patterns:
  ```sql
  CREATE INDEX idx_athletes_espn_id ON Athletes(espn_id);
  CREATE INDEX idx_athletes_name ON Athletes(last_name, first_name);
  ```
- Order by `last_name, first_name` to improve name-based searches through zonemaps

### Positions
Player positions.

| Column | Type | Description | Example |
|--------|------|-------------|---------|
| position_id | INTEGER | Primary key (auto-increment) | 1 |
| espn_id | VARCHAR | ESPN's ID | 1 |
| name | VARCHAR | Position name | Center |
| display_name | VARCHAR | Display name | Center |
| abbreviation | VARCHAR | Abbreviation | C |
| parent_position_id | INTEGER | Self-reference to parent position | NULL |

**Indexing:** No explicit indexes needed due to small table size.

## Core Relationship Tables

### TeamGroups
Maps teams to groups/conferences.

| Column | Type | Description | Example |
|--------|------|-------------|---------|
| team_group_id | INTEGER | Primary key (auto-increment) | 1 |
| team_id | INTEGER | Foreign key to teams | 1 |
| group_id | INTEGER | Foreign key to groups | 1 |

**Indexing:**
- Order by `group_id, team_id` to optimize grouping operations
- No explicit indexes needed beyond the primary key

### AthleteTeams
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

## Core Fact Tables

### Events (Games)
Game events data.

| Column | Type | Description | Example |
|--------|------|-------------|---------|
| event_id | INTEGER | Primary key (auto-increment) | 1 |
| espn_id | VARCHAR | ESPN's ID | 401524661 |
| uid | VARCHAR | Universal identifier | s:40~l:41~e:401524661 |
| season_id | INTEGER | Foreign key to seasons | 1 |
| season_type_id | INTEGER | Foreign key to season types | 1 |
| week_id | INTEGER | Foreign key to weeks | 1 |
| venue_id | INTEGER | Foreign key to venues | 1 |
| name | VARCHAR | Event name | Nebraska at Purdue |
| short_name | VARCHAR | Shortened name | NEB @ PUR |
| start_date | TIMESTAMP | Start date and time | 2024-01-10T00:00Z |
| status | VARCHAR | Game status | STATUS_FINAL |
| period | INTEGER | Current period | 2 |
| clock | VARCHAR | Game clock | 0:00 |
| attendance | INTEGER | Number of attendees | 14876 |
| neutral_site | BOOLEAN | If played at neutral site | false |

**Indexing:**
- Order by `start_date` to maximize zonemap effectiveness for date-range queries
- Create explicit indexes for common query patterns:
  ```sql
  CREATE INDEX idx_events_espn_id ON Events(espn_id);
  CREATE INDEX idx_events_season ON Events(season_id, season_type_id, week_id);
  ```

### TeamEvents
Links teams to events with performance data.

| Column | Type | Description | Example |
|--------|------|-------------|---------|
| team_event_id | INTEGER | Primary key (auto-increment) | 1 |
| event_id | INTEGER | Foreign key to events | 1 |
| team_id | INTEGER | Foreign key to teams | 1 |
| home_away | VARCHAR | Home/away designation | home |
| winner | BOOLEAN | If team won | true |
| score | INTEGER | Team score | 87 |

**Indexing:**
- Create composite indexes for common lookups:
  ```sql
  CREATE INDEX idx_team_events_event ON TeamEvents(event_id);
  CREATE INDEX idx_team_events_team ON TeamEvents(team_id, home_away);
  ```

### GameStatistics
Team statistics per game.

| Column | Type | Description | Example |
|--------|------|-------------|---------|
| game_statistic_id | INTEGER | Primary key (auto-increment) | 1 |
| team_event_id | INTEGER | Foreign key to team_events | 1 |
| statistic_type | VARCHAR | Type of statistic | FG |
| statistic_value | DECIMAL | Value | 33 |
| statistic_display | VARCHAR | Display value | 33-65 |
| percentage | DECIMAL | Percentage if applicable | 50.8 |

**Indexing:**
- Order by `team_event_id, statistic_type` to optimize querying specific stats for specific games
- This ordering maximizes zonemap effectiveness

## Statistical Data Structure

### StatCategories
Categorization of statistics (e.g., offensive, defensive, etc.).

| Column | Type | Description | Example |
|--------|------|-------------|---------|
| category_id | INTEGER | Primary key (auto-increment) | 1 |
| name | VARCHAR | Internal category name | offensive |
| display_name | VARCHAR | Display name | Offensive |
| abbreviation | VARCHAR | Short abbreviation | OFF |
| description | VARCHAR | Description of category | Statistics related to scoring and offense |
| sort_order | INTEGER | Order for display | 1 |

**Indexing:**
- No explicit indexes needed due to small table size
- Order by `sort_order` to maintain the intended display sequence

### StatTypes
Types of statistical measurements available.

| Column | Type | Description | Example |
|--------|------|-------------|---------|
| stat_type_id | INTEGER | Primary key (auto-increment) | 1 |
| name | VARCHAR | Internal stat name | pts |
| display_name | VARCHAR | Display name | Points |
| short_display_name | VARCHAR | Short name | Pts |
| description | VARCHAR | Description | Total points scored |
| abbreviation | VARCHAR | Standard abbreviation | Pts |
| type | VARCHAR | Statistical type | points |
| is_percentage | BOOLEAN | If the stat is a percentage | false |
| category_id | INTEGER | Foreign key to categories | 1 |
| is_countable | BOOLEAN | If the stat is a countable metric | true |
| is_calculated | BOOLEAN | If the stat is calculated from other stats | false |
| decimal_precision | INTEGER | Number of decimal places | 1 |
| has_per_game | BOOLEAN | If per-game average exists | true |
| has_per_minute | BOOLEAN | If per-minute average exists | false |

**Indexing:**
- Create index on `name` for lookups by stat name:
  ```sql
  CREATE INDEX idx_stat_types_name ON StatTypes(name);
  ```
- Order by `category_id, sort_order` to group related stats together

### StatScopes
Defines the scope or context of statistics.

| Column | Type | Description | Example |
|--------|------|-------------|---------|
| stat_scope_id | INTEGER | Primary key (auto-increment) | 1 |
| name | VARCHAR | Scope name | game |
| display_name | VARCHAR | Display name | Game |
| description | VARCHAR | Description | Single game statistics |
| duration_type | VARCHAR | Type of duration | event |

**Indexing:** No explicit indexes needed due to small table size.

### TeamStatisticsFact
Comprehensive team statistics fact table.

| Column | Type | Description | Example |
|--------|------|-------------|---------|
| team_stat_id | INTEGER | Primary key (auto-increment) | 1 |
| team_id | INTEGER | Foreign key to teams | 1 |
| event_id | INTEGER | Foreign key to events (NULL for season stats) | 1 |
| season_id | INTEGER | Foreign key to seasons | 1 |
| stat_type_id | INTEGER | Foreign key to stat types | 1 |
| stat_scope_id | INTEGER | Foreign key to stat scopes | 1 |
| opponent_team_id | INTEGER | Foreign key to opponent team (if applicable) | 2 |
| value | DECIMAL | Statistical value | 87.0 |
| display_value | VARCHAR | Formatted display value | 87 |
| rank | INTEGER | Ranking for this stat (if applicable) | 15 |
| is_projected | BOOLEAN | If the stat is projected/estimated | false |
| last_updated | TIMESTAMP | When record was last updated | 2024-01-10T03:45Z |

**Indexing:**
- Order by `team_id, season_id, stat_type_id` to maximize zonemap effectiveness for team stat queries
- Create compound indexes for common query patterns:
  ```sql
  CREATE INDEX idx_team_stats_team ON TeamStatisticsFact(team_id, season_id, stat_scope_id);
  CREATE INDEX idx_team_stats_event ON TeamStatisticsFact(event_id) WHERE event_id IS NOT NULL;
  CREATE INDEX idx_team_stats_stat ON TeamStatisticsFact(stat_type_id, season_id);
  ```

### AthleteStatisticsFact
Comprehensive athlete statistics fact table.

| Column | Type | Description | Example |
|--------|------|-------------|---------|
| athlete_stat_id | INTEGER | Primary key (auto-increment) | 1 |
| athlete_id | INTEGER | Foreign key to athletes | 1 |
| team_id | INTEGER | Foreign key to teams | 1 |
| event_id | INTEGER | Foreign key to events (NULL for season/career stats) | 1 |
| season_id | INTEGER | Foreign key to seasons | 1 |
| stat_type_id | INTEGER | Foreign key to stat types | 1 |
| stat_scope_id | INTEGER | Foreign key to stat scopes | 1 |
| value | DECIMAL | Statistical value | 33.0 |
| display_value | VARCHAR | Formatted display value | 33 |
| rank | INTEGER | Ranking for this stat (if applicable) | 2 |
| minutes_played | INTEGER | Minutes played (in seconds) | 2100 |
| is_career_high | BOOLEAN | If this is a career high for the athlete | false |
| is_season_high | BOOLEAN | If this is a season high for the athlete | true |
| is_projected | BOOLEAN | If the stat is projected/estimated | false |
| last_updated | TIMESTAMP | When record was last updated | 2024-01-10T03:45Z |

**Indexing:**
- Order by `athlete_id, season_id, stat_type_id` to optimize player stat queries
- Create specific indexes for common analytical patterns:
  ```sql
  CREATE INDEX idx_athlete_stats_athlete ON AthleteStatisticsFact(athlete_id, season_id, stat_scope_id);
  CREATE INDEX idx_athlete_stats_event ON AthleteStatisticsFact(event_id) WHERE event_id IS NOT NULL;
  CREATE INDEX idx_athlete_stats_stat ON AthleteStatisticsFact(stat_type_id, season_id);
  CREATE INDEX idx_athlete_stats_team ON AthleteStatisticsFact(team_id, stat_type_id, season_id);
  CREATE INDEX idx_athlete_stats_high ON AthleteStatisticsFact(athlete_id, stat_type_id) WHERE is_career_high = true OR is_season_high = true;
  ```

### LeagueAverages
League-wide statistical averages for benchmarking.

| Column | Type | Description | Example |
|--------|------|-------------|---------|
| league_avg_id | INTEGER | Primary key (auto-increment) | 1 |
| season_id | INTEGER | Foreign key to seasons | 1 |
| stat_type_id | INTEGER | Foreign key to stat types | 1 |
| stat_scope_id | INTEGER | Foreign key to stat scopes | 1 |
| group_id | INTEGER | Foreign key to groups (NULL for all) | 1 |
| value | DECIMAL | Average value | 74.2 |
| min_value | DECIMAL | Minimum recorded value | 35.0 |
| max_value | DECIMAL | Maximum recorded value | 120.0 |
| median_value | DECIMAL | Median value | 72.5 |
| stddev_value | DECIMAL | Standard deviation | 12.4 |
| last_updated | TIMESTAMP | When record was last updated | 2024-03-01T00:00Z |

**Indexing:**
- Order by `season_id, stat_type_id, group_id` to optimize lookup patterns
- No explicit indexes needed beyond the ordering

### StatisticalLeaders
Tracks statistical leaders for various categories.

| Column | Type | Description | Example |
|--------|------|-------------|---------|
| leader_id | INTEGER | Primary key (auto-increment) | 1 |
| season_id | INTEGER | Foreign key to seasons | 1 |
| stat_type_id | INTEGER | Foreign key to stat types | 1 |
| stat_scope_id | INTEGER | Foreign key to stat scopes | 1 |
| athlete_id | INTEGER | Foreign key to athletes | 1 |
| team_id | INTEGER | Foreign key to teams | 1 |
| group_id | INTEGER | Foreign key to groups (conference) | 1 |
| rank | INTEGER | Rank position | 1 |
| value | DECIMAL | Statistical value | 23.2 |
| display_value | VARCHAR | Formatted display value | 23.2 |
| last_updated | TIMESTAMP | When record was last updated | 2024-03-01T00:00Z |

**Indexing:**
- Order by `season_id, stat_type_id, rank` to optimize leaderboard queries
- Create index for athlete-specific queries:
  ```sql
  CREATE INDEX idx_stat_leaders_athlete ON StatisticalLeaders(athlete_id, season_id);
  ```

### TeamRecords
Team win/loss records.

| Column | Type | Description | Example |
|--------|------|-------------|---------|
| record_id | INTEGER | Primary key (auto-increment) | 1 |
| team_id | INTEGER | Foreign key to teams | 1 |
| season_id | INTEGER | Foreign key to seasons | 1 |
| season_type_id | INTEGER | Foreign key to season types | 1 |
| group_id | INTEGER | Foreign key to groups (NULL for overall) | 1 |
| record_type | VARCHAR | Type of record (total, home, away, etc.) | total |
| wins | INTEGER | Number of wins | 15 |
| losses | INTEGER | Number of losses | 2 |
| ties | INTEGER | Number of ties | 0 |
| win_percentage | DECIMAL | Win percentage | 0.882 |
| streak_count | INTEGER | Current streak length | 5 |
| streak_type | VARCHAR | Type of streak (W/L) | W |
| last_updated | TIMESTAMP | When record was last updated | 2024-03-01T00:00Z |

**Indexing:**
- Order by `team_id, season_id, season_type_id, record_type` to optimize team record lookups
- Create index for standings queries:
  ```sql
  CREATE INDEX idx_team_records_group ON TeamRecords(group_id, season_id, win_percentage DESC) WHERE group_id IS NOT NULL;
  ```

### BoxScores
Detailed game box score data by period.

| Column | Type | Description | Example |
|--------|------|-------------|---------|
| box_score_id | INTEGER | Primary key (auto-increment) | 1 |
| event_id | INTEGER | Foreign key to events | 1 |
| team_id | INTEGER | Foreign key to teams | 1 |
| period | INTEGER | Game period (1=1st half, 2=2nd half, etc.) | 1 |
| period_type | VARCHAR | Type of period (REGULAR, OT) | REGULAR |
| score | INTEGER | Points scored in period | 42 |
| cumulative_score | INTEGER | Cumulative score | 42 |
| last_updated | TIMESTAMP | When record was last updated | 2024-01-10T03:45Z |

**Indexing:**
- Order by `event_id, team_id, period` to optimize box score lookups
- No explicit indexes needed with proper ordering

### PlayByPlay
Detailed play-by-play data.

| Column | Type | Description | Example |
|--------|------|-------------|---------|
| play_id | INTEGER | Primary key (auto-increment) | 1 |
| event_id | INTEGER | Foreign key to events | 1 |
| sequence_number | INTEGER | Play sequence in game | 243 |
| period | INTEGER | Game period | 2 |
| clock_time | VARCHAR | Game clock at time of play | 3:42 |
| clock_seconds_remaining | INTEGER | Seconds remaining in period | 222 |
| team_id | INTEGER | Foreign key to acting team | 1 |
| athlete_id | INTEGER | Foreign key to primary athlete | 1 |
| secondary_athlete_id | INTEGER | Foreign key to secondary athlete | 2 |
| play_type | VARCHAR | Type of play | SHOT |
| play_text | VARCHAR | Description of play | Edey made Dunk. Assisted by Loyer. |
| score_home | INTEGER | Home team score after play | 85 |
| score_away | INTEGER | Away team score after play | 78 |
| coordinates_x | DECIMAL | X-coordinate of play | 50.0 |
| coordinates_y | DECIMAL | Y-coordinate of play | 25.0 |
| importance | DECIMAL | Play importance score | 0.85 |
| success | BOOLEAN | If the play was successful | true |
| scoring_play | BOOLEAN | If the play resulted in points | true |
| points | INTEGER | Points scored on play | 2 |
| possession_prior | INTEGER | Team ID with possession before | 1 |
| possession_after | INTEGER | Team ID with possession after | 2 |
| timestamp | TIMESTAMP | Wall clock time of play | 2024-01-10T01:45:32Z |

**Indexing:**
- Order by `event_id, sequence_number` to maintain chronological ordering within games
- Create indexes for specialized queries:
  ```sql
  CREATE INDEX idx_play_by_play_athlete ON PlayByPlay(athlete_id, event_id);
  CREATE INDEX idx_play_by_play_important ON PlayByPlay(event_id, importance DESC) WHERE importance > 0.7;
  CREATE INDEX idx_play_by_play_scoring ON PlayByPlay(event_id, team_id) WHERE scoring_play = true;
  ```

### StatCalculations
Defines formulas for calculated statistics.

| Column | Type | Description | Example |
|--------|------|-------------|---------|
| calculation_id | INTEGER | Primary key (auto-increment) | 1 |
| target_stat_type_id | INTEGER | Foreign key to the calculated stat | 15 |
| formula | VARCHAR | SQL or math formula | (pts + reb*1.2 + ast*1.5 + stl*3 + blk*3 - to*1) / minutes_played |
| description | VARCHAR | Description of calculation | Player Efficiency Rating |
| components | VARCHAR | JSON array of component stat IDs | [1, 2, 3, 4, 5, 6] |
| last_updated | TIMESTAMP | When calculation was last updated | 2024-03-01T00:00Z |

**Indexing:** No explicit indexes needed due to small table size.

### ShotCharts
Shot location and outcome data.

| Column | Type | Description | Example |
|--------|------|-------------|---------|
| shot_id | INTEGER | Primary key (auto-increment) | 1 |
| event_id | INTEGER | Foreign key to events | 1 |
| play_id | INTEGER | Foreign key to play-by-play | 243 |
| team_id | INTEGER | Foreign key to teams | 1 |
| athlete_id | INTEGER | Foreign key to athletes | 1 |
| assist_athlete_id | INTEGER | Foreign key to assisting athlete | 2 |
| period | INTEGER | Game period | 2 |
| clock_time | VARCHAR | Game clock at time of shot | 3:42 |
| shot_type | VARCHAR | Type of shot | DUNK |
| shot_distance | DECIMAL | Distance in feet | 0.5 |
| coordinates_x | DECIMAL | X-coordinate of shot | 50.0 |
| coordinates_y | DECIMAL | Y-coordinate of shot | 25.0 |
| made | BOOLEAN | If shot was made | true |
| blocked | BOOLEAN | If shot was blocked | false |
| points | INTEGER | Points from shot | 2 |
| fast_break | BOOLEAN | If shot was on fast break | false |
| second_chance | BOOLEAN | If shot was second chance | false |
| timestamp | TIMESTAMP | Wall clock time of shot | 2024-01-10T01:45:32Z |

**Indexing:**
- Order by `event_id, period, clock_seconds_remaining` to chronologically order shots within games
- Create indexes for shot analysis:
  ```sql
  CREATE INDEX idx_shot_charts_athlete ON ShotCharts(athlete_id, made);
  CREATE INDEX idx_shot_charts_distance ON ShotCharts(athlete_id, shot_distance, made);
  CREATE INDEX idx_shot_charts_spatial ON ShotCharts(athlete_id, coordinates_x, coordinates_y, made);
  ```

## Materialized Views

For common analytical queries, we recommend creating materialized views to improve performance:

```sql
-- Player game averages by season
CREATE MATERIALIZED VIEW player_season_averages AS
SELECT 
    a.athlete_id, 
    a.full_name,
    t.team_id,
    f.display_name AS franchise,
    s.season_id,
    s.display_name AS season,
    COUNT(DISTINCT asf.event_id) AS games_played,
    AVG(CASE WHEN st.name = 'pts' THEN asf.value ELSE NULL END) AS ppg,
    AVG(CASE WHEN st.name = 'rpg' THEN asf.value ELSE NULL END) AS rpg,
    AVG(CASE WHEN st.name = 'apg' THEN asf.value ELSE NULL END) AS apg,
    AVG(CASE WHEN st.name = 'spg' THEN asf.value ELSE NULL END) AS spg,
    AVG(CASE WHEN st.name = 'bpg' THEN asf.value ELSE NULL END) AS bpg,
    AVG(CASE WHEN st.name = 'fg%' THEN asf.value ELSE NULL END) AS fg_pct,
    AVG(CASE WHEN st.name = '3p%' THEN asf.value ELSE NULL END) AS fg3_pct
FROM 
    AthleteStatisticsFact asf
JOIN Athletes a ON asf.athlete_id = a.athlete_id
JOIN Teams t ON asf.team_id = t.team_id
JOIN Franchises f ON t.franchise_id = f.franchise_id
JOIN Seasons s ON asf.season_id = s.season_id
JOIN StatTypes st ON asf.stat_type_id = st.stat_type_id
JOIN StatScopes ss ON asf.stat_scope_id = ss.stat_scope_id
WHERE 
    ss.name = 'game'
    AND st.name IN ('pts', 'rpg', 'apg', 'spg', 'bpg', 'fg%', '3p%')
GROUP BY 
    a.athlete_id, a.full_name, t.team_id, f.display_name, 
    s.season_id, s.display_name
ORDER BY 
    s.season_id DESC, a.full_name;

-- Shot success rate by court location
CREATE MATERIALIZED VIEW shot_success_by_location AS
SELECT 
    sc.athlete_id,
    a.full_name,
    s.season_id,
    s.display_name AS season,
    FLOOR(sc.coordinates_x/5)*5 AS court_x,
    FLOOR(sc.coordinates_y/5)*5 AS court_y,
    COUNT(*) AS attempts,
    SUM(CASE WHEN sc.made THEN 1 ELSE 0 END) AS makes,
    SUM(CASE WHEN sc.made THEN 1 ELSE 0 END)::FLOAT / COUNT(*) AS success_rate,
    AVG(sc.shot_distance) AS avg_distance,
    SUM(sc.points) AS total_points
FROM 
    ShotCharts sc
JOIN Athletes a ON sc.athlete_id = a.athlete_id
JOIN Events e ON sc.event_id = e.event_id
JOIN Seasons s ON e.season_id = s.season_id
GROUP BY 
    sc.athlete_id, a.full_name, s.season_id, s.display_name,
    FLOOR(sc.coordinates_x/5)*5, FLOOR(sc.coordinates_y/5)*5
ORDER BY 
    sc.athlete_id, s.season_id;
```

## Memory Management

Since DuckDB indexes can consume significant memory that is not automatically buffer-managed, consider these strategies:

1. Create a maintenance routine to detach and reattach the database periodically:
   ```sql
   -- In a maintenance script
   DETACH DATABASE espn_stats;
   ATTACH DATABASE 'path/to/espn_stats.db' AS espn_stats;
   ```

2. Monitor memory usage and selectively disable indexes when memory pressure is high:
   ```sql
   -- Disable index usage temporarily
   PRAGMA disable_index_scan;
   -- Run memory-intensive operations
   -- Re-enable index usage
   PRAGMA enable_index_scan;
   ```

## Progress Tracking

| Entity | Status | Notes |
|--------|--------|-------|
| Leagues | Mapped | Basic structure established |
| Seasons | Mapped | Basic structure established |
| Season Types | Mapped | Basic structure established |
| Weeks | Mapped | Basic structure established |
| Franchises | Mapped | Basic structure established |
| Teams | Mapped | Basic structure established |
| Groups | Mapped | Basic structure established |
| Venues | Mapped | Basic structure established |
| Athletes | Mapped | Basic structure established |
| Positions | Mapped | Basic structure established |
| Events | Mapped | Basic structure established |
| Rankings | Mapped | Basic structure established |
| Statistics | Refined | Enhanced with detailed dimensions and facts |
| Play-by-Play | Added | Detailed event timeline data |
| Box Scores | Added | Period-by-period scoring |
| Shot Charts | Added | Spatial shot data |
| Indexing | Added | DuckDB-specific indexing strategy defined |

## Next Steps

1. ✅ Refine the granularity of statistical data based on detailed endpoint study
2. ✅ Determine appropriate indexing strategy
3. Develop extraction and data loading process
4. Establish data refresh patterns (incremental vs. full)
5. Create view layer for simplified analytics access
6. Consider partitioning strategy for historical data
7. Develop approach for derived/calculated statistics
8. Create test queries for common analytical scenarios

## Open Questions

- How should we handle very sparse statistical fields that only apply to certain athletes or game situations?
- What is the best storage format for historical time-series analysis of player development?
- Should we implement a separate star schema for analytical queries on top of this normalized structure?
- How frequently should aggregate statistics be recalculated? 