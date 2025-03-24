# Shot Charts and Spatial Analysis Structure

This section documents the tables used for storing and analyzing spatial shot data in our ESPN API data structure. These tables enable detailed shot tracking, location analysis, and visualization of shooting patterns.

## ShotCharts

Stores individual shot attempts with spatial information.

| Column | Type | Description | Example |
|--------|------|-------------|---------|
| shot_id | INTEGER | Primary key (auto-increment) | 1 |
| event_id | INTEGER | Foreign key to events | 1 |
| play_id | INTEGER | Foreign key to plays | 105 |
| athlete_id | INTEGER | Foreign key to athletes | 1 |
| team_id | INTEGER | Foreign key to teams | 1 |
| period | INTEGER | Game period (1=1st half, 2=2nd half) | 1 |
| clock_time | VARCHAR | Game clock at time of shot | 19:36 |
| clock_seconds_remaining | INTEGER | Seconds remaining in period | 1176 |
| coordinates_x | DECIMAL | X coordinate of shot on court | 23.5 |
| coordinates_y | DECIMAL | Y coordinate of shot on court | 7.2 |
| shot_type | VARCHAR | Type of shot | Three Point Jumper |
| shot_distance | DECIMAL | Distance from basket in feet | 22.5 |
| shot_value | INTEGER | Point value of shot if made | 3 |
| made | BOOLEAN | If shot was made | false |
| blocked | BOOLEAN | If shot was blocked | false |
| assisted | BOOLEAN | If shot was assisted | false |
| assist_athlete_id | INTEGER | Foreign key to assisting athlete | NULL |
| fastbreak | BOOLEAN | If shot was in transition | false |
| second_chance | BOOLEAN | If shot was a second chance | false |
| points | INTEGER | Points scored on the shot | 0 |
| points_off_turnover | BOOLEAN | If points were from turnover | false |
| in_paint | BOOLEAN | If shot was in the paint | false |
| event_sequence | INTEGER | Sequence number within game | 23 |
| shot_zone_id | INTEGER | Foreign key to court zones | 5 |
| defender_athlete_id | INTEGER | Foreign key to primary defender | NULL |
| defender_distance | DECIMAL | Distance of closest defender in feet | 3.2 |
| time_on_shot_clock | INTEGER | Shot clock time remaining | 12 |
| possession_prior | INTEGER | Team with possession before | 1 |
| possession_after | INTEGER | Team with possession after | 2 |
| home_score_before | INTEGER | Home score before shot | 15 |
| away_score_before | INTEGER | Away score before shot | 18 |
| shot_quality | DECIMAL | Shot quality metric (0-1) | 0.53 |
| expected_points | DECIMAL | Expected value of shot | 0.89 |
| shot_outcome_description | VARCHAR | Detailed outcome description | Missed, Rim |

**Indexing:**
- Order by `event_id, event_sequence` for chronological analysis
- Create composite indexes for common analysis patterns:
  ```sql
  CREATE INDEX idx_shot_charts_athlete ON ShotCharts(athlete_id, event_id);
  CREATE INDEX idx_shot_charts_team ON ShotCharts(team_id, event_id);
  CREATE INDEX idx_shot_charts_zone ON ShotCharts(shot_zone_id, made);
  CREATE INDEX idx_shot_charts_spatial ON ShotCharts(coordinates_x, coordinates_y);
  ```

## CourtZones

Defines standardized court regions for shot analysis.

