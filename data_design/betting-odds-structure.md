# Betting Odds Structure

This section documents the tables used for storing betting odds and wagering-related data from the ESPN API. These tables enable the tracking of pre-game odds, line movements, and various betting markets for sports events.

## OddsProviders

Stores information about betting odds providers and sportsbooks.

| Column | Type | Description | Example |
|--------|------|-------------|---------|
| odds_provider_id | INTEGER | Primary key (auto-increment) | 1 |
| espn_id | VARCHAR | ESPN's identifier | 5 |
| uid | VARCHAR | Universal identifier | s:40~l:41~op:5 |
| name | VARCHAR | Name of the odds provider | DraftKings |
| abbreviation | VARCHAR | Short abbreviation for provider | DK |
| logo_url | VARCHAR | URL to provider logo | https://a.espncdn.com/i/bookmakers/draft-kings-logo.png |
| description | VARCHAR | Brief description of provider | DraftKings Sportsbook |
| is_official | BOOLEAN | Whether provider is an official partner | true |
| priority | INTEGER | Display priority ranking | 1 |
| is_active | BOOLEAN | Whether provider is currently active | true |
| created_at | TIMESTAMP | Record creation timestamp | 2023-07-12T14:30:00Z |
| updated_at | TIMESTAMP | Record update timestamp | 2023-08-15T09:22:31Z |

**Indexing:**
```sql
CREATE INDEX idx_odds_providers_name ON OddsProviders(name);
CREATE INDEX idx_odds_providers_espn_id ON OddsProviders(espn_id);
```

## EventOdds

Stores the main odds record for each event with consensus lines.

| Column | Type | Description | Example |
|--------|------|-------------|---------|
| event_odds_id | INTEGER | Primary key (auto-increment) | 1 |
| event_id | INTEGER | Foreign key to events | 401524661 |
| season_id | INTEGER | Foreign key to seasons | 2024 |
| season_type | INTEGER | Season type (1=pre, 2=regular, 3=post) | 2 |
| status | VARCHAR | Status of the betting market | open |
| open_date | TIMESTAMP | When betting opened | 2023-12-30 10:00:00 |
| update_date | TIMESTAMP | Last update timestamp | 2024-01-09 23:30:00 |
| home_team_id | INTEGER | Foreign key to home team | 2509 |
| away_team_id | INTEGER | Foreign key to away team | 158 |
| consensus_spread | DECIMAL | Consensus point spread | -9.5 |
| consensus_home_moneyline | INTEGER | Consensus home money line | -450 |
| consensus_away_moneyline | INTEGER | Consensus away money line | +350 |
| consensus_over_under | DECIMAL | Consensus total points | 145.5 |
| consensus_home_spread_odds | INTEGER | Consensus home spread odds | -110 |
| consensus_away_spread_odds | INTEGER | Consensus away spread odds | -110 |
| consensus_over_odds | INTEGER | Consensus over odds | -110 |
| consensus_under_odds | INTEGER | Consensus under odds | -110 |
| movement_direction | VARCHAR | Recent line movement direction | toward_home |
| movement_strength | VARCHAR | Strength of recent movement | moderate |
| created_at | TIMESTAMP | Record creation timestamp | 2023-12-30 10:00:00 |
| updated_at | TIMESTAMP | Record update timestamp | 2024-01-09 23:30:00 |

**Indexing:**
```sql
CREATE INDEX idx_event_odds_event ON EventOdds(event_id);
CREATE INDEX idx_event_odds_teams ON EventOdds(home_team_id, away_team_id);
CREATE INDEX idx_event_odds_season ON EventOdds(season_id, season_type);
```

## ProviderEventOdds

Stores odds from specific providers for each event.

| Column | Type | Description | Example |
|--------|------|-------------|---------|
| provider_odds_id | INTEGER | Primary key (auto-increment) | 1 |
| event_odds_id | INTEGER | Foreign key to EventOdds | 1 |
| odds_provider_id | INTEGER | Foreign key to OddsProviders | 1 |
| timestamp | TIMESTAMP | Time of odds snapshot | 2024-01-09T22:30:00Z |
| home_spread | DECIMAL | Provider's home spread | -9.5 |
| away_spread | DECIMAL | Provider's away spread | +9.5 |
| home_spread_odds | INTEGER | Home spread odds | -110 |
| away_spread_odds | INTEGER | Away spread odds | -110 |
| home_moneyline | INTEGER | Home money line | -450 |
| away_moneyline | INTEGER | Away money line | +350 |
| over_under | DECIMAL | Total points line | 145.5 |
| over_odds | INTEGER | Over bet odds | -110 |
| under_odds | INTEGER | Under bet odds | -110 |
| home_team_cover | BOOLEAN | Whether home team covered | NULL |
| away_team_cover | BOOLEAN | Whether away team covered | NULL |
| over_under_result | VARCHAR | Over/under outcome | NULL |
| created_at | TIMESTAMP | Record creation timestamp | 2024-01-09T22:30:00Z |
| updated_at | TIMESTAMP | Record update timestamp | 2024-01-09T22:30:00Z |

