# Team Endpoints

!!! info "API Base URL"
    The base URL for all endpoints is:
    ```
    https://sports.core.api.espn.com/v2/sports/basketball/leagues/mens-college-basketball
    ```

## Team List Endpoint

| Endpoint | Description |
|----------|-------------|
| `/seasons/{year}/teams` | List of teams in the season |

### Query Parameters

| Parameter | Description | Example |
|-----------|-------------|---------|
| `limit` | Number of teams to return | `?limit=100` |
| `active` | Filter for active teams | `?active=true` |
| `lang` | Language for the response | `?lang=en` |
| `region` | Region for the response | `?region=us` |

### Response Format

The response is a paginated list of team references.

!!! example "Example Response"
    ```json
    {
      "count": 925,
      "pageIndex": 1,
      "pageSize": 1,
      "pageCount": 925,
      "items": [
        {
          "$ref": "http://sports.core.api.espn.com/v2/sports/basketball/leagues/mens-college-basketball/seasons/2024/teams/1?lang=en&region=us"
        }
      ]
    }
    ```

## Team Details Endpoint

| Endpoint | Description |
|----------|-------------|
| `/seasons/{year}/teams/{team_id}` | Information about a specific team |

### Path Parameters

| Parameter | Description | Example |
|-----------|-------------|---------|
| `year` | Season year | `2024` |
| `team_id` | Unique identifier for the team | `52` (North Carolina) |

### Response Format

The response contains comprehensive information about the team, including:
- Basic information (name, colors, abbreviation)
- Links to other team-related endpoints
- Venue information
- Conference/group affiliation
- Logo URLs

!!! example "Example Response"
    ```json
    {
      "id": "1",
      "guid": "761435bb-bde2-29c0-6411-0371be8dc4b0",
      "uid": "s:40~l:41~t:1",
      "alternateIds": {
        "sdr": "5164"
      },
      "slug": "alaska-anchorage-seawolves",
      "location": "Alaska Anchorage",
      "name": "Seawolves",
      "nickname": "AK-Anchorage",
      "abbreviation": "UAA",
      "displayName": "Alaska Anchorage Seawolves",
      "shortDisplayName": "AK-Anchorage",
      "color": "000000",
      "isActive": false,
      "isAllStar": false,
      "logos": [
        {
          "href": "https://a.espncdn.com/i/teamlogos/ncaa/500/1.png",
          "width": 500,
          "height": 500,
          "alt": "",
          "rel": [
            "full",
            "default"
          ],
          "lastUpdated": "2019-11-05T17:36Z"
        },
        {
          "href": "https://a.espncdn.com/i/teamlogos/ncaa/500-dark/1.png",
          "width": 500,
          "height": 500,
          "alt": "",
          "rel": [
            "full",
            "dark"
          ],
          "lastUpdated": "2019-11-05T17:38Z"
        }
      ],
      "record": {
        "$ref": "http://sports.core.api.espn.com/v2/sports/basketball/leagues/mens-college-basketball/seasons/2024/types/3/teams/1/record?lang=en&region=us"
      },
      "athletes": {
        "$ref": "http://sports.core.api.espn.com/v2/sports/basketball/leagues/mens-college-basketball/seasons/2024/teams/1/athletes?lang=en&region=us"
      },
      "venue": {
        "$ref": "http://sports.core.api.espn.com/v2/sports/basketball/leagues/mens-college-basketball/venues/4823?lang=en&region=us",
        "id": "4823",
        "fullName": "Alaska Airlines Center",
        "address": {
          "city": "Anchorage",
          "state": "AK"
        },
        "grass": false,
        "indoor": true,
        "images": []
      },
      "groups": {
        "$ref": "http://sports.core.api.espn.com/v2/sports/basketball/leagues/mens-college-basketball/seasons/2024/types/3/groups/51?lang=en&region=us"
      },
      "links": [
        {
          "language": "en-US",
          "rel": [
            "roster",
            "desktop",
            "team"
          ],
          "href": "https://www.espn.com/mens-college-basketball/team/roster/_/id/1",
          "text": "Roster",
          "shortText": "Roster",
          "isExternal": false,
          "isPremium": false
        }
        // Additional links omitted for brevity
      ]
    }
    ```

