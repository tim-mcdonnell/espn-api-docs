# Statistical Data Structure

This section documents the tables used for storing and organizing statistical data in our ESPN API data structure. These tables provide a flexible foundation for capturing various basketball statistics while maintaining relationships to the core dimensions.

## StatCategories

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

## StatTypes

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

## StatScopes

Defines the scope or context of statistics.

| Column | Type | Description | Example |
|--------|------|-------------|---------|
| stat_scope_id | INTEGER | Primary key (auto-increment) | 1 |
| name | VARCHAR | Scope name | game |
| display_name | VARCHAR | Display name | Game |
| description | VARCHAR | Description | Single game statistics |
| duration_type | VARCHAR | Type of duration | event |

**Indexing:** No explicit indexes needed due to small table size.

## TeamStatisticsFact

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

## AthleteStatisticsFact

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

## LeagueAverages

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

## StatisticalLeaders

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

## Materialized Views for Statistical Analysis

For efficient analysis of statistical data, these materialized views are recommended:

```sql
-- Player season averages by season
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
``` 