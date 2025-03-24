# Team Performance Metrics Framework

This section documents the tables used for storing advanced team performance metrics. The framework provides a comprehensive structure for calculating, storing, and analyzing team efficiency ratings, tempo-free statistics, and comparison metrics.

## TeamPerformanceMetrics

Stores core advanced team performance metrics.

| Column | Type | Description | Example |
|--------|------|-------------|---------|
| metric_id | INTEGER | Primary key (auto-increment) | 1 |
| team_id | INTEGER | Foreign key to teams | 2509 |
| season_id | INTEGER | Foreign key to seasons | 2024 |
| event_id | INTEGER | Foreign key to events (NULL for season metrics) | 401524661 |
| metric_date | TIMESTAMP | Date of metric calculation | 2024-03-12 00:00:00 |
| offensive_rating | DECIMAL | Points scored per 100 possessions | 112.4 |
| defensive_rating | DECIMAL | Points allowed per 100 possessions | 98.6 |
| net_rating | DECIMAL | Net point differential per 100 possessions | 13.8 |
| pace | DECIMAL | Possessions per 40 minutes | 68.5 |
| effective_fg_pct | DECIMAL | Effective field goal percentage | 54.2 |
| true_shooting_pct | DECIMAL | True shooting percentage | 58.7 |
| offensive_rebound_pct | DECIMAL | Percentage of available offensive rebounds secured | 32.5 |
| defensive_rebound_pct | DECIMAL | Percentage of available defensive rebounds secured | 74.8 |
| total_rebound_pct | DECIMAL | Percentage of total available rebounds secured | 53.2 |
| turnover_pct | DECIMAL | Turnovers per 100 possessions | 16.4 |
| assist_pct | DECIMAL | Percentage of made field goals that are assisted | 62.1 |
| block_pct | DECIMAL | Percentage of opponent 2-point attempts blocked | 8.6 |
| steal_pct | DECIMAL | Steals per defensive possession | 9.3 |
| free_throw_rate | DECIMAL | Free throw attempts per field goal attempt | 0.32 |
| created_at | TIMESTAMP | Record creation timestamp | 2024-03-12 00:00:00 |
| updated_at | TIMESTAMP | Record update timestamp | 2024-03-12 00:00:00 |

**Indexing:**
```sql
CREATE INDEX idx_team_metrics_team ON TeamPerformanceMetrics(team_id, season_id);
CREATE INDEX idx_team_metrics_event ON TeamPerformanceMetrics(event_id) WHERE event_id IS NOT NULL;
CREATE INDEX idx_team_metrics_date ON TeamPerformanceMetrics(metric_date);
```

## TeamPossessionData

Stores detailed possession-based statistics for teams.

| Column | Type | Description | Example |
|--------|------|-------------|---------|
| possession_id | INTEGER | Primary key (auto-increment) | 1 |
| team_id | INTEGER | Foreign key to teams | 2509 |
| season_id | INTEGER | Foreign key to seasons | 2024 |
| event_id | INTEGER | Foreign key to events (NULL for season aggregates) | 401524661 |
| total_possessions | INTEGER | Total possessions in the game/season | 65 |
| points_per_possession | DECIMAL | Points scored per possession | 1.12 |
| points_allowed_per_possession | DECIMAL | Points allowed per possession | 0.98 |
| avg_possession_time | DECIMAL | Average seconds per possession | 16.8 |
| avg_possession_distance | DECIMAL | Average distance (feet) ball travels per possession | 52.4 |
| avg_passes_per_possession | DECIMAL | Average passes per possession | 3.2 |
| avg_dribbles_per_possession | DECIMAL | Average dribbles per possession | 5.8 |
| fast_break_possessions | INTEGER | Number of fast break possessions | 12 |
| second_chance_possessions | INTEGER | Number of second chance possessions | 8 |
| points_off_turnovers | INTEGER | Points scored off turnovers | 14 |
| paint_possessions | INTEGER | Possessions with shot attempts in the paint | 25 |
| half_court_possessions | INTEGER | Number of half-court possessions | 53 |
| created_at | TIMESTAMP | Record creation timestamp | 2024-03-12 00:00:00 |
| updated_at | TIMESTAMP | Record update timestamp | 2024-03-12 00:00:00 |

