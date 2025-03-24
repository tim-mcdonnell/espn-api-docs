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

[TOURNAMENTS] ←→ [SEASONS]
      ↑             ↑
      ↓             ↓
[TOURNAMENT_BRACKETS] ←→ [TOURNAMENT_ROUNDS] ←→ [TOURNAMENT_SEEDS]
      ↑                          ↑                      ↑
      ↓                          ↓                      ↓
[TOURNAMENT_MATCHUPS] ←→ [EVENTS/GAMES] ←→ [TEAMS] ←→ [TOURNAMENT_ADVANCEMENT]
      ↑                          ↑             ↑             ↑
      ↓                          ↓             ↓             ↓
[TOURNAMENT_BIDS] ←→ [SELECTION_CRITERIA] ←→ [SELECTION_METRICS] ←→ [BRACKETOLOGY_PROJECTIONS]
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

## Rankings Data Structure

### RankingSystems
Stores information about different ranking systems and polls.

| Column | Type | Description | Example |
|--------|------|-------------|---------|
| ranking_system_id | INTEGER | Primary key (auto-increment) | 1 |
| espn_id | VARCHAR | ESPN's identifier | 1 |
| name | VARCHAR | Full name of the ranking system | AP Top 25 |
| short_name | VARCHAR | Abbreviated name | AP |
| description | VARCHAR | Description of the ranking system | Associated Press Top 25 Poll |
| poll_frequency | VARCHAR | How often the poll is released | weekly |
| poll_day | VARCHAR | Day of week poll is released | Monday |
| first_place_votes | BOOLEAN | If the poll includes first place votes | true |
| points_based | BOOLEAN | If the poll is based on points | true |
| voter_count | INTEGER | Number of voters in the poll | 61 |
| voters | VARCHAR | JSON array of voter information | [{"name": "John Smith", "affiliation": "Tribune"}] |
| active | BOOLEAN | If the poll is currently active | true |
| historical_cutoff | INTEGER | Number of ranked teams historically | 25 |

**Indexing:**
- No explicit indexes needed due to small table size
- Order by `name` to maintain alphabetical ordering

### PollSchedule
Tracks when polls are released across seasons.

| Column | Type | Description | Example |
|--------|------|-------------|---------|
| poll_schedule_id | INTEGER | Primary key (auto-increment) | 1 |
| ranking_system_id | INTEGER | Foreign key to ranking systems | 1 |
| season_id | INTEGER | Foreign key to seasons | 1 |
| week_id | INTEGER | Foreign key to weeks | 1 |
| release_date | TIMESTAMP | When the poll was released | 2024-03-11T18:00Z |
| poll_name | VARCHAR | Name for this specific poll | Week 16 AP Poll |
| notes | VARCHAR | Any special notes for this poll | Final poll before tournament |

**Indexing:**
- Order by `ranking_system_id, season_id, release_date` to optimize chronological ordering
- Create index for week-based lookups:
  ```sql
  CREATE INDEX idx_poll_schedule_week ON PollSchedule(season_id, week_id, ranking_system_id);
  ```

### TeamRankings
Historical team rankings in various polls over time.

| Column | Type | Description | Example |
|--------|------|-------------|---------|
| team_ranking_id | INTEGER | Primary key (auto-increment) | 1 |
| poll_schedule_id | INTEGER | Foreign key to poll schedule | 1 |
| team_id | INTEGER | Foreign key to teams | 1 |
| rank | INTEGER | Current rank | 1 |
| previous_rank | INTEGER | Previous rank | 2 |
| points | DECIMAL | Total points received | 1571.0 |
| first_place_votes | INTEGER | Number of first-place votes | 61 |
| trend | VARCHAR | Direction of movement | up |
| weeks_ranked | INTEGER | Consecutive weeks in rankings | 15 |
| highest_rank | INTEGER | Highest rank this season | 1 |
| dropped_out | BOOLEAN | If team dropped out of rankings | false |
| receiving_votes | BOOLEAN | If receiving votes but not ranked | false |
| points_when_unranked | DECIMAL | Points received when unranked | 15.0 |

**Indexing:**
- Order by `poll_schedule_id, rank` to optimize rank order lookups
- Create composite indexes for common query patterns:
  ```sql
  CREATE INDEX idx_team_rankings_team ON TeamRankings(team_id, poll_schedule_id);
  CREATE INDEX idx_team_rankings_movement ON TeamRankings(poll_schedule_id) WHERE trend = 'up' OR trend = 'down';
  CREATE INDEX idx_team_rankings_votes ON TeamRankings(poll_schedule_id, receiving_votes) WHERE receiving_votes = true;
  ```

### RankingComparisons
Pre-calculated comparisons between different ranking systems.

| Column | Type | Description | Example |
|--------|------|-------------|---------|
| comparison_id | INTEGER | Primary key (auto-increment) | 1 |
| team_id | INTEGER | Foreign key to teams | 1 |
| season_id | INTEGER | Foreign key to seasons | 1 |
| week_id | INTEGER | Foreign key to weeks | 1 |
| primary_ranking_system_id | INTEGER | Primary ranking system | 1 |
| primary_rank | INTEGER | Rank in primary system | 5 |
| comparison_ranking_system_id | INTEGER | System being compared | 2 |
| comparison_rank | INTEGER | Rank in comparison system | 7 |
| rank_difference | INTEGER | Difference between ranks | -2 |
| last_updated | TIMESTAMP | When comparison was calculated | 2024-03-11T22:00Z |

**Indexing:**
- Order by `season_id, week_id, team_id` to optimize lookups for specific teams
- Create index for large discrepancies:
  ```sql
  CREATE INDEX idx_ranking_comparisons_diff ON RankingComparisons(season_id, week_id, ABS(rank_difference) DESC) WHERE ABS(rank_difference) > 5;
  ```

## Awards Data Structure

### AwardCategories
Defines different awards and honors available in the league.