## Team Athletes Endpoint

| Endpoint | Description |
|----------|-------------|
| `/seasons/{year}/teams/{team_id}/athletes` | Athletes on the team |

### Path Parameters

| Parameter | Description | Example |
|-----------|-------------|---------|
| `year` | Season year | `2024` |
| `team_id` | Unique identifier for the team | `52` (North Carolina) |

### Query Parameters

| Parameter | Description | Example |
|-----------|-------------|---------|
| `limit` | Number of athletes to return | `?limit=25` |

### Response Format

The response is a paginated list of athlete references.

!!! example "Example Response"
    ```json
    {
      "count": 27,
      "pageIndex": 1,
      "pageSize": 2,
      "pageCount": 14,
      "items": [
        {
          "$ref": "http://sports.core.api.espn.com/v2/sports/basketball/leagues/mens-college-basketball/seasons/2024/athletes/5114809?lang=en&region=us"
        },
        {
          "$ref": "http://sports.core.api.espn.com/v2/sports/basketball/leagues/mens-college-basketball/seasons/2024/athletes/4899784?lang=en&region=us"
        }
      ]
    }
    ```

## Team Events Endpoint

| Endpoint | Description |
|----------|-------------|
| `/seasons/{year}/teams/{team_id}/events` | Team's schedule/results |

### Path Parameters

| Parameter | Description | Example |
|-----------|-------------|---------|
| `year` | Season year | `2024` |
| `team_id` | Unique identifier for the team | `52` (North Carolina) |

### Query Parameters

| Parameter | Description | Example |
|-----------|-------------|---------|
| `limit` | Number of events to return | `?limit=25` |

### Response Format

The response is a paginated list of event references.

!!! example "Example Response"
    ```json
    {
      "count": 33,
      "pageIndex": 1,
      "pageSize": 2,
      "pageCount": 17,
      "items": [
        {
          "$ref": "http://sports.core.api.espn.com/v2/sports/basketball/leagues/mens-college-basketball/events/401586904?lang=en&region=us"
        },
        {
          "$ref": "http://sports.core.api.espn.com/v2/sports/basketball/leagues/mens-college-basketball/events/401586905?lang=en&region=us"
        }
      ]
    }
    ```

## Team Coaches Endpoint

| Endpoint | Description |
|----------|-------------|
| `/seasons/{year}/teams/{team_id}/coaches` | Team coaches |

### Path Parameters

| Parameter | Description | Example |
|-----------|-------------|---------|
| `year` | Season year | `2024` |
| `team_id` | Unique identifier for the team | `52` (North Carolina) |

### Response Format

The response is a paginated list of coach references.

!!! note "API Behavior"
    While the endpoint exists, in our testing it returned an empty list. This endpoint may be under development or used for specific circumstances.

!!! example "Example Response"
    ```json
    {
      "count": 0,
      "pageIndex": 0,
      "pageSize": 25,
      "pageCount": 0,
      "items": []
    }
    ```

## Team Statistics Endpoints

There are two endpoints for team statistics:

| Endpoint | Description |
|----------|-------------|
| `/seasons/{year}/types/{type}/teams/{team_id}/statistics` | Team statistics for a specific season type |

### Path Parameters

| Parameter | Description | Example |
|-----------|-------------|---------|
| `year` | Season year | `2024` |
| `type` | Season type (1=Preseason, 2=Regular Season, 3=Postseason, 4=Off Season) | `2` |
| `team_id` | Unique identifier for the team | `52` (North Carolina) |

### Response Format

The response contains comprehensive team statistics organized into categories.

#### Statistics Categories
The statistics are organized into three main categories:

