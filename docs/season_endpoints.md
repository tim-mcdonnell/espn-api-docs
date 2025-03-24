# Season Endpoints

This document provides detailed information about the NCAA Men's Basketball season-related endpoints in the ESPN v2 API. All examples show actual API responses obtained through direct API calls.

## Base URL

```
https://sports.core.api.espn.com/v2/sports/basketball/leagues/mens-college-basketball
```

## Season List Endpoint

### `/seasons`

Returns a paginated list of all available seasons.

**Query Parameters:**
- `pageSize` (optional): Number of items per page (default: 25)
- `pageIndex` (optional): Page number (default: 1)

**Response Structure:**
- `count`: Total number of seasons available
- `pageIndex`: Current page number
- `pageSize`: Number of items per page
- `pageCount`: Total number of pages
- `items`: Array of season references

**Example Request:**
```bash
curl -s "https://sports.core.api.espn.com/v2/sports/basketball/leagues/mens-college-basketball/seasons" | jq
```

**Example Response:**
```json
{
  "count": 87,
  "pageIndex": 1,
  "pageSize": 25,
  "pageCount": 4,
  "items": [
    {
      "$ref": "http://sports.core.api.espn.com/v2/sports/basketball/leagues/mens-college-basketball/seasons/2025?lang=en&region=us"
    },
    {
      "$ref": "http://sports.core.api.espn.com/v2/sports/basketball/leagues/mens-college-basketball/seasons/2024?lang=en&region=us"
    },
    // Additional seasons...
  ]
}
```

**Notes:**
- Historical data goes back to the 1939 season (which refers to the 1939-1940 academic year)
- Total of 87 seasons available (as of current API version)
- Seasons are ordered from newest to oldest

## Season Detail Endpoint

### `/seasons/{year}`

Returns detailed information about a specific season.

**Path Parameters:**
- `year`: The year of the season (e.g., 2025 for the 2024-25 season)

**Response Fields:**
- `year`: Season year
- `startDate`: ISO 8601 formatted start date
- `endDate`: ISO 8601 formatted end date
- `displayName`: Human-readable season name (e.g., "2024-25")
- `type`: Current active season type
- `types`: List of all season types for this season
- `rankings`: Reference to rankings endpoint
- `powerIndexes`: Reference to power index endpoint (newer seasons only)
- `powerIndexLeaders`: Reference to power index leaders endpoint (newer seasons only)
- `athletes`: Reference to athletes endpoint
- `awards`: Reference to awards endpoint
- `futures`: Reference to futures/predictions endpoint
- `leaders`: Reference to statistical leaders endpoint

**Example Request:**
```bash
curl -s "https://sports.core.api.espn.com/v2/sports/basketball/leagues/mens-college-basketball/seasons/2025" | jq
```

**Example Response:**
```json
{
  "$ref": "http://sports.core.api.espn.com/v2/sports/basketball/leagues/mens-college-basketball/seasons/2025?lang=en&region=us",
  "year": 2025,
  "startDate": "2024-07-13T07:00Z",
  "endDate": "2025-04-09T06:59Z",
  "displayName": "2024-25",
  "type": {
    "$ref": "http://sports.core.api.espn.com/v2/sports/basketball/leagues/mens-college-basketball/seasons/2025/types/3?lang=en&region=us",
    "id": "3",
    "type": 3,
    "name": "Postseason",
    "abbreviation": "post",
    "year": 2025,
    "startDate": "2025-03-18T07:00Z",
    "endDate": "2025-04-09T06:59Z",
    "hasGroups": false,
    "hasStandings": true,
    "hasLegs": false,
    "groups": {
      "$ref": "http://sports.core.api.espn.com/v2/sports/basketball/leagues/mens-college-basketball/seasons/2025/types/3/groups?lang=en&region=us"
    },
    "week": {
      "$ref": "http://sports.core.api.espn.com/v2/sports/basketball/leagues/mens-college-basketball/seasons/2025/types/3/weeks/2?lang=en&region=us",
      "number": 2,
      "startDate": "2025-03-24T07:00Z",
      "endDate": "2025-03-31T06:59Z",
      "text": "Week 2",
      "rankings": {
        "$ref": "http://sports.core.api.espn.com/v2/sports/basketball/leagues/mens-college-basketball/seasons/2025/types/3/weeks/2/rankings?lang=en&region=us"
      }
    },
    "weeks": {
      "$ref": "http://sports.core.api.espn.com/v2/sports/basketball/leagues/mens-college-basketball/seasons/2025/types/3/weeks?lang=en&region=us"
    },
    "slug": "post-season"
  },
  "types": {
    "$ref": "http://sports.core.api.espn.com/v2/sports/basketball/leagues/mens-college-basketball/seasons/2025/types?lang=en&region=us",
    "count": 4,
    "pageIndex": 1,
    "pageSize": 4,
    "pageCount": 1,
    "items": [
      // Season types listed here with details...
    ]
  },
  "rankings": {
    "$ref": "http://sports.core.api.espn.com/v2/sports/basketball/leagues/mens-college-basketball/seasons/2025/rankings?lang=en&region=us"
  },
  "powerIndexes": {
    "$ref": "http://sports.core.api.espn.com/v2/sports/basketball/leagues/mens-college-basketball/seasons/2025/powerindex?lang=en&region=us"
  },
  "powerIndexLeaders": {
    "$ref": "http://sports.core.api.espn.com/v2/sports/basketball/leagues/mens-college-basketball/seasons/2025/powerindex/leaders?lang=en&region=us"
  },
  "athletes": {
    "$ref": "http://sports.core.api.espn.com/v2/sports/basketball/leagues/mens-college-basketball/seasons/2025/athletes?lang=en&region=us"
  },
  "awards": {
    "$ref": "http://sports.core.api.espn.com/v2/sports/basketball/leagues/mens-college-basketball/seasons/2025/awards?lang=en&region=us"
  },
  "futures": {
    "$ref": "http://sports.core.api.espn.com/v2/sports/basketball/leagues/mens-college-basketball/seasons/2025/futures?lang=en&region=us"
  },
  "leaders": {
    "$ref": "http://sports.core.api.espn.com/v2/sports/basketball/leagues/mens-college-basketball/seasons/2025/types/2/leaders?lang=en&region=us"
  }
}
```

