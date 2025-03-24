# Core Fact Tables

This section documents the core fact tables in our ESPN API data structure. These tables store event data, game results, and other factual information that forms the foundation of basketball analytics.

## Events (Games)

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

## TeamEvents

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

## GameStatistics

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