| Column | Type | Description | Example |
|--------|------|-------------|---------|
| award_category_id | INTEGER | Primary key (auto-increment) | 1 |
| espn_id | VARCHAR | ESPN's identifier | 1 |
| name | VARCHAR | Full name of the award | Player of the Year |
| display_name | VARCHAR | Display name | Player of the Year |
| short_display_name | VARCHAR | Shortened display name | POY |
| description | VARCHAR | Description of award criteria | Annual award given to the top player in NCAA Men's Basketball |
| level | VARCHAR | Level of award (national, conference) | national |
| organization | VARCHAR | Organization presenting award | NABC |
| established_year | INTEGER | Year the award was established | 1969 |
| active | BOOLEAN | If the award is currently active | true |
| award_type | VARCHAR | Type of award (player, coach, team) | player |
| frequency | VARCHAR | How often the award is given | annual |
| historical_notes | VARCHAR | Notes on history/significance | Originally called the UPI Player of the Year |

**Indexing:**
- Order by `level, name` to group related awards together
- No explicit indexes needed due to small table size

### AwardRecipients
Tracks winners of awards across seasons.

| Column | Type | Description | Example |
|--------|------|-------------|---------|
| recipient_id | INTEGER | Primary key (auto-increment) | 1 |
| award_category_id | INTEGER | Foreign key to award categories | 1 |
| season_id | INTEGER | Foreign key to seasons | 1 |
| athlete_id | INTEGER | Foreign key to athletes (NULL for team/coach awards) | 1 |
| team_id | INTEGER | Foreign key to teams | 1 |
| coach_id | INTEGER | Foreign key to coaches (NULL for player/team awards) | NULL |
| award_date | TIMESTAMP | When the award was given | 2024-04-05T00:00Z |
| citation | VARCHAR | Award citation/reason | Led the nation in scoring and rebounding |
| rank | INTEGER | Rank (for runner-ups/honorable mention) | 1 |
| unanimous | BOOLEAN | If the selection was unanimous | true |
| shared | BOOLEAN | If the award was shared | false |
| co_recipients | VARCHAR | JSON array of co-recipient IDs | NULL |
| presentation_url | VARCHAR | Link to presentation video/article | https://example.com/award-ceremony |

**Indexing:**
- Order by `award_category_id, season_id, rank` to optimize lookups by award and season
- Create indexes for athlete and team lookups:
  ```sql
  CREATE INDEX idx_award_recipients_athlete ON AwardRecipients(athlete_id) WHERE athlete_id IS NOT NULL;
  CREATE INDEX idx_award_recipients_team ON AwardRecipients(team_id);
  ```

### AllStarTeams
Tracks all-conference, all-American, and other honorary teams.

| Column | Type | Description | Example |
|--------|------|-------------|---------|
| all_star_id | INTEGER | Primary key (auto-increment) | 1 |
| name | VARCHAR | Team name | All-ACC First Team |
| season_id | INTEGER | Foreign key to seasons | 1 |
| level | VARCHAR | Level (All-American, All-Conference) | conference |
| group_id | INTEGER | Foreign key to groups/conferences (NULL for national) | 1 |
| team_type | VARCHAR | Type of team (first, second, third, honorable mention) | first |
| organization | VARCHAR | Organization selecting the team | ACC |
| selection_date | TIMESTAMP | When the team was announced | 2024-03-11T18:00Z |

**Indexing:**
- Order by `season_id, level, team_type` to group related selections together
- No explicit indexes needed due to small table size

### AllStarSelections
Links athletes to all-star teams.

| Column | Type | Description | Example |
|--------|------|-------------|---------|
| selection_id | INTEGER | Primary key (auto-increment) | 1 |
| all_star_id | INTEGER | Foreign key to all-star teams | 1 |
| athlete_id | INTEGER | Foreign key to athletes | 1 |
| team_id | INTEGER | Foreign key to athlete's team | 1 |
| position_id | INTEGER | Foreign key to positions | 1 |
| unanimous | BOOLEAN | If selection was unanimous | true |
| captain | BOOLEAN | If selected as team captain | false |
| honor_type | VARCHAR | Type of honor (regular, captain, MVP) | regular |
| votes | INTEGER | Number of votes received | 125 |
| order_num | INTEGER | Order listed on team | 1 |

**Indexing:**
- Order by `all_star_id, order_num` to maintain intended display order
- Create index for athlete career accolades:
  ```sql
  CREATE INDEX idx_all_star_selections_athlete ON AllStarSelections(athlete_id, season_id);
  ```

## Broadcast Data Structure

### Networks
Stores information about broadcast networks/channels.

| Column | Type | Description | Example |
|--------|------|-------------|---------|
| network_id | INTEGER | Primary key (auto-increment) | 1 |
| espn_id | VARCHAR | ESPN's identifier (if any) | 101 |
| name | VARCHAR | Full network name | Big Ten Network |
| short_name | VARCHAR | Network abbreviation | BTN |
| type | VARCHAR | Network type | TV |
| logo_url | VARCHAR | URL to network logo | https://a.espncdn.com/i/networks/btn.png |
| parent_company | VARCHAR | Parent company | Fox Corporation |
| is_subscription | BOOLEAN | If requires paid subscription | false |

**Indexing:**
- No explicit indexes needed due to small table size
- Order by `name` for quick lookups

### BroadcastMarkets
Defines different market types and regions.

| Column | Type | Description | Example |
|--------|------|-------------|---------|
| market_id | INTEGER | Primary key (auto-increment) | 1 |
| espn_id | VARCHAR | ESPN's identifier | 1 |
| name | VARCHAR | Market name | National |
| type | VARCHAR | Market type | National/Regional/Local |
| region | VARCHAR | Regional identifier (if applicable) | NULL |
| countries | VARCHAR | JSON array of country codes | ["US", "CA"] |
| description | VARCHAR | Market description | Available throughout US |

**Indexing:**
- No explicit indexes needed due to small table size

### Announcers
People who call/announce games.