**Notes:**
- Historical seasons (like 1939) have fewer fields than recent seasons
- Power index data is only available for more recent seasons
- The current active season type is shown in the `type` field
- All season types are listed in the `types` object

## Season Types

Each season is divided into four types:

1. **Preseason** (type 1): Before the regular season begins
2. **Regular Season** (type 2): Main competitive season
3. **Postseason** (type 3): Tournament play after regular season
4. **Off Season** (type 4): Period between seasons

!!! note
    Older seasons may have fewer season types available. For example, the 1939 season only has the Postseason type.

### Season Type Endpoint

#### `/seasons/{year}/types/{type}`

Returns information about a specific season type.

**Path Parameters:**
- `year`: Season year
- `type`: Season type ID (1-4)

**Response Fields:**
- `id`: Type ID as string
- `type`: Type ID as number
- `name`: Human-readable name
- `abbreviation`: Short code
- `year`: Season year
- `startDate`: ISO 8601 formatted start date
- `endDate`: ISO 8601 formatted end date
- `hasGroups`: Whether this type has group data (conferences)
- `hasStandings`: Whether this type has standings data
- `hasLegs`: Whether this type has legs data
- `groups`: Reference to groups endpoint
- `week`: Current active week (if applicable)
- `weeks`: Reference to weeks endpoint
- `leaders`: Reference to statistical leaders (if available)
- `slug`: URL-friendly name

### Season Types List Endpoint

#### `/seasons/{year}/types`

Returns all season types for a given season.

**Example Request:**
```bash
curl -s "https://sports.core.api.espn.com/v2/sports/basketball/leagues/mens-college-basketball/seasons/2025/types" | jq
```

## Week Endpoints

### `/seasons/{year}/types/{type}/weeks`

Returns all weeks for a specific season type.

**Example Request:**
```bash
curl -s "https://sports.core.api.espn.com/v2/sports/basketball/leagues/mens-college-basketball/seasons/2025/types/3/weeks" | jq
```

**Example Response:**
```json
{
  "count": 3,
  "pageIndex": 1,
  "pageSize": 25,
  "pageCount": 1,
  "items": [
    {
      "$ref": "http://sports.core.api.espn.com/v2/sports/basketball/leagues/mens-college-basketball/seasons/2025/types/3/weeks/1?lang=en&region=us"
    },
    {
      "$ref": "http://sports.core.api.espn.com/v2/sports/basketball/leagues/mens-college-basketball/seasons/2025/types/3/weeks/2?lang=en&region=us"
    },
    {
      "$ref": "http://sports.core.api.espn.com/v2/sports/basketball/leagues/mens-college-basketball/seasons/2025/types/3/weeks/3?lang=en&region=us"
    }
  ]
}
```

### `/seasons/{year}/types/{type}/weeks/{week}`

Returns information about a specific week.

**Path Parameters:**
- `year`: Season year
- `type`: Season type ID (1-4)
- `week`: Week number

**Response Fields:**
- `number`: Week number
- `startDate`: ISO 8601 formatted start date
- `endDate`: ISO 8601 formatted end date
- `text`: Human-readable text (e.g., "Week 1")
- `rankings`: Reference to rankings endpoint