**Indexing:**
```sql
CREATE INDEX idx_provider_event_odds_event ON ProviderEventOdds(event_odds_id);
CREATE INDEX idx_provider_event_odds_provider ON ProviderEventOdds(odds_provider_id);
CREATE INDEX idx_provider_event_odds_timestamp ON ProviderEventOdds(timestamp);
```

## OddsMovement

Tracks line movements over time for historical analysis.

| Column | Type | Description | Example |
|--------|------|-------------|---------|
| movement_id | INTEGER | Primary key (auto-increment) | 1 |
| event_odds_id | INTEGER | Foreign key to EventOdds | 1 |
| odds_provider_id | INTEGER | Foreign key to OddsProviders | 1 |
| movement_type | VARCHAR | Type of line that moved | spread |
| old_value | DECIMAL | Previous value | -8.5 |
| new_value | DECIMAL | New value | -9.5 |
| old_odds | INTEGER | Previous odds | -110 |
| new_odds | INTEGER | New odds | -110 |
| movement_timestamp | TIMESTAMP | When movement occurred | 2024-01-09T15:30:00Z |
| movement_direction | VARCHAR | Direction of movement | toward_home |
| magnitude | DECIMAL | Size of movement | 1.0 |
| created_at | TIMESTAMP | Record creation timestamp | 2024-01-09T15:30:00Z |
| updated_at | TIMESTAMP | Record update timestamp | 2024-01-09T15:30:00Z |

**Indexing:**
```sql
CREATE INDEX idx_odds_movement_event ON OddsMovement(event_odds_id);
CREATE INDEX idx_odds_movement_timestamp ON OddsMovement(movement_timestamp);
CREATE INDEX idx_odds_movement_type ON OddsMovement(movement_type, event_odds_id);
CREATE INDEX idx_odds_movement_provider ON OddsMovement(odds_provider_id);
```

## FuturesMarkets

Stores season-long futures betting markets.

| Column | Type | Description | Example |
|--------|------|-------------|---------|
| futures_id | INTEGER | Primary key (auto-increment) | 1 |
| season_id | INTEGER | Foreign key to seasons | 2024 |
| market_name | VARCHAR | Name of futures market | NCAA Tournament Champion |
| market_description | VARCHAR | Description of market | Team to win NCAA Tournament |
| market_category | VARCHAR | Category of market | championship |
| open_date | TIMESTAMP | When market opened | 2023-04-05 00:00:00 |
| close_date | TIMESTAMP | When market closes | 2024-03-15 00:00:00 |
| status | VARCHAR | Market status | open |
| update_frequency | VARCHAR | How often updated | daily |
| created_at | TIMESTAMP | Record creation timestamp | 2023-04-05 00:00:00 |
| updated_at | TIMESTAMP | Record update timestamp | 2024-01-09 00:00:00 |

**Indexing:**
```sql
CREATE INDEX idx_futures_markets_season ON FuturesMarkets(season_id);
CREATE INDEX idx_futures_markets_category ON FuturesMarkets(market_category);
```

## FuturesOdds

Stores individual team/player odds for futures markets.

| Column | Type | Description | Example |
|--------|------|-------------|---------|
| futures_odds_id | INTEGER | Primary key (auto-increment) | 1 |
| futures_id | INTEGER | Foreign key to FuturesMarkets | 1 |
| odds_provider_id | INTEGER | Foreign key to OddsProviders | 1 |
| team_id | INTEGER | Foreign key to teams | 2509 |
| athlete_id | INTEGER | Foreign key to athletes | NULL |
| odds | INTEGER | American odds | +800 |
| probability | DECIMAL | Implied probability | 0.111 |
| timestamp | TIMESTAMP | Time of odds snapshot | 2024-01-09T00:00:00Z |
| previous_odds | INTEGER | Previous odds value | +900 |
| movement_direction | VARCHAR | Recent movement direction | improved |
| created_at | TIMESTAMP | Record creation timestamp | 2023-12-30T00:00:00Z |
| updated_at | TIMESTAMP | Record update timestamp | 2024-01-09T00:00:00Z |

**Indexing:**
```sql
CREATE INDEX idx_futures_odds_futures ON FuturesOdds(futures_id);
CREATE INDEX idx_futures_odds_team ON FuturesOdds(team_id);
CREATE INDEX idx_futures_odds_athlete ON FuturesOdds(athlete_id);
CREATE INDEX idx_futures_odds_provider ON FuturesOdds(odds_provider_id);
```