1. **Defensive** - Blocks, defensive rebounds, steals, etc.
2. **General** - Fouls, rebounds, minutes played, etc.
3. **Offensive** - Scoring, field goals, assists, etc.

Each statistic includes:
- Name and description
- Value (raw number)
- Display value (formatted for display)
- Rank (where applicable) - Team's position among all teams for this statistic

!!! example "Example Response (Partial)"
    ```json
    {
      "season": {
        "$ref": "http://sports.core.api.espn.com/v2/sports/basketball/leagues/mens-college-basketball/seasons/2024?lang=en&region=us"
      },
      "team": {
        "$ref": "http://sports.core.api.espn.com/v2/sports/basketball/leagues/mens-college-basketball/seasons/2024/teams/52?lang=en&region=us"
      },
      "splits": {
        "id": "0",
        "name": "Season",
        "abbreviation": "Season",
        "type": "season",
        "categories": [
          {
            "name": "defensive",
            "displayName": "Defensive",
            "shortDisplayName": "Defensive",
            "abbreviation": "def",
            "summary": "",
            "stats": [
              {
                "name": "blocks",
                "displayName": "Blocks",
                "shortDisplayName": "BLK",
                "description": "Short for blocked shot, number of times when a defensive player legally deflects a field goal attempt from an offensive player.",
                "abbreviation": "BLK",
                "value": 142.0,
                "displayValue": "142",
                "rank": 58,
                "rankDisplayValue": "Tied-58th"
              },
              // Additional statistics omitted for brevity
            ]
          },
          // Additional categories omitted for brevity
        ]
      },
      "seasonType": {
        "$ref": "http://sports.core.api.espn.com/v2/sports/basketball/leagues/mens-college-basketball/seasons/2024/types/2?lang=en&region=us"
      }
    }
    ```

## Team Record Endpoint

| Endpoint | Description |
|----------|-------------|
| `/seasons/{year}/types/{type}/teams/{team_id}/record` | Team record details for a season type |

### Path Parameters

| Parameter | Description | Example |
|-----------|-------------|---------|
| `year` | Season year | `2024` |
| `type` | Season type (1=Preseason, 2=Regular Season, 3=Postseason, 4=Off Season) | `2` |
| `team_id` | Unique identifier for the team | `52` (North Carolina) |

### Response Format

The response contains detailed record information categorized by different contexts:

1. **overall** - The team's overall record for the season
2. **home** - The team's record in home games
3. **away** - The team's record in away games
4. **vs AP Top 25** - The team's record against AP Top 25 ranked teams
5. **vs USA Today Top 25** - The team's record against USA Today ranked teams
6. **vs Conference** - The team's record in conference play

Each record contains the following key statistics:
- Win/loss summary (e.g., "17-16")
- Win percentage value
- Detailed statistics including:
  - Overtime wins and losses
  - Average points for and against
  - Point differential
  - Games behind leader
  - Games played
  - Winning streaks
  - Conference rankings

!!! example "Example Response (Partial)"
    ```json
    {
      "count": 6,
      "pageIndex": 1,
      "pageSize": 25,
      "pageCount": 1,
      "items": [
        {
          "$ref": "http://sports.core.api.espn.com/v2/sports/basketball/leagues/mens-college-basketball/seasons/2024/types/2/teams/52/records/0?lang=en&region=us",
          "id": "0",
          "name": "overall",
          "abbreviation": "Season",
          "displayName": "Team Season Record",
          "shortDisplayName": "Season",
          "description": "Overall Record",
          "type": "total",
          "summary": "17-16",
          "displayValue": "17-16",
          "value": 0.5151515151515151,
          "stats": [
            {
              "name": "OTLosses",
              "displayName": "Overtime Losses",
              "shortDisplayName": "OTL",
              "description": "Number of Overtime Losses",
              "abbreviation": "OTL",
              "type": "otlosses",
              "value": 0.0,
              "displayValue": "0"
            },
            // Additional statistics omitted for brevity
          ]
        },
        // Additional record types omitted for brevity
      ]
    }
    ```