**Example Request:**
```bash
curl -s "https://sports.core.api.espn.com/v2/sports/basketball/leagues/mens-college-basketball/seasons/2025/types/3/weeks/1" | jq
```

**Example Response:**
```json
{
  "$ref": "http://sports.core.api.espn.com/v2/sports/basketball/leagues/mens-college-basketball/seasons/2025/types/3/weeks/1?lang=en&region=us",
  "number": 1,
  "startDate": "2025-03-18T07:00Z",
  "endDate": "2025-03-24T06:59Z",
  "text": "Week 1",
  "rankings": {
    "$ref": "http://sports.core.api.espn.com/v2/sports/basketball/leagues/mens-college-basketball/seasons/2025/types/3/weeks/1/rankings?lang=en&region=us"
  }
}
```

### `/seasons/{year}/types/{type}/weeks/{week}/events`

Returns all events (games) for a specific week.

**Example Request:**
```bash
curl -s "https://sports.core.api.espn.com/v2/sports/basketball/leagues/mens-college-basketball/seasons/2025/types/3/weeks/1/events" | jq
```

**Example Response:**
```json
{
  "$meta": {
    "parameters": {
      "season": ["2025"],
      "seasontypes": ["3"],
      "dates": ["1"]
    }
  },
  "count": 80,
  "pageIndex": 1,
  "pageSize": 25,
  "pageCount": 4,
  "items": [
    {
      "$ref": "http://sports.core.api.espn.com/v2/sports/basketball/leagues/mens-college-basketball/events/401745911?lang=en&region=us"
    },
    // Additional events...
  ]
}
```

### `/seasons/{year}/types/{type}/weeks/{week}/rankings`

Returns rankings for a specific week. Note that not all weeks have rankings data.

**Example Request:**
```bash
curl -s "https://sports.core.api.espn.com/v2/sports/basketball/leagues/mens-college-basketball/seasons/2025/types/3/weeks/1/rankings" | jq
```

## Group and Conference Endpoints

### `/seasons/{year}/types/{type}/groups`

Returns all groups (divisions, conferences) for a specific season type. For NCAA basketball, this includes all conferences and divisions.

**Example Request:**
```bash
curl -s "https://sports.core.api.espn.com/v2/sports/basketball/leagues/mens-college-basketball/seasons/2025/types/2/groups?limit=3" | jq
```

**Example Response:**
```json
{
  "count": 2,
  "pageIndex": 1,
  "pageSize": 3,
  "pageCount": 1,
  "items": [
    {
      "$ref": "http://sports.core.api.espn.com/v2/sports/basketball/leagues/mens-college-basketball/seasons/2025/types/2/groups/50?lang=en&region=us"
    },
    {
      "$ref": "http://sports.core.api.espn.com/v2/sports/basketball/leagues/mens-college-basketball/seasons/2025/types/2/groups/51?lang=en&region=us"
    }
  ]
}
```

### `/seasons/{year}/types/{type}/groups/{group_id}`

Returns detailed information about a specific conference or division.

**Response Fields:**
- `id`: Group ID
- `uid`: Universal ID 
- `name`: Full name of the group (e.g., "NCAA Division I")
- `abbreviation`: Group abbreviation
- `shortName`: Shortened group name
- `midsizeName`: Medium-length group name
- `season`: Reference to season
- `children`: Reference to child groups (e.g., conferences within a division)
- `parent`: Reference to parent group (if applicable)
- `standings`: Reference to standings
- `isConference`: Boolean indicating if this is a conference
- `slug`: URL-friendly name
- `teams`: Reference to teams in this group

**Example Request:**
```bash
curl -s "https://sports.core.api.espn.com/v2/sports/basketball/leagues/mens-college-basketball/seasons/2025/types/2/groups/46" | jq
```

**Example Response:**
```json
{
  "$ref": "http://sports.core.api.espn.com/v2/sports/basketball/leagues/mens-college-basketball/seasons/2025/types/2/groups/46?lang=en&region=us",
  "uid": "s:40~l:41~g:46",
  "id": "46",
  "name": "ASUN Conference",
  "abbreviation": "a-sun",
  "shortName": "ASUN",
  "midsizeName": "ASUN",
  "season": {
    "$ref": "http://sports.core.api.espn.com/v2/sports/basketball/leagues/mens-college-basketball/seasons/2025?lang=en&region=us"
  },
  "parent": {
    "$ref": "http://sports.core.api.espn.com/v2/sports/basketball/leagues/mens-college-basketball/seasons/2025/types/2/groups/50?lang=en&region=us"
  },
  "standings": {
    "$ref": "http://sports.core.api.espn.com/v2/sports/basketball/leagues/mens-college-basketball/seasons/2025/types/2/groups/46/standings?lang=en&region=us"
  },
  "isConference": true,
  "slug": "asun-conference",
  "teams": {
    "$ref": "http://sports.core.api.espn.com/v2/sports/basketball/leagues/mens-college-basketball/seasons/2025/types/2/groups/46/teams?lang=en&region=us"
  }
}
```