| Column | Type | Description | Example |
|--------|------|-------------|---------|
| zone_id | INTEGER | Primary key (auto-increment) | 1 |
| name | VARCHAR | Zone name | Left Corner 3 |
| type | VARCHAR | Zone type | three_point |
| description | VARCHAR | Zone description | Left corner three-point area |
| short_name | VARCHAR | Abbreviated name | LC3 |
| display_order | INTEGER | Display ordering | 1 |
| min_x | DECIMAL | Minimum X coordinate | -25.0 |
| max_x | DECIMAL | Maximum X coordinate | -22.0 |
| min_y | DECIMAL | Minimum Y coordinate | 0.0 |
| max_y | DECIMAL | Maximum Y coordinate | 5.0 |
| is_paint | BOOLEAN | If zone is in the paint | false |
| is_restricted_area | BOOLEAN | If in restricted area | false |
| is_three_point | BOOLEAN | If zone is three-point area | true |
| is_midrange | BOOLEAN | If zone is midrange area | false |
| average_expected_value | DECIMAL | League average points per shot | 1.05 |
| geometric_definition | VARCHAR | Geometric definition of zone | POLYGON((-25 0, -22 0, -22 5, -25 5, -25 0)) |

**Indexing:**
- No explicit indexes needed due to small table size
- Order by `display_order` to maintain intended display sequence

## ShotZoneEfficiency

Aggregated shooting efficiency by zone, team, and athlete.

| Column | Type | Description | Example |
|--------|------|-------------|---------|
| efficiency_id | INTEGER | Primary key (auto-increment) | 1 |
| zone_id | INTEGER | Foreign key to court zones | 1 |
| athlete_id | INTEGER | Foreign key to athletes (NULL for team) | 1 |
| team_id | INTEGER | Foreign key to teams | 1 |
| season_id | INTEGER | Foreign key to seasons | 1 |
| event_id | INTEGER | Foreign key to events (NULL for season) | NULL |
| attempts | INTEGER | Number of shot attempts | 45 |
| makes | INTEGER | Number of made shots | 18 |
| points | INTEGER | Total points scored | 54 |
| efficiency | DECIMAL | Shooting percentage | 0.40 |
| points_per_shot | DECIMAL | Average points per shot | 1.2 |
| expected_makes | DECIMAL | Expected makes based on league avg | 15.3 |
| expected_points | DECIMAL | Expected points based on league avg | 45.9 |
| efficiency_vs_expected | DECIMAL | Actual vs expected efficiency | 0.06 |
| frequency | DECIMAL | Frequency of shots from this zone | 0.15 |
| last_updated | TIMESTAMP | When stats were last updated | 2024-03-15T16:30Z |

**Indexing:**
- Order by `zone_id, team_id, athlete_id` to optimize zone-based queries
- Create index for athlete-specific analysis:
  ```sql
  CREATE INDEX idx_shot_zone_efficiency_athlete ON ShotZoneEfficiency(athlete_id, season_id) WHERE athlete_id IS NOT NULL;
  ```
- Create index for team analysis:
  ```sql
  CREATE INDEX idx_shot_zone_efficiency_team ON ShotZoneEfficiency(team_id, season_id);
  ```

## ShotLocations

Stores standardized shot location classifications.

| Column | Type | Description | Example |
|--------|------|-------------|---------|
| location_id | INTEGER | Primary key (auto-increment) | 1 |
| name | VARCHAR | Location name | Paint |
| short_name | VARCHAR | Abbreviated name | PT |
| description | VARCHAR | Location description | Shots within the paint area |
| is_restricted_area | BOOLEAN | If restricted area | false |
| display_order | INTEGER | Order for display purposes | 1 |
| color_hex | VARCHAR | Hexadecimal color for visualization | #FF5733 |

**Indexing:**
- No explicit indexes needed due to small table size
- Order by `display_order` to maintain display sequence

## ShotTendencies

Player and team shot tendency metrics.