| Column | Type | Description | Example |
|--------|------|-------------|---------|
| announcer_id | INTEGER | Primary key (auto-increment) | 1 |
| espn_id | VARCHAR | ESPN's identifier | 5678 |
| first_name | VARCHAR | First name | Jim |
| last_name | VARCHAR | Last name | Nantz |
| full_name | VARCHAR | Full name | Jim Nantz |
| role | VARCHAR | Announcer role | Play-by-play |
| bio | VARCHAR | Short biography | CBS Sports lead announcer |
| headshot_url | VARCHAR | URL to headshot image | https://example.com/announcers/nantz.jpg |
| active | BOOLEAN | If currently active | true |

**Indexing:**
- Create index on `last_name, first_name` for name lookups:
  ```sql
  CREATE INDEX idx_announcers_name ON Announcers(last_name, first_name);
  ```

### EventBroadcasts
Broadcasts linked to specific events.

| Column | Type | Description | Example |
|--------|------|-------------|---------|
| broadcast_id | INTEGER | Primary key (auto-increment) | 1 |
| event_id | INTEGER | Foreign key to events | 1 |
| network_id | INTEGER | Foreign key to networks | 1 |
| market_id | INTEGER | Foreign key to broadcast markets | 1 |
| type | VARCHAR | Broadcast type | TV |
| language | VARCHAR | Broadcast language | en |
| start_time | TIMESTAMP | Scheduled start time | 2024-01-10T23:30Z |
| end_time | TIMESTAMP | Scheduled end time | 2024-01-11T01:30Z |
| broadcast_name | VARCHAR | Specific broadcast name | College Basketball |
| channel_number | VARCHAR | Channel number | 610 |
| is_national | BOOLEAN | If nationally televised | true |
| is_home_broadcast | BOOLEAN | If favoring home team | false |
| tape_delayed | BOOLEAN | If not live | false |
| region_restriction | VARCHAR | Regional blackout info | NULL |
| stream_url | VARCHAR | URL to stream (if applicable) | NULL |

**Indexing:**
- Order by `event_id, type` to group broadcasts by type per event
- Create index for network-specific queries:
  ```sql
  CREATE INDEX idx_event_broadcasts_network ON EventBroadcasts(network_id, start_time);
  ```

### BroadcastAnnouncers
Links announcers to specific broadcasts with their roles.

| Column | Type | Description | Example |
|--------|------|-------------|---------|
| broadcast_announcer_id | INTEGER | Primary key (auto-increment) | 1 |
| broadcast_id | INTEGER | Foreign key to event broadcasts | 1 |
| announcer_id | INTEGER | Foreign key to announcers | 1 |
| role | VARCHAR | Role for this broadcast | Play-by-play |
| order_num | INTEGER | Display/priority order | 1 |
| is_primary | BOOLEAN | If primary announcer | true |
| is_sideline | BOOLEAN | If sideline reporter | false |
| notes | VARCHAR | Any specific notes | Guest analyst |

**Indexing:**
- Order by `broadcast_id, order_num` to maintain crew order
- No explicit indexes needed beyond this ordering

### BroadcastRatings
Viewership and ratings data for broadcasts.

| Column | Type | Description | Example |
|--------|------|-------------|---------|
| rating_id | INTEGER | Primary key (auto-increment) | 1 |
| broadcast_id | INTEGER | Foreign key to broadcasts | 1 |
| rating_type | VARCHAR | Type of rating | Nielsen |
| rating_value | DECIMAL | Rating number | 2.4 |
| viewers | INTEGER | Number of viewers (thousands) | 1250 |
| share | DECIMAL | Market share percentage | 5.2 |
| demographic | VARCHAR | Target demographic | P18-49 |
| time_slot | VARCHAR | Broadcasting time slot | Primetime |
| competing_events | VARCHAR | JSON array of competing broadcasts | [{"sport": "NBA", "rating": 3.1}] |
| notes | VARCHAR | Additional context | Conference rivalry game |

**Indexing:**
- Order by `broadcast_id, rating_type` for easy lookups
- Create index for high-rated broadcasts:
  ```sql
  CREATE INDEX idx_broadcast_ratings_value ON BroadcastRatings(rating_value DESC);
  ```

## Play-by-Play Data Structure

The Play-by-Play Data Structure extends our normalized database to capture detailed event timelines, providing play-by-play information for each game. This enables advanced analysis of game flow, player performance in specific situations, and detailed event reconstruction.

### PlayTypes
Categorizes different types of basketball plays.

| Column | Type | Description | Example |
|--------|------|-------------|---------|
| play_type_id | INTEGER | Primary key (auto-increment) | 1 |
| espn_id | VARCHAR | ESPN's identifier | 404 |
| name | VARCHAR | Play type name | Missed Jump Shot |
| abbreviation | VARCHAR | Standard abbreviation | MISSJUMP |
| description | VARCHAR | Description of play type | Player attempted but missed a jump shot |
| category | VARCHAR | General category | Shot |
| is_scoring_play | BOOLEAN | If play can result in points | true |
| points_possible | INTEGER | Maximum points possible | 3 |
| related_stat_type_id | INTEGER | Foreign key to stat types | 21 |

**Indexing:**
- Create explicit index on `espn_id` for API integration:
  ```sql
  CREATE INDEX idx_play_types_espn_id ON PlayTypes(espn_id);
  ```
- Order by `category, name` to group related play types together

### Plays
The core table storing individual plays within games.

