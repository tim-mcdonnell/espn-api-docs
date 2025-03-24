# Officials/Referees Data Structure

This section documents the tables used for storing officials and referees data in our ESPN API data structure. These tables track officiating crews, their assignments, tendencies, and performance metrics.

## Officials

Information about game officials/referees.

| Column | Type | Description | Example |
|--------|------|-------------|---------|
| official_id | INTEGER | Primary key (auto-increment) | 1 |
| espn_id | VARCHAR | ESPN's identifier | 9347 |
| first_name | VARCHAR | First name | Roger |
| last_name | VARCHAR | Last name | Ayers |
| full_name | VARCHAR | Full name | Roger Ayers |
| display_name | VARCHAR | Display name | R. Ayers |
| active | BOOLEAN | If currently active | true |
| experience_years | INTEGER | Years of experience | 15 |
| primary_conference | INTEGER | Foreign key to groups/conferences | 1 |
| birth_date | TIMESTAMP | Birth date | 1966-02-25T00:00Z |
| bio | VARCHAR | Biographical information | Veteran official in his 15th season |
| headshot_url | VARCHAR | URL to headshot image | https://a.espncdn.com/i/officials/ncaam/roger_ayers.jpg |
| start_year | INTEGER | First year officiating | 2009 |
| certification_level | VARCHAR | Certification level | NCAA |
| badge_number | VARCHAR | Official's badge number | N2359 |
| status | VARCHAR | Current status | active |

**Indexing:**
- Create explicit index on `espn_id` for API integration:
  ```sql
  CREATE INDEX idx_officials_espn_id ON Officials(espn_id);
  ```
- Create index on `last_name, first_name` for name-based searches:
  ```sql
  CREATE INDEX idx_officials_name ON Officials(last_name, first_name);
  ```
- Order by `last_name, first_name` to optimize name-based lookups via zonemaps

## OfficialRoles

Defines the different roles officials can have.

| Column | Type | Description | Example |
|--------|------|-------------|---------|
| role_id | INTEGER | Primary key (auto-increment) | 1 |
| name | VARCHAR | Role name | Crew Chief |
| display_name | VARCHAR | Display name | Crew Chief |
| short_name | VARCHAR | Short name | CC |
| description | VARCHAR | Role description | Lead official who manages the officiating crew |
| position | VARCHAR | Physical position designation | Center |
| rank_order | INTEGER | Ordering/hierarchy | 1 |

**Indexing:**
- No explicit indexes needed due to small table size
- Order by `rank_order` to maintain hierarchy

## EventOfficials

Links officials to specific game events with their roles.

| Column | Type | Description | Example |
|--------|------|-------------|---------|
| event_official_id | INTEGER | Primary key (auto-increment) | 1 |
| event_id | INTEGER | Foreign key to events | 1 |
| official_id | INTEGER | Foreign key to officials | 1 |
| role_id | INTEGER | Foreign key to official roles | 1 |
| position | VARCHAR | Court position | Lead |
| order_num | INTEGER | Order in crew | 1 |
| is_alternate | BOOLEAN | If serving as alternate | false |
| assignment_date | TIMESTAMP | When assigned to game | 2024-03-07T14:00Z |
| notes | VARCHAR | Assignment notes | Replacement for injured official |

**Indexing:**
- Order by `event_id, order_num` for efficient crew lookup
- Create index for official assignment history:
  ```sql
  CREATE INDEX idx_event_officials_official ON EventOfficials(official_id, event_id);
  ```
- Create index for role-based queries:
  ```sql
  CREATE INDEX idx_event_officials_role ON EventOfficials(role_id, event_id);
  ```

## OfficialStatistics

Statistical tracking for officials.