### `/seasons/{year}/types/{type}/groups/{group_id}/teams`

Returns teams within a specific conference or division.

**Example Request:**
```bash
curl -s "https://sports.core.api.espn.com/v2/sports/basketball/leagues/mens-college-basketball/seasons/2025/types/2/groups/46/teams?limit=3" | jq
```

**Example Response:**
```json
{
  "count": 12,
  "pageIndex": 1,
  "pageSize": 3,
  "pageCount": 4,
  "items": [
    {
      "$ref": "http://sports.core.api.espn.com/v2/sports/basketball/leagues/mens-college-basketball/seasons/2025/teams/56?lang=en&region=us"
    },
    {
      "$ref": "http://sports.core.api.espn.com/v2/sports/basketball/leagues/mens-college-basketball/seasons/2025/teams/91?lang=en&region=us"
    },
    {
      "$ref": "http://sports.core.api.espn.com/v2/sports/basketball/leagues/mens-college-basketball/seasons/2025/teams/288?lang=en&region=us"
    }
  ]
}
```

## Statistical Leaders Endpoints

### `/seasons/{year}/types/{type}/leaders`

Returns statistical leaders for a specific season type, including points per game, rebounds, assists, and other categories.

**Example Request:**
```bash
curl -s "https://sports.core.api.espn.com/v2/sports/basketball/leagues/mens-college-basketball/seasons/2025/types/2/leaders?limit=3" | jq
```

**Example Response:**
```json
{
  "$ref": "http://sports.core.api.espn.com/v2/sports/basketball/leagues/mens-college-basketball/seasons/2025/types/2/leaders?lang=en&region=us",
  "id": "0",
  "name": "Season",
  "abbreviation": "Season",
  "type": "season",
  "categories": [
    {
      "name": "pointsPerGame",
      "displayName": "Points Per Game",
      "shortDisplayName": "PPG",
      "abbreviation": "PTS",
      "leaders": [
        {
          "displayValue": "32.0",
          "value": 32.0,
          "rel": ["athlete"],
          "athlete": {
            "$ref": "http://sports.core.api.espn.com/v2/sports/basketball/leagues/mens-college-basketball/seasons/2025/athletes/5252523?lang=en&region=us"
          },
          "team": {
            "$ref": "http://sports.core.api.espn.com/v2/sports/basketball/leagues/mens-college-basketball/seasons/2025/teams/3106?lang=en&region=us"
          },
          "statistics": {
            "$ref": "http://sports.core.api.espn.com/v2/sports/basketball/leagues/mens-college-basketball/seasons/2025/types/2/athletes/5252523/statistics/0?lang=en&region=us"
          }
        },
        // Additional leaders...
      ]
    },
    {
      "name": "assistsPerGame",
      "displayName": "Assists Per Game",
      "shortDisplayName": "APG",
      "abbreviation": "AST",
      "leaders": [
        // Leaders listed here...
      ]
    },
    // Additional statistical categories...
  ]
}
```

**Response Structure:**
- The response contains multiple statistical categories
- Each category has a list of leaders with their values and references to their athlete and team profiles
- Categories include:
  - Points per game
  - Assists per game
  - Field goal percentage
  - Rebounds per game
  - Steals per game
  - Blocks per game
  - Free throw percentage
  - 3-point percentage
  - Player efficiency rating
  - 3-pointers made per game
  - Double-doubles
  - Minutes per game
  - Fouls per game
  - Total points

## Rankings Endpoints

### `/seasons/{year}/rankings`

Returns a list of available rankings systems for the season.

**Example Request:**
```bash
curl -s "https://sports.core.api.espn.com/v2/sports/basketball/leagues/mens-college-basketball/seasons/2025/rankings?limit=3" | jq
```

**Example Response:**
```json
{
  "count": 2,
  "pageIndex": 1,
  "pageSize": 3,
  "pageCount": 1,
  "items": [
    {
      "$ref": "http://sports.core.api.espn.com/v2/sports/basketball/leagues/mens-college-basketball/seasons/2025/rankings/1?lang=en&region=us"
    },
    {
      "$ref": "http://sports.core.api.espn.com/v2/sports/basketball/leagues/mens-college-basketball/seasons/2025/rankings/2?lang=en&region=us"
    }
  ]
}
```

### `/seasons/{year}/rankings/{ranking_id}`

