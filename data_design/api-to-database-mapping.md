# API to Database Mapping

This document provides a comprehensive mapping between ESPN API endpoints and our database tables. It serves as a reference for implementing the ETL (Extract, Transform, Load) processes that will populate our analytical database from the ESPN API.

## Purpose

- Clearly define which API endpoints are used to populate each database table
- Provide implementation guidance for data engineers
- Document the relationships between source data and target schema
- Serve as a reference during development and maintenance

## Base URL

All endpoints referenced in this document use the following base URL:

```
https://sports.core.api.espn.com/v2/sports/basketball/leagues/mens-college-basketball
```

## Core Dimension Tables

| Database Table | API Endpoint | Notes |
|----------------|--------------|-------|
| **Leagues** | `/{base_url}` | League information is mostly static and can be hardcoded for NCAA Men's Basketball |
| **Seasons** | `/seasons` <br> `/seasons/{year}` | Use the seasons endpoint to populate basic season information |
| **SeasonTypes** | `/seasons/{year}/types` <br> `/seasons/{year}/types/{type}` | Use season types endpoints to populate the different types (regular season, postseason) |
| **Weeks** | `/seasons/{year}/types/{type}/weeks` <br> `/seasons/{year}/types/{type}/weeks/{week}` | Use week endpoints to populate weekly divisions |
| **Franchises** | `/franchises` <br> `/franchises/{id}` | Get long-term program information |
| **Teams** | `/seasons/{year}/teams` <br> `/seasons/{year}/teams/{team_id}` | Season-specific team instances |
| **Groups** | `/seasons/{year}/types/{type}/groups` <br> `/seasons/{year}/types/{type}/groups/{group_id}` | Conference/division structures |
| **Venues** | Extracted from `/events/{event_id}` | Venue information is embedded in event responses |
| **Athletes** | `/athletes/{athlete_id}` <br> `/seasons/{year}/teams/{team_id}/athletes` | Player information |
| **Positions** | Extracted from `/athletes/{athlete_id}` | Position data is embedded in athlete responses |

## Core Relationship Tables

| Database Table | API Endpoint | Notes |
|----------------|--------------|-------|
| **TeamGroups** | `/seasons/{year}/teams/{team_id}/groups` | Maps teams to conferences/divisions |
| **AthleteTeams** | `/seasons/{year}/teams/{team_id}/athletes` <br> `/athletes/{athlete_id}` | Maps athletes to teams |
| **GroupHierarchy** | `/seasons/{year}/types/{type}/groups` | Maps parent-child relationships for groups |

## Core Fact Tables

| Database Table | API Endpoint | Notes |
|----------------|--------------|-------|
| **Events** | `/events/{event_id}` <br> `/seasons/{year}/types/{type}/weeks/{week}/events` | Game events data |
| **TeamEvents** | `/events/{event_id}` | Team participation in events; extracted from competitions array |
| **GameStatistics** | `/events/{event_id}/competitions/{competition_id}/competitors/{competitor_id}/statistics` | Team statistics per game |

## Statistics Tables

| Database Table | API Endpoint | Notes |
|----------------|--------------|-------|
| **StatCategories** | `/athletes/{athlete_id}/statistics/0` <br> `/teams/{team_id}/statistics` | Extract from the categories array in statistics responses |
| **StatTypes** | `/athletes/{athlete_id}/statistics/0` <br> `/teams/{team_id}/statistics` | Extract from the stats array within categories |
| **StatScopes** | Derived | Game/season/career scopes are derived from context |
| **TeamStatisticsFact** | `/teams/{team_id}/statistics` <br> `/events/{event_id}/competitions/{competition_id}/competitors/{competitor_id}/statistics` | Team statistics (season and game) |
| **AthleteStatisticsFact** | `/athletes/{athlete_id}/statistics/0` <br> `/events/{event_id}/competitions/{competition_id}/competitors/{competitor_id}/athletes/{athlete_id}/statistics` | Athlete statistics (career, season, and game) |
| **LeagueAverages** | Calculated | Derived from collected statistics |
| **StatisticalLeaders** | `/seasons/{year}/types/{type}/leaders` | League statistical leaders |

## Rankings Tables

| Database Table | API Endpoint | Notes |
|----------------|--------------|-------|
| **RankingSystems** | `/rankings` | Information about different polls (AP, Coaches) |
| **PollSchedule** | `/seasons/{year}/types/{type}/weeks/{week}/rankings` | When polls are released |
| **TeamRankings** | `/seasons/{year}/types/{type}/weeks/{week}/rankings/{ranking_id}` | Historical team rankings |
| **RankingComparisons** | Calculated | Derived from collected ranking data |