| Column | Type | Description | Example |
|--------|------|-------------|---------|
| official_stat_id | INTEGER | Primary key (auto-increment) | 1 |
| official_id | INTEGER | Foreign key to officials | 1 |
| season_id | INTEGER | Foreign key to seasons | 1 |
| event_id | INTEGER | Foreign key to events (NULL for season stats) | 1 |
| group_id | INTEGER | Foreign key to groups/conferences | 1 |
| stat_type | VARCHAR | Type of statistic | fouls_per_game |
| stat_value | DECIMAL | Statistical value | 18.7 |
| rank | INTEGER | Rank (if applicable) | 23 |
| percentile | DECIMAL | Percentile among officials | 0.83 |
| includes_tournaments | BOOLEAN | If includes tournament games | true |
| last_updated | TIMESTAMP | When stat was updated | 2024-03-15T16:30Z |

**Indexing:**
- Order by `official_id, season_id, stat_type` to optimize official-specific stat retrieval
- Create index for comparative analysis:
  ```sql
  CREATE INDEX idx_official_statistics_type ON OfficialStatistics(stat_type, season_id, stat_value);
  ```

## OfficialFoulCalls

Detailed tracking of fouls called by officials.

| Column | Type | Description | Example |
|--------|------|-------------|---------|
| foul_call_id | INTEGER | Primary key (auto-increment) | 1 |
| event_id | INTEGER | Foreign key to events | 1 |
| official_id | INTEGER | Foreign key to officials | 1 |
| play_id | INTEGER | Foreign key to plays | 124 |
| team_id | INTEGER | Team called for foul | 1 |
| opponent_team_id | INTEGER | Opponent team | 2 |
| foul_type | VARCHAR | Type of foul | personal |
| foul_severity | VARCHAR | Severity level | common |
| period | INTEGER | Game period | 1 |
| clock_time | VARCHAR | Game clock at time of call | 15:42 |
| home_score | INTEGER | Home team score at time of call | 18 |
| away_score | INTEGER | Away team score at time of call | 14 |
| score_differential | INTEGER | Score differential | 4 |
| seconds_remaining | INTEGER | Seconds remaining in game | 2142 |
| coordinates_x | DECIMAL | X coordinate on court | 12.5 |
| coordinates_y | DECIMAL | Y coordinate on court | 18.2 |
| is_disputed | BOOLEAN | If call was disputed | false |
| overturned | BOOLEAN | If call was overturned | false |
| athlete_id | INTEGER | Athlete charged with foul | 28 |
| is_technical | BOOLEAN | If technical foul | false |
| is_flagrant | BOOLEAN | If flagrant foul | false |
| review_time | INTEGER | Seconds spent reviewing | 0 |

**Indexing:**
- Order by `event_id, clock_time` for chronological ordering
- Create indexes for analysis patterns:
  ```sql
  CREATE INDEX idx_official_foul_calls_official ON OfficialFoulCalls(official_id, foul_type);
  CREATE INDEX idx_official_foul_calls_team ON OfficialFoulCalls(team_id, event_id);
  CREATE INDEX idx_official_foul_calls_period ON OfficialFoulCalls(event_id, period) WHERE seconds_remaining < 120;
  ```

## OfficialReviews

Tracks replay reviews conducted by officials.

| Column | Type | Description | Example |
|--------|------|-------------|---------|
| review_id | INTEGER | Primary key (auto-increment) | 1 |
| event_id | INTEGER | Foreign key to events | 1 |
| official_id | INTEGER | Lead official for review | 1 |
| play_id | INTEGER | Foreign key to plays | 156 |
| review_type | VARCHAR | Type of review | shot_clock_violation |
| trigger_type | VARCHAR | What triggered review | coach_challenge |
| period | INTEGER | Game period | 2 |
| clock_time | VARCHAR | Game clock at review | 5:23 |
| duration_seconds | INTEGER | Review duration in seconds | 85 |
| ruling | VARCHAR | Final ruling | confirmed |
| original_call | VARCHAR | Original call | violation |
| changed_call | BOOLEAN | If call was changed | false |
| review_outcome | VARCHAR | Outcome description | Shot clock violation confirmed |
| initiating_team_id | INTEGER | Team that initiated review | 2 |
| team_beneficiary | INTEGER | Team benefiting from ruling | 2 |

**Indexing:**
- Order by `event_id, clock_time` for chronological analysis
- Create index for review analysis:
  ```sql
  CREATE INDEX idx_official_reviews_type ON OfficialReviews(review_type, ruling);
  CREATE INDEX idx_official_reviews_outcome ON OfficialReviews(changed_call) WHERE changed_call = true;
  ```