| Column | Type | Description | Example |
|--------|------|-------------|---------|
| play_id | INTEGER | Primary key (auto-increment) | 1 |
| espn_id | VARCHAR | ESPN's identifier | 401524661:2 |
| event_id | INTEGER | Foreign key to events | 1 |
| sequence_number | INTEGER | Order of play within game | 2 |
| play_type_id | INTEGER | Foreign key to play types | 1 |
| team_id | INTEGER | Foreign key to team performing action | 1 |
| period | INTEGER | Game period (1=1st half, 2=2nd half) | 1 |
| clock_time | VARCHAR | Game clock at time of play | 19:36 |
| clock_seconds_remaining | INTEGER | Seconds remaining in period | 1176 |
| wallclock | TIMESTAMP | Real-world timestamp | 2024-01-10T00:00:24Z |
| text | VARCHAR | Description of the play | Brice Williams missed Three Point Jumper |
| score_value | INTEGER | Points scored on the play | 0 |
| home_score | INTEGER | Home team score after play | 0 |
| away_score | INTEGER | Away team score after play | 0 |
| score_differential | INTEGER | Score differential after play | 0 |
| win_probability | DECIMAL | Win probability after play | 0.52 |
| possession_prior | INTEGER | Team ID with possession before play | 1 |
| possession_after | INTEGER | Team ID with possession after play | 2 |
| scoring_play | BOOLEAN | If points were scored | false |
| shot_type | VARCHAR | Type of shot if applicable | Three Point Jumper |
| shot_made | BOOLEAN | If shot was made | false |
| assist | BOOLEAN | If play was an assist | false |
| turnover | BOOLEAN | If play was a turnover | false |
| foul | BOOLEAN | If play was a foul | false |
| timeout | BOOLEAN | If play was a timeout | false |
| coordinates_x | DECIMAL | X coordinate of play on court | 23.5 |
| coordinates_y | DECIMAL | Y coordinate of play on court | 7.2 |
| is_timeout | BOOLEAN | If play represents a timeout | false |
| timeout_team | INTEGER | Team calling timeout | NULL |
| timeout_type | VARCHAR | Type of timeout | NULL |
| fastbreak | BOOLEAN | If play occurred in fastbreak | false |
| second_chance | BOOLEAN | If play was a second chance opportunity | false |
| paint | BOOLEAN | If play occurred in the paint | false |

**Indexing:**
- Order by `event_id, sequence_number` to optimize chronological querying
- Create explicit indexes for common query patterns:
  ```sql
  CREATE INDEX idx_plays_event ON Plays(event_id);
  CREATE INDEX idx_plays_team ON Plays(team_id, event_id);
  CREATE INDEX idx_plays_scoring ON Plays(event_id) WHERE scoring_play = true;
  CREATE INDEX idx_plays_period_end ON Plays(event_id, period) WHERE clock_seconds_remaining < 60;
  ```

### PlayParticipants
Links athletes to specific plays with their roles.

| Column | Type | Description | Example |
|--------|------|-------------|---------|
| play_participant_id | INTEGER | Primary key (auto-increment) | 1 |
| play_id | INTEGER | Foreign key to plays | 1 |
| athlete_id | INTEGER | Foreign key to athletes | 1 |
| team_id | INTEGER | Foreign key to teams | 1 |
| role | VARCHAR | Role in the play | shooter |
| primary_participant | BOOLEAN | If athlete is primary actor | true |
| stat_value | INTEGER | Statistical value for participant | 1 |
| stat_type_id | INTEGER | Foreign key to stat types | 1 |
| jersey | VARCHAR | Jersey number during play | 15 |

**Indexing:**
- Order by `play_id, primary_participant DESC` to prioritize primary participants
- Create indexes for common analysis queries:
  ```sql
  CREATE INDEX idx_play_participants_athlete ON PlayParticipants(athlete_id, team_id);
  CREATE INDEX idx_play_participants_role ON PlayParticipants(role, play_id);
  ```

### PlayStreaks
Tracks significant streaks within games.

| Column | Type | Description | Example |
|--------|------|-------------|---------|
| streak_id | INTEGER | Primary key (auto-increment) | 1 |
| event_id | INTEGER | Foreign key to events | 1 |
| start_play_id | INTEGER | Foreign key to starting play | 10 |
| end_play_id | INTEGER | Foreign key to ending play | 20 |
| team_id | INTEGER | Foreign key to team | 1 |
| streak_type | VARCHAR | Type of streak | scoring_run |
| value | INTEGER | Numeric value of streak | 10 |
| start_period | INTEGER | Period when streak started | 1 |
| end_period | INTEGER | Period when streak ended | 1 |
| start_time | VARCHAR | Game clock when streak started | 12:45 |
| end_time | VARCHAR | Game clock when streak ended | 10:23 |
| home_score_start | INTEGER | Home score at streak start | 5 |
| away_score_start | INTEGER | Away score at streak start | 8 |
| home_score_end | INTEGER | Home score at streak end | 15 |
| away_score_end | INTEGER | Away score at streak end | 8 |
| seconds_elapsed | INTEGER | Real-time seconds elapsed | 142 |

**Indexing:**
- Order by `event_id, start_play_id` for chronological ordering
- Create index for significant streaks:
  ```sql
  CREATE INDEX idx_play_streaks_value ON PlayStreaks(event_id, value DESC);
  ```

### GameFlowMetrics
Stores derived metrics about game flow and momentum.

| Column | Type | Description | Example |
|--------|------|-------------|---------|
| metric_id | INTEGER | Primary key (auto-increment) | 1 |
| event_id | INTEGER | Foreign key to events | 1 |
| period | INTEGER | Game period | 1 |
| minute_segment | INTEGER | Minute of game (bucketed) | 5 |
| team_id | INTEGER | Foreign key to teams | 1 |
| metric_type | VARCHAR | Type of metric | momentum_score |
| metric_value | DECIMAL | Value of the metric | 0.75 |
| score_differential | INTEGER | Score differential | 7 |
| lead_changes | INTEGER | Lead changes in segment | 2 |
| largest_lead | INTEGER | Largest lead in segment | 8 |
| efficiency_rating | DECIMAL | Offensive efficiency | 1.2 |
| possessions | INTEGER | Possession count | 5 |
| points_per_possession | DECIMAL | Points per possession | 1.4 |
| fast_break_points | INTEGER | Fast break points | 6 |
| second_chance_points | INTEGER | Second chance points | 2 |
| points_in_paint | INTEGER | Points in the paint | 8 |
| points_off_turnovers | INTEGER | Points off turnovers | 4 |

**Indexing:**
- Order by `event_id, period, minute_segment` for time-series analysis
- Create index for team-specific queries:
  ```sql
  CREATE INDEX idx_game_flow_team ON GameFlowMetrics(team_id, event_id);
  ```