## Team Ranks Endpoint

| Endpoint | Description |
|----------|-------------|
| `/seasons/{year}/teams/{team_id}/ranks` | Team ranking information |

### Path Parameters

| Parameter | Description | Example |
|-----------|-------------|---------|
| `year` | Season year | `2024` |
| `team_id` | Unique identifier for the team | `52` (North Carolina) |

### Response Format

The response is a paginated list of ranking references.

!!! note "API Behavior"
    Based on testing, this endpoint currently returns an empty list. The endpoint exists but may be under development or only populated for certain teams/seasons.

!!! example "Example Response"
    ```json
    {
      "count": 0,
      "pageIndex": 0,
      "pageSize": 25,
      "pageCount": 0,
      "items": []
    }
    ```

## Against The Spread (ATS) Endpoint

| Endpoint | Description |
|----------|-------------|
| `/seasons/{year}/types/{type}/teams/{team_id}/ats` | Against the spread records for a specific team |

### Path Parameters

| Parameter | Description | Example |
|-----------|-------------|---------|
| `year` | Season year | `2024` |
| `type` | Season type (1=Preseason, 2=Regular Season, 3=Postseason, 4=Off Season) | `2` |
| `team_id` | Unique identifier for the team | `52` (North Carolina) |

### Response Format

The response contains an array of ATS (Against The Spread) records categorized by different betting scenarios:

1. **atsOverall** - Overall record against the spread
2. **atsFavorite** - Record when the team is the favorite
3. **atsUnderdog** - Record when the team is the underdog
4. **atsAway** - Record in away games against the spread
5. **atsHome** - Record in home games against the spread
6. **atsAwayFavorite** - Record as the away favorite against the spread
7. **atsAwayUnderdog** - Record as the away underdog against the spread
8. **atsHomeFavorite** - Record as the home favorite against the spread
9. **atsHomeUnderdog** - Record as the home underdog against the spread

Each record includes:
- Number of wins against the spread
- Number of losses against the spread
- Number of pushes (ties)
- Type information with ID, name, and description

!!! example "Example Response"
    ```json
    {
      "count": 9,
      "pageIndex": 1,
      "pageSize": 25,
      "pageCount": 1,
      "items": [
        {
          "wins": 16,
          "losses": 16,
          "pushes": 1,
          "type": {
            "id": "0",
            "name": "atsOverall",
            "description": "Overall team season record against the spread"
          }
        },
        {
          "wins": 8,
          "losses": 6,
          "pushes": 1,
          "type": {
            "id": "1",
            "name": "atsFavorite",
            "description": "Team season record against the spread as the favorite"
          }
        },
        // Additional ATS record types omitted for brevity
      ]
    }
    ```

## Team Awards Endpoint

| Endpoint | Description |
|----------|-------------|
| `/seasons/{year}/teams/{team_id}/awards` | Awards received by a team in a specific season |

### Path Parameters

| Parameter | Description | Example |
|-----------|-------------|---------|
| `year` | Season year | `2024` |
| `team_id` | Unique identifier for the team | `52` (North Carolina) |

### Response Format

The response is a paginated list of award references.

!!! note "API Behavior"
    Based on testing, this endpoint currently returns an empty list. The endpoint exists but appears to only be populated for certain teams and seasons.

!!! example "Example Response"
    ```json
    {
      "count": 0,
      "pageIndex": 0,
      "pageSize": 25,
      "pageCount": 0,
      "items": []
    }
    ```

## Team Notes Endpoint

| Endpoint | Description |
|----------|-------------|
| `/teams/{team_id}/notes` | Notes related to a specific team |

### Path Parameters

| Parameter | Description | Example |
|-----------|-------------|---------|
| `team_id` | Unique identifier for the team | `52` (North Carolina) |

### Response Format

The response is a paginated list of note references.