| Column | Type | Description | Example |
|--------|------|-------------|---------|
| tendency_id | INTEGER | Primary key (auto-increment) | 1 |
| athlete_id | INTEGER | Foreign key to athletes (NULL for team) | 1 |
| team_id | INTEGER | Foreign key to teams | 1 |
| season_id | INTEGER | Foreign key to seasons | 1 |
| tendency_type | VARCHAR | Type of tendency | shot_selection |
| tendency_value | DECIMAL | Value of tendency metric | 0.65 |
| tendency_description | VARCHAR | Description of tendency | Strong preference for right side |
| percentile | DECIMAL | Percentile rank | 0.85 |
| tendency_category | VARCHAR | Category of tendency | court_region |
| sample_size | INTEGER | Number of shots in sample | 250 |
| versus_average | DECIMAL | Comparison to average | 0.12 |
| last_updated | TIMESTAMP | When updated | 2024-03-15T16:30Z |

**Indexing:**
- Order by `athlete_id, team_id, season_id, tendency_type` for efficient tendency lookups
- Create index for outlier tendencies:
  ```sql
  CREATE INDEX idx_shot_tendencies_outliers ON ShotTendencies(tendency_type, ABS(versus_average) DESC);
  ```

## DefenderImpact

Measures defensive impact on shooting.

| Column | Type | Description | Example |
|--------|------|-------------|---------|
| impact_id | INTEGER | Primary key (auto-increment) | 1 |
| defender_athlete_id | INTEGER | Foreign key to defender | 1 |
| shooter_athlete_id | INTEGER | Foreign key to shooter | 2 |
| defender_team_id | INTEGER | Foreign key to defender's team | 1 |
| season_id | INTEGER | Foreign key to seasons | 1 |
| zone_id | INTEGER | Foreign key to court zones (NULL for all) | NULL |
| contests | INTEGER | Number of contested shots | 120 |
| field_goals_made | INTEGER | Field goals made when contesting | 42 |
| field_goals_attempted | INTEGER | Field goals attempted when contesting | 120 |
| field_goal_pct | DECIMAL | FG% when contesting | 0.35 |
| expected_fg_pct | DECIMAL | Expected FG% based on shot quality | 0.45 |
| impact_pct | DECIMAL | Impact on shooting percentage | -0.10 |
| points_saved | DECIMAL | Estimated points saved | 24.5 |
| defensive_rating | DECIMAL | Defensive impact rating | 92.5 |
| contest_rate | DECIMAL | Rate of contesting shots | 0.68 |
| last_updated | TIMESTAMP | When updated | 2024-03-15T16:30Z |

**Indexing:**
- Order by `defender_athlete_id, season_id` to optimize defender-specific queries
- Create index for defensive standouts:
  ```sql
  CREATE INDEX idx_defender_impact_leaders ON DefenderImpact(season_id, impact_pct) WHERE contests > 50;
  ```

## Materialized Views for Shot Analysis

To facilitate common analytics queries on shot data, these materialized views are recommended:

```sql
-- Player shot profile
CREATE MATERIALIZED VIEW player_shot_profile AS
SELECT 
    a.athlete_id,
    a.full_name,
    t.team_id,
    t.display_name AS team_name,
    s.season_id,
    s.display_name AS season,
    COUNT(*) AS total_shots,
    SUM(CASE WHEN sc.made THEN 1 ELSE 0 END) AS made_shots,
    SUM(CASE WHEN sc.made THEN 1 ELSE 0 END)::FLOAT / COUNT(*) AS shot_pct,
    SUM(sc.points) AS total_points,
    SUM(sc.points)::FLOAT / COUNT(*) AS points_per_shot,
    SUM(CASE WHEN sc.in_paint THEN 1 ELSE 0 END) AS paint_shots,
    SUM(CASE WHEN sc.shot_type LIKE '%Three Point%' THEN 1 ELSE 0 END) AS three_point_attempts,
    SUM(CASE WHEN sc.shot_type LIKE '%Three Point%' AND sc.made THEN 1 ELSE 0 END) AS three_point_makes,
    SUM(CASE WHEN sc.shot_type LIKE '%Three Point%' AND sc.made THEN 1 ELSE 0 END)::FLOAT / 
        NULLIF(SUM(CASE WHEN sc.shot_type LIKE '%Three Point%' THEN 1 ELSE 0 END), 0) AS three_point_pct,
    SUM(CASE WHEN sc.assisted THEN 1 ELSE 0 END)::FLOAT / 
        NULLIF(SUM(CASE WHEN sc.made THEN 1 ELSE 0 END), 0) AS assisted_rate,
    AVG(sc.shot_distance) AS avg_shot_distance,
    SUM(CASE WHEN sc.shot_quality > 0.6 THEN 1 ELSE 0 END)::FLOAT / COUNT(*) AS quality_shot_rate,
    SUM(CASE WHEN sc.made AND sc.shot_quality < 0.4 THEN 1 ELSE 0 END) AS difficult_makes,
    COUNT(DISTINCT cz.zone_id) AS zones_used,
    MODE() WITHIN GROUP (ORDER BY cz.name) AS favorite_zone
FROM 
    ShotCharts sc
JOIN Athletes a ON sc.athlete_id = a.athlete_id
JOIN Teams t ON sc.team_id = t.team_id
JOIN Events e ON sc.event_id = e.event_id
JOIN Seasons s ON e.season_id = s.season_id
JOIN CourtZones cz ON sc.shot_zone_id = cz.zone_id
GROUP BY 
    a.athlete_id, a.full_name, t.team_id, t.display_name, s.season_id, s.display_name
HAVING 
    COUNT(*) >= 50
ORDER BY 
    s.season_id DESC, total_shots DESC;

-- Team shooting heatmap data
CREATE MATERIALIZED VIEW team_shooting_heatmap AS
SELECT 
    t.team_id,
    t.display_name AS team_name,
    s.season_id,
    s.display_name AS season,
    FLOOR(sc.coordinates_x/2)*2 AS x_bin,
    FLOOR(sc.coordinates_y/2)*2 AS y_bin,
    COUNT(*) AS attempts,
    SUM(CASE WHEN sc.made THEN 1 ELSE 0 END) AS makes,
    SUM(CASE WHEN sc.made THEN 1 ELSE 0 END)::FLOAT / COUNT(*) AS fg_pct,
    SUM(sc.points) AS points,
    SUM(sc.points)::FLOAT / COUNT(*) AS points_per_shot,
    AVG(sc.expected_points) AS expected_points_per_shot,
    SUM(sc.points)::FLOAT / NULLIF(SUM(sc.expected_points * COUNT(*)), 0) AS efficiency_vs_expected
FROM 
    ShotCharts sc
JOIN Teams t ON sc.team_id = t.team_id
JOIN Events e ON sc.event_id = e.event_id
JOIN Seasons s ON e.season_id = s.season_id
GROUP BY 
    t.team_id, t.display_name, s.season_id, s.display_name, 
    FLOOR(sc.coordinates_x/2)*2, FLOOR(sc.coordinates_y/2)*2
HAVING 
    COUNT(*) >= 5
ORDER BY 
    team_id, season_id, attempts DESC;
```

## Technical Implementation Considerations

1. **Spatial Indexing Limitations**: DuckDB does not natively support spatial indexes like PostGIS. Instead, we'll use binned coordinates and zone-based partitioning for efficient spatial lookups.

2. **Coordinate System**: Standardize on a consistent coordinate system where:
   - (0,0) represents the center of the basket
   - Positive X extends to the right of the basket
   - Positive Y extends away from the basket (toward half-court)
   - All coordinates are normalized to a single basket orientation

3. **Efficient Storage**: Since shot chart data can grow large, consider:
   - Partitioning by season
   - Periodically aggregating older games into statistical summaries
   - Using appropriate compression settings in DuckDB

4. **Data Loading Pipeline**:
   - Extract shot coordinates from ESPN's play-by-play data
   - Apply coordinate system transformations
   - Classify shots into standardized zones
   - Calculate derived metrics (expected points, shot quality) 