## BettingTrendsMatchup

Stores historical betting patterns for team matchups.

| Column | Type | Description | Example |
|--------|------|-------------|---------|
| trend_id | INTEGER | Primary key (auto-increment) | 1 |
| team_id | INTEGER | Foreign key to teams | 2509 |
| opponent_id | INTEGER | Foreign key to opponent team | 158 |
| season_id | INTEGER | Foreign key to seasons | 2024 |
| season_type | INTEGER | Season type (1=pre, 2=regular, 3=post) | 2 |
| games_ats_won | INTEGER | Games won against the spread | 3 |
| games_ats_lost | INTEGER | Games lost against the spread | 1 |
| games_ats_push | INTEGER | Games pushed against the spread | 0 |
| ats_cover_percent | DECIMAL | Percentage of ATS covers | 75.0 |
| avg_spread | DECIMAL | Average spread | -7.5 |
| over_count | INTEGER | Count of over results | 2 |
| under_count | INTEGER | Count of under results | 2 |
| avg_total | DECIMAL | Average total points | 142.5 |
| created_at | TIMESTAMP | Record creation timestamp | 2024-01-09 00:00:00 |
| updated_at | TIMESTAMP | Record update timestamp | 2024-01-09 00:00:00 |

**Indexing:**
```sql
CREATE INDEX idx_betting_trends_matchup_teams ON BettingTrendsMatchup(team_id, opponent_id);
CREATE INDEX idx_betting_trends_matchup_season ON BettingTrendsMatchup(season_id, season_type);
```

## Entity Relationships

The betting odds structure integrates with the core data model through the following relationships:

- `EventOdds` links to `Events` via `event_id` to associate odds with specific games
- `EventOdds` links to `Teams` via `home_team_id` and `away_team_id` to identify the competing teams
- `EventOdds` links to `Seasons` via `season_id` to associate odds with a specific season
- `OddsMovement` links to `EventOdds` via `event_odds_id` to track line changes for specific events
- `ProviderEventOdds` links to `EventOdds` via `event_odds_id` and to `OddsProviders` via `odds_provider_id`
- `FuturesMarkets` links to `Seasons` via `season_id` to associate futures with specific seasons
- `FuturesOdds` links to `Teams` via `team_id` and `Athletes` via `athlete_id` for team and player futures

## Usage Examples

### Retrieving Current Odds for an Event

```sql
SELECT e.name as event_name, peo.home_spread, peo.away_spread,
       peo.home_moneyline, peo.away_moneyline, peo.over_under,
       op.name as provider
FROM ProviderEventOdds peo
JOIN EventOdds eo ON peo.event_odds_id = eo.event_odds_id
JOIN Events e ON eo.event_id = e.event_id
JOIN OddsProviders op ON peo.odds_provider_id = op.odds_provider_id
WHERE e.event_id = 401524661
ORDER BY op.priority;
```

### Tracking Line Movement

```sql
SELECT om.movement_timestamp, om.movement_type, om.old_value, om.new_value,
       om.movement_direction, op.name
FROM OddsMovement om
JOIN OddsProviders op ON om.odds_provider_id = op.odds_provider_id
JOIN EventOdds eo ON om.event_odds_id = eo.event_odds_id
WHERE eo.event_id = 401524661
ORDER BY om.movement_timestamp DESC;
```

### Futures Odds for Teams

```sql
SELECT t.display_name AS team, fo.odds AS current_odds, fo.previous_odds,
       fo.probability AS implied_probability, op.name
FROM FuturesOdds fo
JOIN FuturesMarkets fm ON fo.futures_id = fm.futures_id
JOIN Teams t ON fo.team_id = t.team_id
JOIN OddsProviders op ON fo.odds_provider_id = op.odds_provider_id
WHERE fm.market_name = 'NCAA Tournament Champion' 
  AND fm.season_id = 2024
ORDER BY fo.probability DESC;
```

### Finding Teams with Best ATS Records

```sql
SELECT t.display_name AS team_name, 
       SUM(bt.games_ats_won) AS ats_wins,
       SUM(bt.games_ats_lost) AS ats_losses,
       SUM(bt.games_ats_push) AS ats_pushes,
       SUM(bt.games_ats_won) / (SUM(bt.games_ats_won) + SUM(bt.games_ats_lost)) AS cover_rate
FROM BettingTrendsMatchup bt
JOIN Teams t ON bt.team_id = t.team_id
WHERE bt.season_id = 2024 AND bt.season_type = 2
GROUP BY t.team_id, t.display_name
HAVING (SUM(bt.games_ats_won) + SUM(bt.games_ats_lost)) >= 10
ORDER BY cover_rate DESC
LIMIT 10;
``` 