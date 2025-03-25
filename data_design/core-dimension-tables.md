# Core Dimension Tables

This section documents the core dimension tables in our ESPN API data structure. These tables define the primary entities such as leagues, teams, athletes, and other fundamental components of basketball data.

## Leagues

Stores the basic league information.

| Column | Type | Description | Example |
|--------|------|-------------|---------|
| league_id | INTEGER | Primary key | 41 |
| uid | VARCHAR | Universal identifier | s:40~l:41 |
| name | VARCHAR | League name | NCAA Men's Basketball |
| abbreviation | VARCHAR | League abbreviation | NCAAM |
| short_name | VARCHAR | Short name | NCAAM |
| slug | VARCHAR | URL identifier | mens-college-basketball |
| created_at | TIMESTAMP | Record creation timestamp | 2024-03-01T12:00:00Z |
| updated_at | TIMESTAMP | Record update timestamp | 2024-03-15T09:30:00Z |

**Indexing:** No explicit indexes needed as this is a small table with an integer primary key.

## Seasons

Stores information about seasons.

| Column | Type | Description | Example |
|--------|------|-------------|---------|
| season_id | INTEGER | Primary key (auto-increment) | 1 |
| year | INTEGER | Season year | 2025 |
| display_name | VARCHAR | Display name | 2024-25 |
| start_date | TIMESTAMP | Season start date | 2024-07-13T07:00Z |
| end_date | TIMESTAMP | Season end date | 2025-04-09T06:59Z |
| league_id | INTEGER | Foreign key to leagues | 41 |
| created_at | TIMESTAMP | Record creation timestamp | 2024-03-01T12:00:00Z |
| updated_at | TIMESTAMP | Record update timestamp | 2024-03-15T09:30:00Z |

**Indexing:** 
- Order by `year` to optimize zonemap effectiveness for year-based filters
- No explicit indexes needed due to small table size and integer primary key

## SeasonTypes

Stores types of seasons (preseason, regular season, etc.).

| Column | Type | Description | Example |
|--------|------|-------------|---------|
| season_type_id | INTEGER | Primary key (auto-increment) | 1 |
| espn_id | VARCHAR | ESPN's ID | 2 |
| season_id | INTEGER | Foreign key to seasons | 1 |
| name | VARCHAR | Type name | Regular Season |
| abbreviation | VARCHAR | Short abbreviation | reg |
| start_date | TIMESTAMP | Type start date | 2024-11-04T08:00Z |
| end_date | TIMESTAMP | Type end date | 2025-03-18T06:59Z |
| has_groups | BOOLEAN | If has group info | true |
| has_standings | BOOLEAN | If has standings | true |
| slug | VARCHAR | URL identifier | regular-season |
| created_at | TIMESTAMP | Record creation timestamp | 2024-03-01T12:00:00Z |
| updated_at | TIMESTAMP | Record update timestamp | 2024-03-15T09:30:00Z |

**Indexing:** No explicit indexes needed as this is a small table with few distinct values.

## Weeks

Weekly divisions within a season type.

| Column | Type | Description | Example |
|--------|------|-------------|---------|
| week_id | INTEGER | Primary key (auto-increment) | 1 |
| season_type_id | INTEGER | Foreign key to season types | 1 |
| number | INTEGER | Week number | 1 |
| text | VARCHAR | Display text | Week 1 |
| start_date | TIMESTAMP | Week start date | 2024-11-04T08:00Z |
| end_date | TIMESTAMP | Week end date | 2024-11-11T07:59Z |
| created_at | TIMESTAMP | Record creation timestamp | 2024-03-01T12:00:00Z |
| updated_at | TIMESTAMP | Record update timestamp | 2024-03-15T09:30:00Z |

**Indexing:**
- Order by `season_type_id, start_date` to improve filtering by date ranges within a season type
- No explicit indexes needed due to small cardinality

## Franchises (Programs)

Long-term athletic programs.

| Column | Type | Description | Example |
|--------|------|-------------|---------|
| franchise_id | INTEGER | Primary key (auto-increment) | 1 |
| espn_id | VARCHAR | ESPN's ID | 153 |
| uid | VARCHAR | Universal identifier | s:40~l:41~f:153 |
| location | VARCHAR | Geographic location | North Carolina |
| name | VARCHAR | Team name | Tar Heels |
| abbreviation | VARCHAR | Short abbreviation | UNC |
| display_name | VARCHAR | Full display name | North Carolina Tar Heels |
| short_display_name | VARCHAR | Short display name | UNC |
| color | VARCHAR | Primary color | 7BAFD4 |
| alternate_color | VARCHAR | Alternate color | 13294B |
| is_active | BOOLEAN | If franchise is active | true |
| slug | VARCHAR | URL identifier | north-carolina-tar-heels |