### GameSegments
Defines significant game segments for analysis.

| Column | Type | Description | Example |
|--------|------|-------------|---------|
| segment_id | INTEGER | Primary key (auto-increment) | 1 |
| event_id | INTEGER | Foreign key to events | 1 |
| segment_type | VARCHAR | Type of segment | crunch_time |
| start_play_id | INTEGER | Foreign key to starting play | 280 |
| end_play_id | INTEGER | Foreign key to ending play | 304 |
| start_period | INTEGER | Period when segment started | 2 |
| end_period | INTEGER | Period when segment ended | 2 |
| start_time | VARCHAR | Game clock when segment started | 5:00 |
| end_time | VARCHAR | Game clock when segment ended | 0:00 |
| home_score_start | INTEGER | Home score at segment start | 70 |
| away_score_start | INTEGER | Away score at segment start | 68 |
| home_score_end | INTEGER | Home score at segment end | 82 |
| away_score_end | INTEGER | Away score at segment end | 75 |
| description | VARCHAR | Description of segment | Final five minutes |
| significance | INTEGER | Significance score (1-10) | 9 |

**Indexing:**
- Order by `event_id, segment_type` to group related segments
- Create index for significant segments:
  ```sql
  CREATE INDEX idx_game_segments_significance ON GameSegments(event_id, significance DESC);
  ```

### PlaysTagMap
Provides a flexible tagging system for plays.

| Column | Type | Description | Example |
|--------|------|-------------|---------|
| plays_tag_id | INTEGER | Primary key (auto-increment) | 1 |
| play_id | INTEGER | Foreign key to plays | 1 |
| tag | VARCHAR | Tag applied to play | clutch |
| tag_category | VARCHAR | Category of tag | significance |
| tag_value | DECIMAL | Numeric value if applicable | 0.9 |

**Indexing:**
- Order by `play_id, tag_category, tag` for efficient play retrieval
- Create index for specific tag queries:
  ```sql
  CREATE INDEX idx_plays_tag_map_tag ON PlaysTagMap(tag, tag_category);
  ```

## Materialized Views for Play-by-Play Analysis

For efficient analysis of play-by-play data, we recommend creating these materialized views:

```sql
-- Game momentum analysis
CREATE MATERIALIZED VIEW game_momentum AS
SELECT 
    e.event_id,
    e.name AS event_name,
    e.start_date AS game_date,
    p.sequence_number,
    p.period,
    p.clock_time,
    p.text AS play_description,
    th.display_name AS home_team,
    ta.display_name AS away_team,
    p.home_score,
    p.away_score,
    p.score_differential,
    p.win_probability,
    CASE WHEN p.team_id = te.home_team_id THEN 'home' ELSE 'away' END AS team_type,
    t.display_name AS acting_team,
    pt.name AS play_type,
    CASE 
        WHEN LAG(p.win_probability) OVER(PARTITION BY p.event_id ORDER BY p.sequence_number) IS NULL THEN 0
        ELSE p.win_probability - LAG(p.win_probability) OVER(PARTITION BY p.event_id ORDER BY p.sequence_number)
    END AS win_prob_change
FROM 
    Plays p
JOIN PlayTypes pt ON p.play_type_id = pt.play_type_id
JOIN Events e ON p.event_id = e.event_id
JOIN TeamEvents te ON e.event_id = te.event_id
JOIN Teams t ON p.team_id = t.team_id
JOIN Teams th ON te.team_id = th.team_id AND te.home_away = 'home'
JOIN Teams ta ON te.team_id = ta.team_id AND te.home_away = 'away'
ORDER BY 
    e.event_id, p.sequence_number;

-- Player play-by-play contribution
CREATE MATERIALIZED VIEW player_play_contribution AS
SELECT 
    pp.play_id,
    p.event_id,
    p.sequence_number,
    p.period,
    p.clock_time,
    p.text AS play_description,
    a.full_name AS athlete_name,
    t.display_name AS team_name,
    pp.role,
    pt.name AS play_type,
    p.score_value,
    st.name AS stat_type,
    pp.stat_value,
    p.scoring_play,
    p.coordinates_x,
    p.coordinates_y
FROM 
    PlayParticipants pp
JOIN Plays p ON pp.play_id = p.play_id
JOIN Athletes a ON pp.athlete_id = a.athlete_id
JOIN Teams t ON pp.team_id = t.team_id
JOIN PlayTypes pt ON p.play_type_id = pt.play_type_id
LEFT JOIN StatTypes st ON pp.stat_type_id = st.stat_type_id
ORDER BY 
    p.event_id, p.sequence_number, pp.primary_participant DESC;

-- Shot chart data
CREATE MATERIALIZED VIEW shot_chart_data AS
SELECT 
    p.play_id,
    p.event_id,
    p.sequence_number,
    e.name AS event_name,
    e.start_date AS game_date,
    p.period,
    p.clock_time,
    a.full_name AS athlete_name,
    t.display_name AS team_name,
    p.shot_type,
    p.shot_made,
    p.score_value,
    p.coordinates_x,
    p.coordinates_y,
    SQRT(POW(p.coordinates_x, 2) + POW(p.coordinates_y, 2)) AS shot_distance,
    CASE
        WHEN p.coordinates_x BETWEEN -8 AND 8 AND p.coordinates_y BETWEEN 0 AND 19 THEN 'paint'
        WHEN SQRT(POW(p.coordinates_x, 2) + POW(p.coordinates_y, 2)) > 23.75 THEN 'three_point'
        ELSE 'mid_range'
    END AS shot_zone
FROM 
    Plays p
JOIN Events e ON p.event_id = e.event_id
JOIN PlayParticipants pp ON p.play_id = pp.play_id AND pp.role = 'shooter'
JOIN Athletes a ON pp.athlete_id = a.athlete_id
JOIN Teams t ON pp.team_id = t.team_id
WHERE 
    p.play_type_id IN (SELECT play_type_id FROM PlayTypes WHERE category = 'Shot')
ORDER BY 
    e.start_date, p.event_id, p.sequence_number;
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
| Play-by-Play | Implemented | Complete data structure with plays, participants, streaks and game flow |
| Box Scores | Added | Period-by-period scoring |
| Shot Charts | Added | Spatial shot data |
| Indexing | Added | DuckDB-specific indexing strategy defined |
| Ranking Systems | Added | Complete poll data structures |
| Awards & Honors | Added | Award categories and recipients |
| Broadcast Data | Added | Networks, markets, and broadcast details |

## Next Steps

### High Priority
1. **Add Officials/Referees Data Structure**
   - Create `Officials` table for referee information
   - Develop `EventOfficials` join table to link officials to games
   - Track historical officiating assignments and tendencies
   - Include role designations (head referee, etc.)

2. **Create Shot Charts and Spatial Analysis Structure**
   - Implement `ShotCharts` table with x/y coordinates
   - Add made/missed, distance, and points metadata
   - Include assist tracking for made baskets
   - Support spatial analysis queries

3. **Add Coaching Staff Data Structure**
   - Create `Coaches` dimension table with biographical information
   - Implement `TeamCoaches` relationship table with roles
   - Track coaching history and tenure
   - Include assistant coaches and staff roles

4. **Implement Betting Odds Structure**
   - Create tables for pre-game odds and lines
   - Track line movements over time
   - Include over/under and point spread data
   - Store odds provider information

### Medium Priority
5. **Develop Injury Tracking System**
   - Implement `Injuries` table with injury types and severity
   - Create `AthleteInjuries` table linking injuries to players
   - Track timelines, return dates, and status updates
   - Support historical injury analysis

6. **Implement Media Resource Tracking**
   - Create `MediaResources` table for logos, headshots, and venue images
   - Develop metadata for image dimensions and types
   - Track resource currency and updates
   - Include alt text and accessibility information

7. **Create Event Status History Tracking**
   - Implement temporal tracking of game status changes
   - Store clock, period, and status type information
   - Support timeline reconstruction
   - Include status change reasons

8. **Develop Reference Resolution Strategy**
    - Create approach for handling ESPN API's `$ref` fields
    - Implement reference caching mechanisms
    - Establish rules for reference staleness
    - Develop hierarchical resolution patterns

9. **Add News/Headlines/Notes Structure**
    - Create `News` table for articles and stories
    - Implement `EventHeadlines` for game-specific headlines
    - Track news sources and publication timestamps
    - Include relevance scores and categorization

10. **Ensure Consistent UTC Time Handling**
    - Standardize all temporal fields in UTC
    - Implement display time zone conversion
    - Create timestamp precision standards
    - Ensure date range query optimization

11. **Add Venue Capacity and Attendance Tracking**
    - Enhance `Venues` table with detailed capacity information
    - Add historical attendance tracking
    - Calculate attendance percentages
    - Include venue configuration information

12. **Establish Data Refresh Patterns**
    - Define incremental vs. full refresh strategies
    - Create data freshness metadata tracking
    - Implement priority-based update scheduling
    - Develop change detection mechanisms

13. **Create View Layer for Analytics**
    - Develop materialized views for common analytical patterns
    - Create data marts for specific analysis domains
    - Implement role-based access controls
    - Support visualization-friendly data structures

14. **Consider Partitioning Strategy**
    - Implement partitioning for large historical tables
    - Develop season-based partitioning scheme
    - Create hot/cold data management strategy
    - Optimize partition sizes for DuckDB

15. **Develop Approach for Derived Statistics**
    - Create calculation methodology documentation
    - Implement derived statistic triggers or procedures
    - Establish calculation provenance tracking
    - Develop statistic version control

16. **Create Test Queries for Analytics**
    - Develop benchmark query suite
    - Create validation test cases
    - Implement performance testing framework
    - Document common analytical patterns

17. **Implement Team Conference Alignment History**
    - Track historical conference membership
    - Create effective dating for alignment changes
    - Support realignment analysis
    - Include division classification history

18. **Develop Memory Management Routines**
    - Create scheduled database detach/reattach jobs
    - Implement index usage monitoring
    - Develop memory pressure handling
    - Create index rebuild optimization

19. **Add Alternate Names and Identifiers**
    - Create historical name tracking for teams/venues
    - Implement external ID mapping system
    - Support alias resolution
    - Track brand changes and renamings

20. **Develop Game Highlight Structure**
    - Create `Highlights` table for key moments
    - Link highlights to plays and athletes
    - Include media references and timestamps
    - Support highlight categorization

21. **Implement Records and Milestones Tracking**
    - Track team and player records
    - Create milestone achievement history
    - Implement record-breaking detection
    - Support historical significance scoring

## Technical Considerations

1. Reference Resolution: Create a strategy to properly resolve ESPN API's `$ref` fields into normalized database entities.
2. Temporal Data Management: Enforce consistent UTC time formatting for all date/time fields to support historical analysis.
3. Memory Management: Implement regular maintenance routines to manage DuckDB's memory usage for indexes.
4. Data Synchronization: Design pipelines to handle real-time updates during active game periods.
5. Data Retention: Define policies for historical data retention and archiving strategies.
6. API Rate Limiting: Implement request throttling to respect ESPN's API rate limits.
7. Error Handling: Develop robust error handling for API unavailability or schema changes.

## Open Questions

- How should we handle very sparse statistical fields that only apply to certain athletes or game situations?
- What is the best storage format for historical time-series analysis of player development?
- Should we implement a separate star schema for analytical queries on top of this normalized structure?
- How frequently should aggregate statistics be recalculated?
- What is the optimal backup strategy for a DuckDB-based analytics database?
- How should we handle ESPN API schema changes or deprecations?
- What is the proper approach for handling team identity changes (rebrands, relocations) over time?

## Tournament Structure

### Tournaments
Core tournament information.

| Column | Type | Description | Example |
|--------|------|-------------|---------|
| tournament_id | INTEGER | Primary key (auto-increment) | 1 |
| espn_id | VARCHAR | ESPN's identifier | 3136952 |
| uid | VARCHAR | Universal identifier | s:40~l:41~t:3136952 |
| name | VARCHAR | Tournament name | NCAA Men's Basketball Tournament |
| short_name | VARCHAR | Short display name | NCAA Tournament |
| abbreviation | VARCHAR | Tournament abbreviation | NCAAT |
| season_id | INTEGER | Foreign key to seasons | 1 |
| start_date | TIMESTAMP | Tournament start date | 2025-03-18T00:00Z |
| end_date | TIMESTAMP | Tournament end date | 2025-04-07T00:00Z |
| num_teams | INTEGER | Number of participating teams | 68 |
| type | VARCHAR | Tournament type | championship |
| level | VARCHAR | Tournament level | national |
| slug | VARCHAR | URL identifier | ncaa-tournament |
| status | VARCHAR | Current status | scheduled |
| format | VARCHAR | Tournament format | single-elimination |
| is_postseason | BOOLEAN | If postseason tournament | true |
| is_conference | BOOLEAN | If conference tournament | false |
| logo_url | VARCHAR | URL to tournament logo | https://a.espncdn.com/i/tournaments/ncaa/ncaam.png |

**Indexing:**
- Order by `season_id, start_date` for chronological queries
- Create explicit index for API integration:
  ```sql
  CREATE INDEX idx_tournaments_espn_id ON Tournaments(espn_id);
  ```

### TournamentBrackets
Defines bracket structures within tournaments.

| Column | Type | Description | Example |
|--------|------|-------------|---------|
| bracket_id | INTEGER | Primary key (auto-increment) | 1 |
| tournament_id | INTEGER | Foreign key to tournaments | 1 |
| espn_id | VARCHAR | ESPN's identifier | 108 |
| name | VARCHAR | Bracket name | East |
| display_name | VARCHAR | Display name | East Region |
| short_name | VARCHAR | Short display name | East |
| abbreviation | VARCHAR | Bracket abbreviation | E |
| order_num | INTEGER | Display/priority order | 1 |
| num_teams | INTEGER | Number of teams in bracket | 16 |
| round_count | INTEGER | Number of rounds in bracket | 4 |
| location | VARCHAR | Regional location | New York, NY |
| venue_id | INTEGER | Foreign key to venues (regional site) | 24 |
| is_active | BOOLEAN | If bracket is active | true |

**Indexing:**
- Order by `tournament_id, order_num` to maintain bracket order
- No explicit indexes needed due to small table size

### TournamentRounds
Defines rounds within tournaments.

| Column | Type | Description | Example |
|--------|------|-------------|---------|
| round_id | INTEGER | Primary key (auto-increment) | 1 |
| tournament_id | INTEGER | Foreign key to tournaments | 1 |
| bracket_id | INTEGER | Foreign key to brackets (NULL for cross-bracket rounds) | 1 |
| espn_id | VARCHAR | ESPN's identifier | 1156 |
| name | VARCHAR | Round name | Sweet 16 |
| short_name | VARCHAR | Short display name | Sweet 16 |
| abbreviation | VARCHAR | Round abbreviation | S16 |
| round_number | INTEGER | Sequential round number | 3 |
| start_date | TIMESTAMP | Round start date | 2025-03-27T00:00Z |
| end_date | TIMESTAMP | Round end date | 2025-03-28T23:59Z |
| num_games | INTEGER | Number of games in round | 8 |
| teams_eliminated | INTEGER | Teams eliminated in round | 8 |
| teams_advanced | INTEGER | Teams advancing from round | 8 |
| is_play_in | BOOLEAN | If play-in/first four | false |
| is_championship | BOOLEAN | If championship round | false |
| round_sequence | INTEGER | Overall tournament sequence | 5 |

**Indexing:**
- Order by `tournament_id, round_sequence` for chronological round order
- No explicit indexes needed due to moderate table size

### TournamentSeeds
Defines seed positions within tournament brackets.

| Column | Type | Description | Example |
|--------|------|-------------|---------|
| seed_id | INTEGER | Primary key (auto-increment) | 1 |
| tournament_id | INTEGER | Foreign key to tournaments | 1 |
| bracket_id | INTEGER | Foreign key to brackets | 1 |
| seed_number | INTEGER | Seed position (1-16) | 1 |
| display_name | VARCHAR | Display representation | #1 |
| team_id | INTEGER | Foreign key to teams | 52 |
| team_entry_type | VARCHAR | Entry type | automatic |
| team_entry_reason | VARCHAR | Reason for entry | conference-champion |
| is_protected | BOOLEAN | If protected seed | true |
| original_seed_id | INTEGER | Original seed ID if play-in winner | NULL |
| initial_matchup_round_id | INTEGER | First round for this seed | 2 |
| is_play_in | BOOLEAN | If involved in play-in game | false |

**Indexing:**
- Order by `tournament_id, bracket_id, seed_number` for efficient lookups
- Create index for team participation:
  ```sql
  CREATE INDEX idx_tournament_seeds_team ON TournamentSeeds(team_id, tournament_id);
  ```

### TournamentMatchups
Defines tournament game matchups.

| Column | Type | Description | Example |
|--------|------|-------------|---------|
| matchup_id | INTEGER | Primary key (auto-increment) | 1 |
| tournament_id | INTEGER | Foreign key to tournaments | 1 |
| bracket_id | INTEGER | Foreign key to brackets | 1 |
| round_id | INTEGER | Foreign key to rounds | 1 |
| event_id | INTEGER | Foreign key to events | 401526511 |
| matchup_number | INTEGER | Sequence within round | 1 |
| home_seed_id | INTEGER | Foreign key to higher seed | 1 |
| away_seed_id | INTEGER | Foreign key to lower seed | 16 |
| next_matchup_id | INTEGER | Next matchup for winner | 33 |
| next_matchup_position | VARCHAR | Position in next matchup | home |
| pod_number | INTEGER | Group/pod identifier | 1 |
| status | VARCHAR | Matchup status | scheduled |
| order_num | INTEGER | Display order | 1 |

**Indexing:**
- Order by `tournament_id, round_id, matchup_number` for bracket display
- Create index for event linkage:
  ```sql
  CREATE INDEX idx_tournament_matchups_event ON TournamentMatchups(event_id);
  ```

### TournamentAdvancement
Tracks team progression through tournament.

| Column | Type | Description | Example |
|--------|------|-------------|---------|
| advancement_id | INTEGER | Primary key (auto-increment) | 1 |
| tournament_id | INTEGER | Foreign key to tournaments | 1 |
| team_id | INTEGER | Foreign key to teams | 52 |
| seed_id | INTEGER | Foreign key to seeds | 1 |
| from_round_id | INTEGER | Round advanced from | 2 |
| to_round_id | INTEGER | Round advanced to | 3 |
| from_matchup_id | INTEGER | Matchup advanced from | 1 |
| to_matchup_id | INTEGER | Matchup advanced to | 33 |
| advancement_date | TIMESTAMP | When advancement occurred | 2025-03-22T23:15Z |
| is_upset | BOOLEAN | If considered an upset | false |
| upset_differential | INTEGER | Seed differential if upset | NULL |
| advancement_type | VARCHAR | How team advanced | win |
| eliminated | BOOLEAN | If team was eliminated | false |
| elimination_round_id | INTEGER | Round where eliminated | NULL |

**Indexing:**
- Order by `tournament_id, team_id, from_round_id` for team progression
- Create index for upset analysis:
  ```sql
  CREATE INDEX idx_tournament_advancement_upset ON TournamentAdvancement(tournament_id) WHERE is_upset = true;
  ```

### TournamentSelectionCriteria
Tracks criteria used for tournament selection.

| Column | Type | Description | Example |
|--------|------|-------------|---------|
| criteria_id | INTEGER | Primary key (auto-increment) | 1 |
| tournament_id | INTEGER | Foreign key to tournaments | 1 |
| season_id | INTEGER | Foreign key to seasons | 1 |
| criteria_name | VARCHAR | Name of criterion | NET Ranking |
| criteria_type | VARCHAR | Type of criterion | metric |
| weight | DECIMAL | Relative importance | 0.7 |
| description | VARCHAR | Description of criterion | NCAA Evaluation Tool ranking |
| threshold | VARCHAR | Typical threshold value | Top 40 |
| is_public | BOOLEAN | If publicly disclosed | true |

**Indexing:**
- Order by `tournament_id, weight DESC` to prioritize important criteria
- No explicit indexes needed due to small table size

### TeamTournamentSelectionMetrics
Metrics related to team selection for tournaments.

| Column | Type | Description | Example |
|--------|------|-------------|---------|
| selection_metric_id | INTEGER | Primary key (auto-increment) | 1 |
| tournament_id | INTEGER | Foreign key to tournaments | 1 |
| team_id | INTEGER | Foreign key to teams | 52 |
| criteria_id | INTEGER | Foreign key to selection criteria | 1 |
| metric_value | VARCHAR | Value of metric | 5 |
| metric_rank | INTEGER | Rank within criterion | 5 |
| is_strength | BOOLEAN | If considered a strength | true |
| is_weakness | BOOLEAN | If considered a weakness | false |
| selection_impact | VARCHAR | Impact on selection | high |
| selection_notes | VARCHAR | Notes on selection impact | Strong Quad 1 record |

**Indexing:**
- Order by `tournament_id, team_id` to group metrics by team
- Create index for significant selection factors:
  ```sql
  CREATE INDEX idx_selection_metrics_impact ON TeamTournamentSelectionMetrics(tournament_id, selection_impact) WHERE selection_impact = 'high';
  ```

### TournamentBids
Tracks automatic and at-large bids.

| Column | Type | Description | Example |
|--------|------|-------------|---------|
| bid_id | INTEGER | Primary key (auto-increment) | 1 |
| tournament_id | INTEGER | Foreign key to tournaments | 1 |
| bid_type | VARCHAR | Type of bid | automatic |
| group_id | INTEGER | Foreign key to groups/conferences | 1 |
| team_id | INTEGER | Foreign key to teams | 52 |
| selection_date | TIMESTAMP | When bid was awarded | 2025-03-15T23:00Z |
| bid_reason | VARCHAR | Reason for bid | conference-champion |
| selection_rank | INTEGER | Selection committee ranking | 1 |
| last_team_in | BOOLEAN | If last team selected | false |
| first_team_out | BOOLEAN | If first team not selected | false |
| selection_controversy | INTEGER | Level of selection controversy | 0 |

**Indexing:**
- Order by `tournament_id, bid_type, selection_rank`
- Create index for bubble teams:
  ```sql
  CREATE INDEX idx_tournament_bids_bubble ON TournamentBids(tournament_id) WHERE last_team_in = true OR first_team_out = true;
  ```

### BracketologyProjections
Historical projections from bracketologists.

| Column | Type | Description | Example |
|--------|------|-------------|---------|
| projection_id | INTEGER | Primary key (auto-increment) | 1 |
| tournament_id | INTEGER | Foreign key to tournaments | 1 |
| team_id | INTEGER | Foreign key to teams | 52 |
| projection_date | TIMESTAMP | When projection was made | 2025-03-05T12:00Z |
| analyst_name | VARCHAR | Bracketologist name | Joe Lunardi |
| publication | VARCHAR | Publication source | ESPN |
| projected_seed | VARCHAR | Projected seed | 2 |
| projected_region | VARCHAR | Projected region | East |
| selection_likelihood | DECIMAL | Likelihood of selection | 0.95 |
| projected_status | VARCHAR | Status in projection | lock |
| actual_selected | BOOLEAN | If actually selected | true |
| actual_seed | VARCHAR | Actual seed received | 1 |
| prediction_accuracy | VARCHAR | Accuracy of prediction | accurate |

**Indexing:**
- Order by `tournament_id, projection_date DESC` for latest projections
- Create index for team tracking:
  ```sql
  CREATE INDEX idx_bracketology_projections_team ON BracketologyProjections(team_id, projection_date DESC);
  ``` 