## Tournament Structure Tables

| Database Table | API Endpoint | Notes |
|----------------|--------------|-------|
| **Tournaments** | `/seasons/{year}/types/3` | Postseason tournaments |
| **TournamentBrackets** | `/tournaments/{tournament_id}/brackets` | Tournament bracket structure |
| **TournamentRounds** | `/tournaments/{tournament_id}/brackets/{bracket_id}/rounds` | Rounds within tournaments |
| **TournamentSeeds** | `/tournaments/{tournament_id}/brackets/{bracket_id}/seeds` | Team seeding information |
| **TournamentMatchups** | `/tournaments/{tournament_id}/brackets/{bracket_id}/matchups` | Matchup information for tournament games |

## Play-by-Play Tables

| Database Table | API Endpoint | Notes |
|----------------|--------------|-------|
| **Plays** | `/events/{event_id}/competitions/{competition_id}/plays` | Detailed play-by-play data |
| **PlayTypes** | `/events/{event_id}/competitions/{competition_id}/plays` | Extracted from play data |
| **PlayParticipants** | `/events/{event_id}/competitions/{competition_id}/plays` | Extracted from play data |

## Shot Charts Tables

| Database Table | API Endpoint | Notes |
|----------------|--------------|-------|
| **ShotCharts** | `/events/{event_id}/competitions/{competition_id}/shotcharts` | Shot location data |
| **CourtZones** | Derived | Calculated from shot chart data |
| **ShotLocations** | `/events/{event_id}/competitions/{competition_id}/shotcharts` | Individual shot locations |

## Injury Tracking Tables

| Database Table | API Endpoint | Notes |
|----------------|--------------|-------|
| **InjuryTypes** | `/athletes/{athlete_id}/injuries` | Extracted from injury data |
| **Injuries** | `/athletes/{athlete_id}/injuries` | Athlete injury records |
| **AthleteInjuries** | `/athletes/{athlete_id}/injuries` | Mapping athletes to injuries |
| **InjuryUpdates** | `/athletes/{athlete_id}/injuries` | Updates to injury status |
| **InjuryPlayEvents** | `/events/{event_id}/competitions/{competition_id}/plays` | Cross-referenced with injury data |

## Betting Odds Tables

| Database Table | API Endpoint | Notes |
|----------------|--------------|-------|
| **OddsProviders** | `/events/{event_id}/competitions/{competition_id}/odds` | Extracted from odds providers data |
| **EventOdds** | `/events/{event_id}/competitions/{competition_id}/odds` | Game odds information |
| **FuturesMarkets** | `/odds/futures` | Futures betting markets |
| **FuturesOdds** | `/odds/futures/{market_id}` | Futures odds for teams/athletes |

## Team Performance Metrics Tables

| Database Table | API Endpoint | Notes |
|----------------|--------------|-------|
| **TeamPerformanceMetrics** | Calculated | Derived from team statistics |
| **TeamPossessionData** | Calculated | Derived from play-by-play data |
| **TeamRankingPerformance** | Calculated | Combines ranking and performance data |

## Data Population Strategy

The recommended approach for populating these tables follows these steps:

1. **Populate Core Dimensions First**
   - Leagues, Seasons, SeasonTypes, Teams, Groups, Athletes

2. **Build Relationships**
   - TeamGroups, AthleteTeams, GroupHierarchy

3. **Populate Core Facts**
   - Events, TeamEvents, GameStatistics

4. **Populate Statistics and Rankings**
   - StatCategories, StatTypes, TeamStatisticsFact, AthleteStatisticsFact
   - RankingSystems, TeamRankings

5. **Add Specialized Components**
   - Play-by-Play data
   - Shot Charts
   - Tournament structures
   - Injury tracking
   - Betting odds

## Implementation Approach

### Batch Processing

For initial data loading and historical data:
1. Start with the most recent complete season
2. Process each entity type in the order outlined above
3. For each endpoint that returns collections, implement pagination handling
4. Store the raw API responses for potential reprocessing
5. Transform and load the data into the appropriate tables

### Incremental Updates

For ongoing data maintenance:
1. Implement a scheduler to check for new or updated data daily
2. Focus on active season data during the season
3. Process events as they occur
4. Update statistics and derived metrics after games
5. Handle rankings updates on their regular schedule (weekly for polls)

## Handling API Limitations

- Implement appropriate rate limiting in the ETL process
- Store data locally to minimize redundant API calls
- Use conditional HTTP requests (If-Modified-Since) where supported
- Implement robust error handling and retry logic 