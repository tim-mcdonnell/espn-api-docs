# Broadcast Data Structure

This section documents the tables used for storing broadcast information in our ESPN API data structure. These tables track information about networks, announcers, and broadcast details for basketball games.

## Networks

Stores information about broadcast networks/channels.

| Column | Type | Description | Example |
|--------|------|-------------|---------|
| network_id | INTEGER | Primary key (auto-increment) | 1 |
| espn_id | VARCHAR | ESPN's identifier (if any) | 101 |
| name | VARCHAR | Full network name | Big Ten Network |
| short_name | VARCHAR | Network abbreviation | BTN |
| type | VARCHAR | Network type | TV |
| logo_url | VARCHAR | URL to network logo | https://a.espncdn.com/i/networks/btn.png |
| parent_company | VARCHAR | Parent company | Fox Corporation |
| is_subscription | BOOLEAN | If requires paid subscription | false |

**Indexing:**
- No explicit indexes needed due to small table size
- Order by `name` for quick lookups

## BroadcastMarkets

Defines different market types and regions.

| Column | Type | Description | Example |
|--------|------|-------------|---------|
| market_id | INTEGER | Primary key (auto-increment) | 1 |
| espn_id | VARCHAR | ESPN's identifier | 1 |
| name | VARCHAR | Market name | National |
| type | VARCHAR | Market type | National/Regional/Local |
| region | VARCHAR | Regional identifier (if applicable) | NULL |
| countries | VARCHAR | JSON array of country codes | ["US", "CA"] |
| description | VARCHAR | Market description | Available throughout US |

**Indexing:**
- No explicit indexes needed due to small table size

## Announcers

People who call/announce games.

| Column | Type | Description | Example |
|--------|------|-------------|---------|
| announcer_id | INTEGER | Primary key (auto-increment) | 1 |
| espn_id | VARCHAR | ESPN's identifier | 5678 |
| first_name | VARCHAR | First name | Jim |
| last_name | VARCHAR | Last name | Nantz |
| full_name | VARCHAR | Full name | Jim Nantz |
| role | VARCHAR | Announcer role | Play-by-play |
| bio | VARCHAR | Short biography | CBS Sports lead announcer |
| headshot_url | VARCHAR | URL to headshot image | https://example.com/announcers/nantz.jpg |
| active | BOOLEAN | If currently active | true |

**Indexing:**
- Create index on `last_name, first_name` for name lookups:
  ```sql
  CREATE INDEX idx_announcers_name ON Announcers(last_name, first_name);
  ```

## EventBroadcasts

Broadcasts linked to specific events.

| Column | Type | Description | Example |
|--------|------|-------------|---------|
| broadcast_id | INTEGER | Primary key (auto-increment) | 1 |
| event_id | INTEGER | Foreign key to events | 1 |
| network_id | INTEGER | Foreign key to networks | 1 |
| market_id | INTEGER | Foreign key to broadcast markets | 1 |
| type | VARCHAR | Broadcast type | TV |
| language | VARCHAR | Broadcast language | en |
| start_time | TIMESTAMP | Scheduled start time | 2024-01-10T23:30Z |
| end_time | TIMESTAMP | Scheduled end time | 2024-01-11T01:30Z |
| broadcast_name | VARCHAR | Specific broadcast name | College Basketball |
| channel_number | VARCHAR | Channel number | 610 |
| is_national | BOOLEAN | If nationally televised | true |
| is_home_broadcast | BOOLEAN | If favoring home team | false |
| tape_delayed | BOOLEAN | If not live | false |
| region_restriction | VARCHAR | Regional blackout info | NULL |
| stream_url | VARCHAR | URL to stream (if applicable) | NULL |

**Indexing:**
- Order by `event_id, type` to group broadcasts by type per event
- Create index for network-specific queries:
  ```sql
  CREATE INDEX idx_event_broadcasts_network ON EventBroadcasts(network_id, start_time);
  ```

## BroadcastAnnouncers

Links announcers to specific broadcasts with their roles.

| Column | Type | Description | Example |
|--------|------|-------------|---------|
| broadcast_announcer_id | INTEGER | Primary key (auto-increment) | 1 |
| broadcast_id | INTEGER | Foreign key to event broadcasts | 1 |
| announcer_id | INTEGER | Foreign key to announcers | 1 |
| role | VARCHAR | Role for this broadcast | Play-by-play |
| order_num | INTEGER | Display/priority order | 1 |
| is_primary | BOOLEAN | If primary announcer | true |
| is_sideline | BOOLEAN | If sideline reporter | false |
| notes | VARCHAR | Any specific notes | Guest analyst |

**Indexing:**
- Order by `broadcast_id, order_num` to maintain crew order
- No explicit indexes needed beyond this ordering

## BroadcastRatings

Viewership and ratings data for broadcasts.

| Column | Type | Description | Example |
|--------|------|-------------|---------|
| rating_id | INTEGER | Primary key (auto-increment) | 1 |
| broadcast_id | INTEGER | Foreign key to broadcasts | 1 |
| rating_type | VARCHAR | Type of rating | Nielsen |
| rating_value | DECIMAL | Rating number | 2.4 |
| viewers | INTEGER | Number of viewers (thousands) | 1250 |
| share | DECIMAL | Market share percentage | 5.2 |
| demographic | VARCHAR | Target demographic | P18-49 |
| time_slot | VARCHAR | Broadcasting time slot | Primetime |
| competing_events | VARCHAR | JSON array of competing broadcasts | [{"sport": "NBA", "rating": 3.1}] |
| notes | VARCHAR | Additional context | Conference rivalry game |

**Indexing:**
- Order by `broadcast_id, rating_type` for easy lookups
- Create index for high-rated broadcasts:
  ```sql
  CREATE INDEX idx_broadcast_ratings_value ON BroadcastRatings(rating_value DESC);
  ``` 