!!! note "API Behavior"
    Based on testing, this endpoint currently returns an empty list. The endpoint exists but appears to only be populated in certain circumstances.

!!! example "Example Response"
    ```json
    {
      "count": 0,
      "pageIndex": 0,
      "pageSize": 25,
      "pageCount": 0,
      "items": []
    }
    ```

## Team Record Endpoint Clarification

| Endpoint | Description |
|----------|-------------|
| `/seasons/{year}/types/{type}/teams/{team_id}/record` | Team record details for a specific season type |

!!! important "Required Path Parameter"
    Note that the season type (`type`) is a required path parameter for this endpoint. Attempting to access `/seasons/{year}/teams/{team_id}/record` without specifying the season type will result in a 404 error.

### Path Parameters

| Parameter | Description | Example |
|-----------|-------------|---------|
| `year` | Season year | `2024` |
| `type` | Season type (1=Preseason, 2=Regular Season, 3=Postseason, 4=Off Season) | `2` |
| `team_id` | Unique identifier for the team | `52` (North Carolina) |

### Response Format

The response contains detailed record information categorized by different contexts, with each record type providing comprehensive statistics:

1. **overall** - The team's overall record for the season
2. **home** - The team's record in home games
3. **away** - The team's record in away games
4. **vs AP Top 25** - The team's record against AP Top 25 ranked teams
5. **vs USA Today Top 25** - The team's record against USA Today ranked teams
6. **vs Conference** - The team's record in conference play

Each record contains the following key statistics:
- Win/loss summary (e.g., "17-16")
- Win percentage value
- Detailed statistics including:
  - Overtime wins and losses
  - Average points for and against
  - Point differential
  - Games behind leader
  - Games played
  - Winning streaks
  - Conference rankings

!!! example "Example Response (Partial)"
    ```json
    {
      "count": 6,
      "pageIndex": 1,
      "pageSize": 25,
      "pageCount": 1,
      "items": [
        {
          "$ref": "http://sports.core.api.espn.com/v2/sports/basketball/leagues/mens-college-basketball/seasons/2024/types/2/teams/52/records/0?lang=en&region=us",
          "id": "0",
          "name": "overall",
          "abbreviation": "Season",
          "displayName": "Team Season Record",
          "shortDisplayName": "Season",
          "description": "Overall Record",
          "type": "total",
          "summary": "17-16",
          "displayValue": "17-16",
          "value": 0.5151515151515151,
          "stats": [
            {
              "name": "OTLosses",
              "displayName": "Overtime Losses",
              "shortDisplayName": "OTL",
              "description": "Number of Overtime Losses",
              "abbreviation": "OTL",
              "type": "otlosses",
              "value": 0.0,
              "displayValue": "0"
            },
            // Additional statistics omitted for brevity
          ]
        },
        // Additional record types omitted for brevity
      ]
    }
    ```

## Individual Record Type Endpoint

| Endpoint | Description |
|----------|-------------|
| `/seasons/{year}/types/{type}/teams/{team_id}/records/{record_id}` | Detailed information about a specific record type |

### Path Parameters

| Parameter | Description | Example |
|-----------|-------------|---------|
| `year` | Season year | `2024` |
| `type` | Season type (1=Preseason, 2=Regular Season, 3=Postseason, 4=Off Season) | `2` |
| `team_id` | Unique identifier for the team | `52` (North Carolina) |
| `record_id` | Type of record to retrieve | `0` (Overall), `1` (Conference), etc. |

### Record ID Values

| Record ID | Description |
|-----------|-------------|
| `0` | Overall record |
| `1` | Conference record |
| `2` | Home record |
| `3` | Away record |
| `4` | vs AP Top 25 record |
| `5` | vs USA Today Top 25 record |

### Response Format

The response contains comprehensive statistics for the specified record type:

- Basic information (name, summary, description)
- Winning percentage 
- Detailed statistics array with:
  - Overtime wins and losses
  - Points for and against (total and average)
  - Point differential
  - Games behind leader
  - Games played
  - Win/loss streak information
  - Conference standings information