**Indexing:**
- Create explicit index on `espn_id` for API integration queries: 
  ```sql
  CREATE INDEX idx_franchises_espn_id ON Franchises(espn_id);
  ```
- Consider index on `slug` if URL-based lookups are frequent

## Teams

Season-specific team instances.

| Column | Type | Description | Example |
|--------|------|-------------|---------|
| team_id | INTEGER | Primary key (auto-increment) | 1 |
| espn_id | VARCHAR | ESPN's ID | 52 |
| uid | VARCHAR | Universal identifier | s:40~l:41~t:52 |
| season_id | INTEGER | Foreign key to seasons | 1 |
| franchise_id | INTEGER | Foreign key to franchises | 1 |
| is_active | BOOLEAN | If team is active | true |
| is_all_star | BOOLEAN | If all-star team | false |

**Indexing:**
- Create explicit index on `espn_id` for API queries:
  ```sql
  CREATE INDEX idx_teams_espn_id ON Teams(espn_id);
  ```
- Order table by `season_id, franchise_id` to optimize filtering by both dimensions

## Groups (Conferences/Divisions)

Conference and division structures.

| Column | Type | Description | Example |
|--------|------|-------------|---------|
| group_id | INTEGER | Primary key (auto-increment) | 1 |
| espn_id | VARCHAR | ESPN's ID | 1 |
| season_id | INTEGER | Foreign key to seasons | 1 |
| season_type_id | INTEGER | Foreign key to season types | 1 |
| name | VARCHAR | Group name | Atlantic Coast |
| abbreviation | VARCHAR | Abbreviation | ACC |
| short_name | VARCHAR | Short name | ACC |
| is_conference | BOOLEAN | If it's a conference | true |
| parent_group_id | INTEGER | Self-reference to parent group | NULL |
| slug | VARCHAR | URL identifier | atlantic-coast |

**Indexing:**
- Order by `season_id, season_type_id` to optimize filtering on these frequently queried dimensions
- No explicit indexes needed due to moderate size

## Venues

Game locations.

| Column | Type | Description | Example |
|--------|------|-------------|---------|
| venue_id | INTEGER | Primary key (auto-increment) | 1 |
| espn_id | VARCHAR | ESPN's ID | 4103 |
| name | VARCHAR | Venue name | Mackey Arena |
| capacity | INTEGER | Seating capacity | 14804 |
| indoor | BOOLEAN | If indoor venue | true |
| city | VARCHAR | City | West Lafayette |
| state | VARCHAR | State | IN |
| country | VARCHAR | Country | USA |

**Indexing:**
- Create index on `espn_id` for API integration:
  ```sql
  CREATE INDEX idx_venues_espn_id ON Venues(espn_id);
  ```
- Consider index on `state, city` if geographic filtering is common

## Athletes

Player information.

| Column | Type | Description | Example |
|--------|------|-------------|---------|
| athlete_id | INTEGER | Primary key (auto-increment) | 1 |
| espn_id | VARCHAR | ESPN's ID | 4433137 |
| uid | VARCHAR | Universal identifier | s:40~l:41~a:4433137 |
| guid | VARCHAR | Global unique ID | 9a0fb84c-85b5-47b2-b877-d44ff3a5c93f |
| first_name | VARCHAR | First name | Zach |
| last_name | VARCHAR | Last name | Edey |
| full_name | VARCHAR | Full name | Zach Edey |
| display_name | VARCHAR | Display name | Zach Edey |
| short_name | VARCHAR | Short name | Z. Edey |
| weight | INTEGER | Weight in pounds | 305 |
| height | INTEGER | Height in inches | 87 |
| display_height | VARCHAR | Formatted height | 7' 3" |
| date_of_birth | TIMESTAMP | Birth date | 2002-05-14T04:00Z |
| position_id | INTEGER | Foreign key to positions | 1 |
| jersey | VARCHAR | Jersey number | 15 |
| slug | VARCHAR | URL identifier | zach-edey |

**Indexing:**
- Create explicit indexes for common search patterns:
  ```sql
  CREATE INDEX idx_athletes_espn_id ON Athletes(espn_id);
  CREATE INDEX idx_athletes_name ON Athletes(last_name, first_name);
  ```
- Order by `last_name, first_name` to improve name-based searches through zonemaps

## Positions

Player positions.

| Column | Type | Description | Example |
|--------|------|-------------|---------|
| position_id | INTEGER | Primary key (auto-increment) | 1 |
| espn_id | VARCHAR | ESPN's ID | 1 |
| name | VARCHAR | Position name | Center |
| display_name | VARCHAR | Display name | Center |
| abbreviation | VARCHAR | Abbreviation | C |
| parent_position_id | INTEGER | Self-reference to parent position | NULL |

**Indexing:** No explicit indexes needed due to small table size. 