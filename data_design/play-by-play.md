# Play-by-Play Data Structure

This section documents the tables used for storing play-by-play data in our ESPN API data structure. These tables track detailed event timelines, providing play-by-play information for each game to enable advanced analysis of game flow and player performance in specific situations.

## PlayTypes

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

## Plays

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

## PlayParticipants

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

## PlayStreaks

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

## GameFlowMetrics

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

## GameSegments

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

## PlaysTagMap

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
``` 