Returns detailed ranking information for a specific ranking system (e.g., AP Top 25).

**Example Request:**
```bash
curl -s "https://sports.core.api.espn.com/v2/sports/basketball/leagues/mens-college-basketball/seasons/2025/rankings/1" | jq
```

**Example Response:**
```json
{
  "$ref": "http://sports.core.api.espn.com/v2/sports/basketball/leagues/mens-college-basketball/seasons/2025/rankings/1?lang=en&region=us",
  "id": "1",
  "name": "AP Top 25",
  "shortName": "AP Poll",
  "type": "ap",
  "rankings": [
    {
      "$ref": "http://sports.core.api.espn.com/v2/sports/basketball/leagues/mens-college-basketball/seasons/2025/types/2/weeks/1/rankings/1?lang=en&region=us"
    },
    {
      "$ref": "http://sports.core.api.espn.com/v2/sports/basketball/leagues/mens-college-basketball/seasons/2025/types/2/weeks/2/rankings/1?lang=en&region=us"
    },
    // Rankings for each week...
  ]
}
```

### `/seasons/{year}/types/{type}/weeks/{week}/rankings/{ranking_id}`

Returns detailed ranking data for a specific week.

**Example Request:**
```bash
curl -s "https://sports.core.api.espn.com/v2/sports/basketball/leagues/mens-college-basketball/seasons/2025/types/2/weeks/20/rankings/1" | jq
```

**Response Fields:**
- `current`: Current rank
- `previous`: Previous rank
- `points`: Ranking points
- `firstPlaceVotes`: Number of first-place votes
- `trend`: Rank change indicator
- `record`: Team's won-loss record
- `team`: Reference to team
- `date`: Ranking date
- `lastUpdated`: Last updated date

## Power Index Endpoints

### `/seasons/{year}/powerindex`

Returns power index data for teams in the season, which is ESPN's Basketball Power Index (BPI) metric.

**Example Request:**
```bash
curl -s "https://sports.core.api.espn.com/v2/sports/basketball/leagues/mens-college-basketball/seasons/2025/powerindex?limit=5" | jq
```

**Response Structure:**
- Teams are ranked by the BPI metric, which combines offensive and defensive strength
- Each team has comprehensive statistics including:
  - `bpi`: Basketball Power Index value
  - `bpirank`: BPI rank
  - `sor`: Strength of Record
  - `sorrank`: Strength of Record rank
  - `sospast`: Strength of Schedule
  - `sospastrank`: Strength of Schedule rank
  - `wins`, `losses`, `winpct`: Win-loss record
  - `bpioffense`, `bpidefense`: Offensive and defensive BPI components
  - `projectedtournamentseed`: Projected NCAA Tournament seed
  - `chancefinal4`, `chancencaachampion`: Probabilities for tournament success

### `/seasons/{year}/powerindex/leaders`

Returns the leaders in various power index categories.

**Example Request:**
```bash
curl -s "https://sports.core.api.espn.com/v2/sports/basketball/leagues/mens-college-basketball/seasons/2025/powerindex/leaders?limit=3" | jq
```

**Example Response:**
```json
{
  "count": 9,
  "pageIndex": 1,
  "pageSize": 3,
  "pageCount": 3,
  "items": [
    {
      "name": "bpi",
      "displayName": "BPI Leader",
      "leaders": [
        {
          "displayValue": "25",
          "shortDisplayValue": "33-3",
          "value": 24.9863,
          "team": {
            "$ref": "http://sports.core.api.espn.com/v2/sports/basketball/leagues/mens-college-basketball/seasons/2025/teams/150?lang=en&region=us"
          }
        }
      ]
    },
    {
      "name": "rpirank",
      "displayName": "NCAAM RPI Leader",
      "leaders": [
        {
          "shortDisplayValue": "33-3",
          "team": {
            "$ref": "http://sports.core.api.espn.com/v2/sports/basketball/leagues/mens-college-basketball/seasons/2025/teams/150?lang=en&region=us"
          }
        }
      ]
    },
    {
      "name": "sospast",
      "displayName": "SOS Leader",
      "leaders": [
        {
          "displayValue": "0.2",
          "shortDisplayValue": "27-8",
          "value": 0.179,
          "team": {
            "$ref": "http://sports.core.api.espn.com/v2/sports/basketball/leagues/mens-college-basketball/seasons/2025/teams/333?lang=en&region=us"
          }
        }
      ]
    }
  ]
}
```

## Awards Endpoints

### `/seasons/{year}/awards`

Returns a list of awards given for the season, such as All-American teams and individual honors.

**Example Request:**
```bash
curl -s "https://sports.core.api.espn.com/v2/sports/basketball/leagues/mens-college-basketball/seasons/2025/awards?limit=3" | jq
```