## OfficialAssignments

Season-level assignment tracking for officials.

| Column | Type | Description | Example |
|--------|------|-------------|---------|
| assignment_id | INTEGER | Primary key (auto-increment) | 1 |
| official_id | INTEGER | Foreign key to officials | 1 |
| season_id | INTEGER | Foreign key to seasons | 1 |
| group_id | INTEGER | Foreign key to groups/conferences | 1 |
| assignment_type | VARCHAR | Type of assignment | regular |
| max_games | INTEGER | Maximum allowed games | 65 |
| rank | INTEGER | Official's rank within conference | 5 |
| is_tournament_eligible | BOOLEAN | If eligible for tournament | true |
| is_postseason_eligible | BOOLEAN | If eligible for postseason | true |
| status | VARCHAR | Assignment status | active |
| assigner_name | VARCHAR | Name of assigner | John Smith |
| notes | VARCHAR | Assignment notes | Primary ACC official with Big Ten cross-assignments |

**Indexing:**
- Order by `official_id, season_id` to optimize lookup by official
- Create index for conference assignments:
  ```sql
  CREATE INDEX idx_official_assignments_group ON OfficialAssignments(group_id, season_id);
  ```

## OfficialCrews

Persistent officiating crew information.

| Column | Type | Description | Example |
|--------|------|-------------|---------|
| crew_id | INTEGER | Primary key (auto-increment) | 1 |
| name | VARCHAR | Crew name | Ayers Crew |
| season_id | INTEGER | Foreign key to seasons | 1 |
| primary_conference | INTEGER | Foreign key to groups/conferences | 1 |
| is_permanent | BOOLEAN | If permanent crew | true |
| games_together | INTEGER | Number of games officiated together | 22 |
| crew_chief_id | INTEGER | Foreign key to officials (crew chief) | 1 |
| last_active_date | TIMESTAMP | Date of last game together | 2024-03-10T21:45Z |
| formation_date | TIMESTAMP | When crew was formed | 2023-10-15T00:00Z |
| notes | VARCHAR | Additional information | Regular ACC crew with strong performance metrics |

**Indexing:**
- Order by `season_id, primary_conference` to optimize conference lookups
- Create index for crew chief lookups:
  ```sql
  CREATE INDEX idx_official_crews_chief ON OfficialCrews(crew_chief_id, season_id);
  ```

## OfficialCrewMembers

Links officials to their crews.

| Column | Type | Description | Example |
|--------|------|-------------|---------|
| crew_member_id | INTEGER | Primary key (auto-increment) | 1 |
| crew_id | INTEGER | Foreign key to crews | 1 |
| official_id | INTEGER | Foreign key to officials | 1 |
| role_id | INTEGER | Foreign key to official roles | 1 |
| is_regular | BOOLEAN | If regular member | true |
| games_with_crew | INTEGER | Games with this crew | 22 |
| joined_date | TIMESTAMP | When joined the crew | 2023-10-15T00:00Z |
| last_game_date | TIMESTAMP | Last game with crew | 2024-03-10T21:45Z |
| order_num | INTEGER | Position in crew | 1 |

**Indexing:**
- Order by `crew_id, order_num` to optimize crew listing
- Create index for official membership:
  ```sql
  CREATE INDEX idx_official_crew_members_official ON OfficialCrewMembers(official_id, crew_id);
  ```

## OfficialTendencies

Tracks officiating style and tendencies.

| Column | Type | Description | Example |
|--------|------|-------------|---------|
| tendency_id | INTEGER | Primary key (auto-increment) | 1 |
| official_id | INTEGER | Foreign key to officials | 1 |
| season_id | INTEGER | Foreign key to seasons | 1 |
| tendency_type | VARCHAR | Type of tendency | foul_calling |
| tendency_category | VARCHAR | Category | strictness |
| tendency_value | DECIMAL | Numerical value | 0.72 |
| percentile | DECIMAL | Percentile among officials | 0.85 |
| description | VARCHAR | Description | Calls fouls at a higher than average rate |
| confidence | DECIMAL | Statistical confidence | 0.95 |
| sample_size | INTEGER | Games in sample | 45 |
| last_updated | TIMESTAMP | When updated | 2024-03-01T00:00Z |

