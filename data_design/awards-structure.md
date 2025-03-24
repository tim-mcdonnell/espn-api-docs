# Awards Data Structure

This section documents the tables used for storing awards and honors data in our ESPN API data structure. These tables track various basketball awards, their criteria, and their recipients across seasons.

## AwardCategories

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

## AwardRecipients

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

## AllStarTeams

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

## AllStarSelections

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