**Indexing:**
```sql
CREATE INDEX idx_team_possession_team ON TeamPossessionData(team_id, season_id);
CREATE INDEX idx_team_possession_event ON TeamPossessionData(event_id) WHERE event_id IS NOT NULL;
```

## TeamShootingDistribution

Stores detailed shooting distribution data for teams.

| Column | Type | Description | Example |
|--------|------|-------------|---------|
| distribution_id | INTEGER | Primary key (auto-increment) | 1 |
| team_id | INTEGER | Foreign key to teams | 2509 |
| season_id | INTEGER | Foreign key to seasons | 2024 |
| event_id | INTEGER | Foreign key to events (NULL for season aggregates) | 401524661 |
| rim_attempts | INTEGER | Shot attempts at the rim | 24 |
| rim_made | INTEGER | Shots made at the rim | 15 |
| rim_pct | DECIMAL | Shooting percentage at the rim | 62.5 |
| paint_non_rim_attempts | INTEGER | Non-rim shot attempts in the paint | 12 |
| paint_non_rim_made | INTEGER | Non-rim shots made in the paint | 6 |
| paint_non_rim_pct | DECIMAL | Non-rim shooting percentage in the paint | 50.0 |
| mid_range_attempts | INTEGER | Mid-range shot attempts | 15 |
| mid_range_made | INTEGER | Mid-range shots made | 6 |
| mid_range_pct | DECIMAL | Mid-range shooting percentage | 40.0 |
| corner_3_attempts | INTEGER | Corner 3-point attempts | 8 |
| corner_3_made | INTEGER | Corner 3-point shots made | 3 |
| corner_3_pct | DECIMAL | Corner 3-point percentage | 37.5 |
| above_break_3_attempts | INTEGER | Above-the-break 3-point attempts | 18 |
| above_break_3_made | INTEGER | Above-the-break 3-point shots made | 6 |
| above_break_3_pct | DECIMAL | Above-the-break 3-point percentage | 33.3 |
| point_distribution_rim | DECIMAL | Percentage of points from rim shots | 30.0 |
| point_distribution_paint | DECIMAL | Percentage of points from paint shots | 12.0 |
| point_distribution_mid | DECIMAL | Percentage of points from mid-range | 12.0 |
| point_distribution_3pt | DECIMAL | Percentage of points from 3-pointers | 27.0 |
| point_distribution_ft | DECIMAL | Percentage of points from free throws | 19.0 |
| created_at | TIMESTAMP | Record creation timestamp | 2024-03-12 00:00:00 |
| updated_at | TIMESTAMP | Record update timestamp | 2024-03-12 00:00:00 |

**Indexing:**
```sql
CREATE INDEX idx_team_shooting_team ON TeamShootingDistribution(team_id, season_id);
CREATE INDEX idx_team_shooting_event ON TeamShootingDistribution(event_id) WHERE event_id IS NOT NULL;
```

## TeamPerformanceFactors

Stores the breakdown of performance factors contributing to wins and losses.

| Column | Type | Description | Example |
|--------|------|-------------|---------|
| factor_id | INTEGER | Primary key (auto-increment) | 1 |
| team_id | INTEGER | Foreign key to teams | 2509 |
| season_id | INTEGER | Foreign key to seasons | 2024 |
| event_id | INTEGER | Foreign key to events (NULL for season aggregates) | 401524661 |
| shooting_factor | DECIMAL | Contribution of shooting to performance (-1.0 to 1.0) | 0.65 |
| turnover_factor | DECIMAL | Contribution of turnovers to performance (-1.0 to 1.0) | -0.15 |
| rebounding_factor | DECIMAL | Contribution of rebounding to performance (-1.0 to 1.0) | 0.25 |
| free_throw_factor | DECIMAL | Contribution of free throws to performance (-1.0 to 1.0) | 0.10 |
| defense_factor | DECIMAL | Contribution of defense to performance (-1.0 to 1.0) | 0.45 |
| bench_factor | DECIMAL | Contribution of bench to performance (-1.0 to 1.0) | -0.05 |
| created_at | TIMESTAMP | Record creation timestamp | 2024-03-12 00:00:00 |
| updated_at | TIMESTAMP | Record update timestamp | 2024-03-12 00:00:00 |

