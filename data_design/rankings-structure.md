# Rankings Data Structure

This section documents the tables used for storing and organizing rankings and polls data in our ESPN API data structure. These tables track team rankings across different ranking systems and time periods.

## RankingSystems

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

## PollSchedule

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

## TeamRankings

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

## RankingComparisons

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