**Example Response:**
```json
{
  "count": 1,
  "pageIndex": 1,
  "pageSize": 3,
  "pageCount": 1,
  "items": [
    {
      "$ref": "http://sports.core.api.espn.com/v2/sports/basketball/leagues/mens-college-basketball/seasons/2025/awards/342?lang=en&region=us"
    }
  ]
}
```

### `/seasons/{year}/awards/{award_id}`

Returns detailed information about a specific award, including the winners.

**Example Request:**
```bash
curl -s "https://sports.core.api.espn.com/v2/sports/basketball/leagues/mens-college-basketball/seasons/2025/awards/342" | jq
```

**Example Response:**
```json
{
  "$ref": "http://sports.core.api.espn.com/v2/sports/basketball/leagues/mens-college-basketball/seasons/2025/awards/342?lang=en&region=us",
  "id": "342",
  "name": "1st Team AP All-American",
  "season": {
    "$ref": "http://sports.core.api.espn.com/v2/sports/basketball/leagues/mens-college-basketball/seasons/2025?lang=en&region=us"
  },
  "winners": [
    {
      "athlete": {
        "$ref": "http://sports.core.api.espn.com/v2/sports/basketball/leagues/mens-college-basketball/seasons/2025/athletes/4433569?lang=en&region=us"
      },
      "team": {
        "$ref": "http://sports.core.api.espn.com/v2/sports/basketball/leagues/mens-college-basketball/seasons/2025/teams/2?lang=en&region=us"
      }
    },
    // Additional award winners...
  ],
  "links": [
    {
      "language": "en-US",
      "rel": ["index", "desktop", "awards"],
      "href": "https://www.espn.com/mens-college-basketball/awards/_/id/342",
      "text": "Awards",
      "shortText": "Awards",
      "isExternal": false,
      "isPremium": false
    }
  ]
}
```

## Futures/Betting Endpoints

### `/seasons/{year}/futures`

Returns betting futures and odds for the season, including tournament and conference champions.

**Example Request:**
```bash
curl -s "https://sports.core.api.espn.com/v2/sports/basketball/leagues/mens-college-basketball/seasons/2025/futures?limit=3" | jq
```

**Example Response:**
```json
{
  "count": 33,
  "pageIndex": 1,
  "pageSize": 3,
  "pageCount": 11,
  "items": [
    {
      "$ref": "http://sports.core.api.espn.com/v2/sports/basketball/leagues/mens-college-basketball/seasons/2025/futures/6099?lang=en&region=us",
      "id": 6099,
      "name": "NCAA(B) - West Region",
      "futures": [
        {
          "provider": {
            "id": "58",
            "name": "ESPN BET",
            "active": 1,
            "priority": 1
          },
          "books": [
            {
              "team": {
                "$ref": "http://sports.core.api.espn.com/v2/sports/basketball/leagues/mens-college-basketball/seasons/2025/teams/57?lang=en&region=us"
              },
              "value": "-135"
            },
            // Additional teams with odds...
          ]
        }
      ],
      "displayName": "NCAA(B) - West Region"
    },
    // Additional futures markets...
  ]
}
```

**Response Structure:**
- Each item represents a futures betting market (e.g., regional champion, conference tournament winner)
- Each market contains:
  - `id`: Market ID
  - `name`: Market name
  - `futures`: Array of providers and their odds
  - `displayName`: Display name for the market
- Within each provider's `books` array, each entry contains:
  - `team`: Reference to the team
  - `value`: Odds value (American format)

## Athletes Endpoint

### `/seasons/{year}/athletes`

Returns a paginated list of athletes for the season.

**Query Parameters:**
- `limit` (optional): Number of items to return

**Example Request:**
```bash
curl -s "https://sports.core.api.espn.com/v2/sports/basketball/leagues/mens-college-basketball/seasons/2025/athletes?limit=1" | jq
```

**Example Response:**
```json
{
  "count": 6172,
  "pageIndex": 1,
  "pageSize": 1,
  "pageCount": 6172,
  "items": [
    {
      "$ref": "http://sports.core.api.espn.com/v2/sports/basketball/leagues/mens-college-basketball/seasons/2025/athletes/23574?lang=en&region=us"
    }
  ]
}
```

**Notes:**
- Recent seasons have thousands of athletes (6,172 for the 2025 season)
- Historical seasons may have limited or no athlete data (the 1939 season has 0 athletes)

### `/seasons/{year}/athletes/{athlete_id}`

Returns detailed information about a specific athlete for the season.