**Indexing:**
```sql
CREATE INDEX idx_team_factors_team ON TeamPerformanceFactors(team_id, season_id);
CREATE INDEX idx_team_factors_event ON TeamPerformanceFactors(event_id) WHERE event_id IS NOT NULL;
```

## TeamStrengthMetrics

Stores strength of schedule and quality metrics.

| Column | Type | Description | Example |
|--------|------|-------------|---------|
| strength_id | INTEGER | Primary key (auto-increment) | 1 |
| team_id | INTEGER | Foreign key to teams | 2509 |
| season_id | INTEGER | Foreign key to seasons | 2024 |
| calculation_date | TIMESTAMP | Date of metric calculation | 2024-03-12 00:00:00 |
| strength_of_schedule | DECIMAL | Strength of schedule rating | 8.45 |
| sos_rank | INTEGER | Strength of schedule national ranking | 12 |
| strength_of_record | DECIMAL | Strength of record rating | 7.32 |
| sor_rank | INTEGER | Strength of record national ranking | 15 |
| quality_wins | INTEGER | Number of wins against top 50 teams | 5 |
| quality_win_pct | DECIMAL | Win percentage against top 50 teams | 62.5 |
| bad_losses | INTEGER | Number of losses against sub-100 teams | 1 |
| home_strength | DECIMAL | Performance strength at home | 9.23 |
| away_strength | DECIMAL | Performance strength away | 6.71 |
| neutral_strength | DECIMAL | Performance strength at neutral sites | 7.85 |
| projected_seed | INTEGER | Projected tournament seed | 4 |
| tournament_likelihood | DECIMAL | Probability of making tournament | 97.8 |
| created_at | TIMESTAMP | Record creation timestamp | 2024-03-12 00:00:00 |
| updated_at | TIMESTAMP | Record update timestamp | 2024-03-12 00:00:00 |

**Indexing:**
```sql
CREATE INDEX idx_team_strength_team ON TeamStrengthMetrics(team_id, season_id);
CREATE INDEX idx_team_strength_date ON TeamStrengthMetrics(calculation_date);
```

## TeamComparisonSnapshots

Stores head-to-head team comparison metrics.

| Column | Type | Description | Example |
|--------|------|-------------|---------|
| comparison_id | INTEGER | Primary key (auto-increment) | 1 |
| team_id | INTEGER | Foreign key to primary team | 2509 |
| comparison_team_id | INTEGER | Foreign key to comparison team | 158 |
| season_id | INTEGER | Foreign key to seasons | 2024 |
| event_id | INTEGER | Foreign key to events (NULL for non-event comparisons) | 401524661 |
| comparison_date | TIMESTAMP | Date of comparison generation | 2024-03-12 00:00:00 |
| offensive_advantage | DECIMAL | Offensive advantage for primary team (-10 to 10) | 2.5 |
| defensive_advantage | DECIMAL | Defensive advantage for primary team (-10 to 10) | -1.5 |
| rebounding_advantage | DECIMAL | Rebounding advantage for primary team (-10 to 10) | 3.0 |
| pace_advantage | DECIMAL | Pace advantage for primary team (-10 to 10) | 0.5 |
| experience_advantage | DECIMAL | Experience advantage for primary team (-10 to 10) | -2.0 |
| bench_advantage | DECIMAL | Bench advantage for primary team (-10 to 10) | 1.0 |
| home_court_advantage | DECIMAL | Home court advantage factor | 3.5 |
| win_probability | DECIMAL | Win probability for primary team | 65.7 |
| projected_score_team | DECIMAL | Projected score for primary team | 75.3 |
| projected_score_comparison | DECIMAL | Projected score for comparison team | 68.9 |
| created_at | TIMESTAMP | Record creation timestamp | 2024-03-12 00:00:00 |
| updated_at | TIMESTAMP | Record update timestamp | 2024-03-12 00:00:00 |

**Indexing:**
```sql
CREATE INDEX idx_team_comparison_teams ON TeamComparisonSnapshots(team_id, comparison_team_id, season_id);
CREATE INDEX idx_team_comparison_event ON TeamComparisonSnapshots(event_id) WHERE event_id IS NOT NULL;
CREATE INDEX idx_team_comparison_date ON TeamComparisonSnapshots(comparison_date);
```

## Entity Relationships

The team performance metrics framework integrates with the core data model through the following relationships:

- `TeamPerformanceMetrics` links to `Teams` via `team_id` to associate metrics with specific teams
- `TeamPerformanceMetrics` links to `Seasons` via `season_id` to associate metrics with specific seasons
- `TeamPerformanceMetrics` links to `Events` via `event_id` to associate metrics with specific games
- All metrics tables connect to the core `Teams` and `Seasons` tables
- `TeamComparisonSnapshots` additionally links to a second `Teams` record via `comparison_team_id`
- All metrics can be joined with existing statistics from `TeamStatisticsFact` for comprehensive analysis

## Usage Examples

### Retrieving Offensive and Defensive Efficiency Metrics

```sql
SELECT t.display_name AS team, 
       s.display_name AS season,
       tpm.offensive_rating, 
       tpm.defensive_rating,
       tpm.net_rating,
       tpm.pace
FROM TeamPerformanceMetrics tpm
JOIN Teams t ON tpm.team_id = t.team_id
JOIN Seasons s ON tpm.season_id = s.season_id
WHERE s.season_id = 2024
  AND tpm.event_id IS NULL
ORDER BY tpm.net_rating DESC
LIMIT 25;
```

### Analyzing Point Distribution and Shot Quality

```sql
SELECT t.display_name AS team,
       tsd.point_distribution_rim,
       tsd.point_distribution_3pt,
       tsd.point_distribution_ft,
       tsd.rim_pct,
       tsd.corner_3_pct
FROM TeamShootingDistribution tsd
JOIN Teams t ON tsd.team_id = t.team_id
WHERE tsd.season_id = 2024
  AND tsd.event_id IS NULL
ORDER BY t.display_name;
```

### Comparing Teams Head-to-Head

```sql
SELECT 
    t1.display_name AS team,
    t2.display_name AS opponent,
    tcs.offensive_advantage,
    tcs.defensive_advantage,
    tcs.rebounding_advantage,
    tcs.win_probability,
    tcs.projected_score_team,
    tcs.projected_score_comparison
FROM TeamComparisonSnapshots tcs
JOIN Teams t1 ON tcs.team_id = t1.team_id
JOIN Teams t2 ON tcs.comparison_team_id = t2.team_id
WHERE tcs.event_id = 401524661;
```

### Finding Teams with Most Efficient Offenses

```sql
WITH ranked_teams AS (
    SELECT 
        t.team_id,
        t.display_name AS team,
        g.abbreviation AS conference,
        tpm.offensive_rating,
        tpm.pace,
        tpm.effective_fg_pct,
        tpm.turnover_pct,
        RANK() OVER (ORDER BY tpm.offensive_rating DESC) AS offense_rank
    FROM TeamPerformanceMetrics tpm
    JOIN Teams t ON tpm.team_id = t.team_id
    JOIN TeamGroupMemberships tgm ON t.team_id = tgm.team_id AND tgm.season_id = tpm.season_id
    JOIN Groups g ON tgm.group_id = g.group_id
    WHERE tpm.season_id = 2024
      AND tpm.event_id IS NULL
      AND g.group_type = 'conference'
)
SELECT 
    team,
    conference,
    offensive_rating,
    pace,
    effective_fg_pct,
    turnover_pct,
    offense_rank
FROM ranked_teams
WHERE offense_rank <= 25
ORDER BY offense_rank;
```

### Analyzing Factors Contributing to Team Success

```sql
SELECT 
    t.display_name AS team,
    ROUND(AVG(CASE WHEN e.home_score > e.away_score AND e.home_team_id = t.team_id THEN 1
               WHEN e.away_score > e.home_score AND e.away_team_id = t.team_id THEN 1
               ELSE 0 END), 3) AS win_pct,
    ROUND(AVG(tpf.shooting_factor), 2) AS avg_shooting_factor,
    ROUND(AVG(tpf.rebounding_factor), 2) AS avg_rebounding_factor,
    ROUND(AVG(tpf.defense_factor), 2) AS avg_defense_factor
FROM TeamPerformanceFactors tpf
JOIN Teams t ON tpf.team_id = t.team_id
JOIN Events e ON tpf.event_id = e.event_id
WHERE tpf.season_id = 2024
  AND tpf.event_id IS NOT NULL
GROUP BY t.team_id, t.display_name
HAVING COUNT(*) >= 10
ORDER BY win_pct DESC
LIMIT 20;
``` 