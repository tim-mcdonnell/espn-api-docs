# Tournament Structure

This section documents the tables used for storing tournament data in our ESPN API data structure. These tables track tournaments, brackets, seeding, matchups, and team advancement.

## Tournaments

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

## TournamentBrackets

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

## TournamentRounds

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

## TournamentSeeds

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

## TournamentMatchups

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

## TournamentAdvancement

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

## TournamentSelectionCriteria

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

## TeamTournamentSelectionMetrics

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

## TournamentBids

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

## BracketologyProjections

Historical projections from bracketologists.

| Column | Type | Description | Example |
|--------|------|-------------|---------|
| projection_id | INTEGER | Primary key (auto-increment) | 1 |
| tournament_id | INTEGER | Foreign key to tournaments | 1 |
| source | VARCHAR | Projection source/author | Joe Lunardi |
| projection_date | TIMESTAMP | When projection was made | 2025-03-10T12:00Z |
| team_id | INTEGER | Foreign key to teams | 52 |
| seed_number | INTEGER | Projected seed | 3 |
| bracket_name | VARCHAR | Projected bracket | East |
| is_last_four_in | BOOLEAN | If projected as last four in | false |
| is_first_four_out | BOOLEAN | If projected as first four out | false |
| seed_movement | INTEGER | Change from previous projection | 1 |
| notes | VARCHAR | Additional projection notes | Moving up after conference tournament win |

**Indexing:**
- Order by `tournament_id, projection_date DESC, source` for chronological analysis
- Create index for team projections:
  ```sql
  CREATE INDEX idx_bracketology_projections_team ON BracketologyProjections(team_id, tournament_id, projection_date DESC);
  ``` 