!!! example "Example Response (Overall Record)"
    ```json
    {
      "$ref": "http://sports.core.api.espn.com/v2/sports/basketball/leagues/mens-college-basketball/seasons/2024/types/2/teams/52/records/0?lang=en&region=us",
      "id": "0",
      "name": "overall",
      "abbreviation": "Season",
      "displayName": "Team Season Record",
      "shortDisplayName": "Season",
      "description": "Overall Record",
      "type": "total",
      "summary": "17-16",
      "displayValue": "17-16",
      "value": 0.5151515151515151,
      "stats": [
        {
          "name": "OTLosses",
          "displayName": "Overtime Losses",
          "shortDisplayName": "OTL",
          "description": "Number of Overtime Losses",
          "abbreviation": "OTL",
          "type": "otlosses",
          "value": 0.0,
          "displayValue": "0"
        },
        {
          "name": "OTWins",
          "displayName": "Overtime Wins",
          "shortDisplayName": "OT Wins",
          "description": "Number of Overtime Wins",
          "abbreviation": "OTW",
          "type": "otwins",
          "value": 1.0,
          "displayValue": "1"
        },
        {
          "name": "avgPointsAgainst",
          "displayName": "Opponent Points Per Game",
          "shortDisplayName": "OPP PPG",
          "description": "Opponent Points Per Game",
          "abbreviation": "OPP PPG",
          "type": "avgpointsagainst",
          "value": 76.30303,
          "displayValue": "76.3"
        },
        // Additional statistics omitted for brevity
      ]
    }
    ```

!!! tip "Accessing Individual Records"
    When using the team record list endpoint (`/seasons/{year}/types/{type}/teams/{team_id}/record`), you receive an array of record references. Each reference contains an `id` that can be used to access the detailed individual record via this endpoint.

## Common Data Structures

### Team Object

The basic team object contains the following key fields:

| Field | Description |
|-------|-------------|
| `id` | Unique identifier for the team |
| `uid` | Unique identifier with format `s:40~l:41~t:{team_id}` |
| `slug` | URL-friendly identifier for the team |
| `location` | Geographic location of the team |
| `name` | Team name (without location) |
| `nickname` | Shortened/alternate name |
| `abbreviation` | 2-4 letter abbreviation for the team |
| `displayName` | Full display name (location + name) |
| `shortDisplayName` | Shortened display name |
| `color` | Primary team color (hex code without #) |
| `alternateColor` | Secondary team color (hex code without #) |
| `isActive` | Boolean indicating if team is active |
| `logos` | Array of logo images in different formats |
| `venue` | Information about the team's home venue |
| `links` | Array of related links (roster, stats, etc.) |

### Pagination Structure

All list endpoints (athletes, events, coaches) return paginated results with the following structure:

| Field | Description |
|-------|-------------|
| `count` | Total number of items available |
| `pageIndex` | Current page number (1-based) |
| `pageSize` | Number of items per page |
| `pageCount` | Total number of pages |
| `items` | Array of items for the current page |

### Reference Structure

Many objects contain a `$ref` field pointing to more detailed data:

```json
{
  "$ref": "http://sports.core.api.espn.com/v2/sports/basketball/leagues/mens-college-basketball/seasons/2024/teams/52?lang=en&region=us"
}
```

## Notes on Usage

!!! tip "Requesting Complete Data"
    For team statistics and records, you must specify the season type (e.g., `/seasons/2024/types/2/teams/52/statistics`) to get complete data. Using the endpoint without specifying a season type may return incomplete data or an error.

!!! warning "Active Flag"
    Not all teams are active in each season. Use the `active=true` query parameter when fetching a list of teams to get only currently active teams.

!!! note "Record Types"
    The team record endpoint returns multiple record types (overall, home, away, etc.). Each record type provides a different perspective on the team's performance.