**Example Response Fields:**
- `id`: Athlete ID
- `uid`: Universal ID
- `guid`: Global unique ID
- `type`: Sport type
- `alternateIds`: Other system IDs
- `firstName`, `lastName`, `fullName`, `displayName`, `shortName`: Name variants
- `weight`, `displayWeight`: Weight information
- `height`, `displayHeight`: Height information
- `dateOfBirth`: Date of birth
- `links`: Array of related links
- `birthPlace`: Place of birth
- `birthCountry`: Country of birth
- `college`: College reference
- `slug`: URL-friendly name
- `headshot`: Player image
- `jersey`: Jersey number
- `flag`: Country flag
- `position`: Player position
- `injuries`: Array of injuries
- `linked`: Boolean indicating if athlete is linked
- `team`: Current team
- `teams`: All teams
- `statistics`: Reference to statistics
- `notes`: Reference to notes
- `experience`: College year information
- `active`: Whether player is active
- `eventLog`: Reference to game log
- `status`: Player status

**Example Request:**
```bash
curl -s "https://sports.core.api.espn.com/v2/sports/basketball/leagues/mens-college-basketball/seasons/2025/athletes/23574" | jq
```

## Teams Endpoint

### `/seasons/{year}/teams`

Returns a paginated list of teams for the season.

**Example Request:**
```bash
curl -s "https://sports.core.api.espn.com/v2/sports/basketball/leagues/mens-college-basketball/seasons/2025/teams?limit=1" | jq
```

**Example Response:**
```json
{
  "count": 914,
  "pageIndex": 1,
  "pageSize": 1,
  "pageCount": 914,
  "items": [
    {
      "$ref": "http://sports.core.api.espn.com/v2/sports/basketball/leagues/mens-college-basketball/seasons/2025/teams/1?lang=en&region=us"
    }
  ]
}
```

**Notes:**
- Recent seasons have hundreds of teams (914 for the 2025 season)

### `/seasons/{year}/teams/{team_id}`

Returns detailed information about a specific team for the season.

**Example Response Fields:**
- `id`: Team ID
- `guid`: Global unique ID
- `uid`: Universal ID
- `alternateIds`: Other system IDs
- `slug`: URL-friendly name
- `location`: Team location name
- `name`: Team name
- `nickname`: Short team name
- `abbreviation`: Team abbreviation 
- `displayName`: Full team name
- `shortDisplayName`: Shortened display name
- `color`: Primary color (hex)
- `isActive`: Whether team is active
- `isAllStar`: Whether team is an all-star team
- `logos`: Array of team logos
- `record`: Reference to team record
- `athletes`: Reference to team athletes
- `venue`: Team venue information
- `groups`: Reference to conference/group
- `ranks`: Reference to team rankings
- `links`: Array of related links
- `injuries`: Reference to team injuries
- `notes`: Reference to team notes
- `againstTheSpreadRecords`: Reference to betting records
- `awards`: Reference to team awards
- `franchise`: Reference to team franchise
- `college`: Reference to college

**Example Request:**
```bash
curl -s "https://sports.core.api.espn.com/v2/sports/basketball/leagues/mens-college-basketball/seasons/2025/teams/1" | jq
```

## Events Endpoint

Although not directly a season endpoint, the events endpoint provides game data for specific weeks within a season:

### `/seasons/{year}/types/{type}/weeks/{week}/events`

Returns all events (games) for a specific week.

**Example Event Response Fields:**
- `id`: Event ID
- `uid`: Universal ID
- `date`: Game date and time
- `name`: Game name (e.g., "North Carolina Tar Heels at San Diego State Aztecs")
- `shortName`: Shortened game name
- `season`: Season reference
- `seasonType`: Season type reference
- `week`: Week reference
- `timeValid`: Whether time is valid
- `competitions`: Array of competition details
- `links`: Array of related links
- `venues`: Array of venues
- `league`: League reference
- `tournament`: Tournament reference

## Season Structure and Availability

Based on the API responses, we can observe:

1. **Season range**: Data is available from the 1939 season (1939-1940) to the 2025 season (2024-2025)
2. **Data completeness**: More recent seasons have more complete data
   - Recent seasons have 4 season types (Preseason, Regular Season, Postseason, Off Season)
   - Older seasons may only have Postseason data
   - Athlete data is more complete for recent seasons
3. **Statistical coverage**:
   - Power index data only exists for recent seasons
   - Historical seasons have limited statistical information

## Data Relationships

The season endpoints create a hierarchical structure:

```
Season
  └── Season Types (1-4)
       ├── Groups (Conferences/Divisions)
       |    └── Teams
       └── Weeks
            └── Events (Games)
                 └── Teams
                      └── Athletes
```

Each level provides references to related data through `$ref` fields, allowing for detailed exploration of the entire NCAA Men's Basketball ecosystem.

## Specific Ranking System Endpoint