**Indexing:**
- Order by `official_id, season_id, tendency_type` for tendency retrieval
- Create index for comparative analysis:
  ```sql
  CREATE INDEX idx_official_tendencies_type ON OfficialTendencies(tendency_type, tendency_value DESC);
  ```

## OfficialEvaluations

Performance evaluations of officials.

| Column | Type | Description | Example |
|--------|------|-------------|---------|
| evaluation_id | INTEGER | Primary key (auto-increment) | 1 |
| official_id | INTEGER | Foreign key to officials | 1 |
| event_id | INTEGER | Foreign key to events | 1 |
| evaluator_id | INTEGER | Foreign key to evaluators | 5 |
| evaluation_date | TIMESTAMP | When evaluation occurred | 2024-01-15T12:00Z |
| overall_rating | DECIMAL | Overall rating (1-5) | 4.5 |
| mechanics_rating | DECIMAL | Mechanics rating | 4.3 |
| judgment_rating | DECIMAL | Judgment rating | 4.7 |
| game_management_rating | DECIMAL | Game management rating | 4.2 |
| teamwork_rating | DECIMAL | Teamwork rating | 4.6 |
| feedback | VARCHAR | Evaluation feedback | Excellent positioning and communication |
| areas_for_improvement | VARCHAR | Areas to improve | Could improve on block/charge calls |
| review_status | VARCHAR | Status of evaluation | final |
| is_self_evaluation | BOOLEAN | If self-evaluation | false |

**Indexing:**
- Create index for official performance tracking:
  ```sql
  CREATE INDEX idx_official_evaluations_official ON OfficialEvaluations(official_id, evaluation_date DESC);
  ```
- Create index for event-specific evaluations:
  ```sql
  CREATE INDEX idx_official_evaluations_event ON OfficialEvaluations(event_id, official_id);
  ```

## Materialized Views for Officials Analysis

To facilitate common analytics queries on officials data, we recommend creating these materialized views:

```sql
-- Official foul calling patterns by game situation
CREATE MATERIALIZED VIEW official_foul_patterns AS
SELECT 
    o.official_id,
    o.full_name,
    s.season_id,
    s.display_name AS season,
    fc.foul_type,
    CASE 
        WHEN fc.seconds_remaining <= 120 THEN 'crunch_time'
        WHEN fc.seconds_remaining <= 300 THEN 'late_game'
        WHEN fc.period = 2 AND fc.seconds_remaining >= 1140 THEN 'early_second_half'
        WHEN fc.period = 1 AND fc.seconds_remaining <= 300 THEN 'late_first_half'
        ELSE 'regular_play'
    END AS game_situation,
    COUNT(*) AS total_calls,
    AVG(ABS(fc.score_differential)) AS avg_score_differential,
    COUNT(CASE WHEN fc.is_disputed = true THEN 1 END) AS disputed_calls,
    COUNT(CASE WHEN fc.overturned = true THEN 1 END) AS overturned_calls,
    COUNT(CASE WHEN fc.is_technical = true THEN 1 END) AS technical_fouls
FROM 
    OfficialFoulCalls fc
JOIN Events e ON fc.event_id = e.event_id
JOIN Officials o ON fc.official_id = o.official_id
JOIN Seasons s ON e.season_id = s.season_id
GROUP BY 
    o.official_id, o.full_name, s.season_id, s.display_name, fc.foul_type,
    CASE 
        WHEN fc.seconds_remaining <= 120 THEN 'crunch_time'
        WHEN fc.seconds_remaining <= 300 THEN 'late_game'
        WHEN fc.period = 2 AND fc.seconds_remaining >= 1140 THEN 'early_second_half'
        WHEN fc.period = 1 AND fc.seconds_remaining <= 300 THEN 'late_first_half'
        ELSE 'regular_play'
    END
ORDER BY 
    s.season_id DESC, o.full_name, total_calls DESC;
``` 