### `/seasons/{year}/types/{type}/weeks/{week}/rankings/{ranking_id}`

Returns detailed ranking information for a specific ranking system in a specific week.

**Path Parameters:**
- `year`: Season year (e.g., 2024)
- `type`: Season type ID (1=Preseason, 2=Regular Season, 3=Postseason, 4=Off Season)
- `week`: Week number within the season type
- `ranking_id`: ID of the ranking system (1=AP Top 25, 2=Coaches Poll)

**Common Ranking ID Values:**
| Ranking ID | Description |
|------------|-------------|
| `1` | AP Top 25 |
| `2` | USA Today Coaches Poll |
| `3` | NET Rankings |
| `4` | ESPN Daily RPI |

**Response Structure:**
- `id`: Unique identifier for the ranking system
- `name`: Full name of the ranking system (e.g., "AP Top 25")
- `shortName`: Abbreviated name (e.g., "AP Poll")
- `type`: Type code for the ranking system
- `occurrence`: Object with details about the week
- `date`: ISO-8601 formatted date when the rankings were released
- `headline`: Full headline for the rankings
- `lastUpdated`: ISO-8601 timestamp of when the data was last updated
- `ranks`: Array of ranked teams, with each entry containing:
  - `current`: Current rank position
  - `previous`: Previous rank position
  - `points`: Vote points received
  - `firstPlaceVotes`: Number of first-place votes
  - `trend`: Change in ranking ("+1", "-2", or "-" for unchanged)
  - `record`: Object containing team's record with wins and losses
  - `team`: Reference to detailed team information
  - `date`: ISO-8601 date when this ranking was effective
  - `lastUpdated`: Last update timestamp for this rank

**Example Request:**
```bash
curl -s "https://sports.core.api.espn.com/v2/sports/basketball/leagues/mens-college-basketball/seasons/2024/types/2/weeks/18/rankings/1" | jq
```

**Example Response (Partial):**
```json
{
  "$ref": "http://sports.core.api.espn.com/v2/sports/basketball/leagues/mens-college-basketball/seasons/2024/types/2/weeks/18/rankings/1?lang=en&region=us",
  "id": "1",
  "name": "AP Top 25",
  "shortName": "AP Poll",
  "type": "ap",
  "occurrence": {
    "number": 18,
    "type": "week",
    "last": false,
    "value": "18",
    "displayValue": "Week 18"
  },
  "date": "2024-03-04T08:00Z",
  "headline": "2024 NCAA Basketball Rankings - AP Top 25 Week 18",
  "shortHeadline": "2024 : Week 18",
  "season": {
    // Season details omitted for brevity
  },
  "lastUpdated": "2024-03-04T21:06Z",
  "ranks": [
    {
      "current": 1,
      "previous": 1,
      "points": 1539.0,
      "firstPlaceVotes": 52,
      "trend": "-",
      "record": {
        "summary": "26-3",
        "stats": [
          {
            "name": "wins",
            "displayName": "Wins",
            "shortDisplayName": "W",
            "description": "Wins",
            "abbreviation": "W",
            "type": "wins",
            "value": 26.0,
            "displayValue": "26"
          },
          {
            "name": "losses",
            "displayName": "Losses",
            "shortDisplayName": "L",
            "description": "Losses",
            "abbreviation": "L",
            "type": "losses",
            "value": 3.0,
            "displayValue": "3"
          }
        ]
      },
      "team": {
        "$ref": "http://sports.core.api.espn.com/v2/sports/basketball/leagues/mens-college-basketball/seasons/2024/teams/248?lang=en&region=us"
      },
      "date": "2024-03-04T08:00Z",
      "lastUpdated": "2024-03-04T21:06Z"
    },
    // Additional ranked teams omitted for brevity
  ]
}
```

**Notes:**
- The response includes every ranked team (25 for AP Poll) with comprehensive information about their ranking.
- The `trend` field shows the change in ranking from the previous poll with "+" for improvement, "-" for decline.
- `firstPlaceVotes` shows how many voters ranked this team #1 in the poll.
- The `points` field represents the weighted voting score used to determine the rankings.
- Historical rankings are available by specifying previous weeks.

!!! tip "Retrieving Complete Rankings"
    This endpoint can be used to get the complete rankings for a specific poll. The `ranks` array contains every team that received votes, including those outside the top 25 that received votes.

!!! info "Difference Between Ranking Systems"
    Different ranking systems use different methodologies:
    - AP Top 25: Human-voted poll from media members
    - Coaches Poll: Human-voted poll from coaches
    - NET Rankings: NCAA's primary sorting tool using game results, strength of schedule, etc.
    - RPI: Rating Percentage Index based on a team's wins/losses and strength of schedule
