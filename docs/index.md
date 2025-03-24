# ESPN API Documentation - NCAA Men's Basketball

!!! info "API Base URL"
    The base URL for all endpoints is:
    ```
    https://sports.core.api.espn.com/v2/sports/basketball/leagues/mens-college-basketball
    ```

## Documentation Status

✅ Complete - Well documented with examples and field descriptions  
🔄 Partial - Basic documentation exists but needs more detail or examples  
⚠️ Minimal - Only basic information available, needs significant work  
❌ Not Started - Listed but not documented yet  

## TODO List

The following areas need additional documentation:

1. **Standings Endpoints** - Document the `/seasons/{year}/types/{type}/groups/{group_id}/standings` endpoint for retrieving conference standings data
2. **Capitalization Inconsistency** - Note the inconsistency between `powerindex` and `powerIndex` in some references
3. **Specific Futures Details** - Add documentation for `/seasons/{year}/futures/{futures_id}` endpoint
4. **Child Groups Navigation** - Better explain how to navigate the hierarchy between divisions and conferences
5. **Standings Format** - Explain the data format for standings (division standings, conference standings, etc.)
6. **Historical Data Limitations** - Provide more specific information about which endpoints have reliable historical data

## Documentation Structure

This API documentation is organized into the following files:

| Document | Contents |
|----------|----------|
| [Home](index.md) | Overview and index (this file) |
| [Season Endpoints](season_endpoints.md) | Season, week, and calendar endpoints with examples |
| [Team Endpoints](team_endpoints.md) | Team data, rosters, and statistics endpoints |
| [Athlete Endpoints](athlete_endpoints.md) | Player information and statistics endpoints |
| [Event Endpoints](event_endpoints.md) | Game data, play-by-play, and competition endpoints |
| [Data Structures](data_structures.md) | Common data structures and object definitions |

## Base Endpoints

| Endpoint | Status | Description |
|----------|--------|-------------|
| `/` | 🔄 | Top-level league information |
| `/seasons` | ✅ | List of available seasons |
| `/events` | 🔄 | List of events (games) |
| `/rankings` | 🔄 | Rankings information |
| `/calendar` | ❌ | Calendar information |
| `/franchises` | ❌ | List of franchises |
| `/notes` | ❌ | Notes related to the league |
| `/transactions` | ❌ | List of transactions |

## Season-specific Endpoints

!!! note "Dynamic Parameter"
    Replace `{year}` with the desired season year (e.g., 2025).

| Endpoint | Status | Description |
|----------|--------|-------------|
| `/seasons/{year}` | ✅ | Information about a specific season |
| `/seasons/{year}/athletes` | ✅ | List of athletes in the season |
| `/seasons/{year}/teams` | ✅ | List of teams in the season |
| `/seasons/{year}/powerindex` | ✅ | Power index rankings for the season |
| `/seasons/{year}/futures` | ✅ | Futures/predictions for the season |
| `/seasons/{year}/awards` | ✅ | Awards for the season |
| `/seasons/{year}/rankings` | ✅ | Season rankings |
| `/seasons/{year}/powerIndex/leaders` | ✅ | Power index leaders for the season |

### Season Type Endpoints

!!! info "Season Types"
    Season types are categorized as:
    
    - Type 1: Preseason
    - Type 2: Regular Season
    - Type 3: Postseason
    - Type 4: Off Season

| Endpoint | Status | Description |
|----------|--------|-------------|
| `/seasons/{year}/types/{type}` | ✅ | Information about a specific season type |
| `/seasons/{year}/types` | ✅ | List of all season types for a given season |
| `/seasons/{year}/types/{type}/groups` | ✅ | Groups (conferences) in the season type |
| `/seasons/{year}/types/{type}/weeks` | ✅ | Weeks in the season type |
| `/seasons/{year}/types/{type}/leaders` | ✅ | Statistical leaders for the season type |

#### Week-specific Endpoints

| Endpoint | Status | Description |
|----------|--------|-------------|
| `/seasons/{year}/types/{type}/weeks/{week}` | ✅ | Information about a specific week |
| `/seasons/{year}/types/{type}/weeks/{week}/events` | ✅ | Events in a specific week |
| `/seasons/{year}/types/{type}/weeks/{week}/rankings` | ✅ | Rankings for a specific week |
| `/seasons/{year}/types/{type}/weeks/{week}/rankings/{ranking_id}` | ✅ | Detailed ranking data for a specific ranking system in a specific week |

## Team Endpoints

!!! note "Dynamic Parameters"
    Replace `{year}` with the season year and `{team_id}` with the team ID.

| Endpoint | Status | Description |
|----------|--------|-------------|
| `/seasons/{year}/teams/{team_id}` | ✅ | Information about a specific team |
| `/seasons/{year}/teams/{team_id}/athletes` | ✅ | Athletes on the team - Returns array of athlete references with pagination |
| `/seasons/{year}/teams/{team_id}/events` | ✅ | Team's schedule/results - Returns array of event references with pagination |
| `/seasons/{year}/teams/{team_id}/coaches` | ✅ | Team coaches - Returns array of coaches references with pagination |
| `/seasons/{year}/teams/{team_id}/statistics` | ✅ | Team statistics |
| `/seasons/{year}/types/{type}/teams/{team_id}/statistics` | ✅ | Team statistics for a specific season type |
| `/seasons/{year}/types/{type}/teams/{team_id}/record` | ✅ | Team record details for a season type |
| `/seasons/{year}/types/{type}/teams/{team_id}/records/{record_id}` | ✅ | Detailed information about a specific record type |
| `/seasons/{year}/teams/{team_id}/ranks` | ✅ | Team ranking information |
| `/seasons/{year}/types/{type}/teams/{team_id}/ats` | ✅ | Against the spread records for a specific team |
| `/seasons/{year}/teams/{team_id}/awards` | ✅ | Awards received by a team in a specific season |
| `/teams/{team_id}/notes` | ✅ | Notes related to a specific team |

## Group and Conference Endpoints

| Endpoint | Status | Description |
|----------|--------|-------------|
| `/seasons/{year}/types/{type}/groups/{group_id}` | ✅ | Information about a specific conference or division |
| `/seasons/{year}/types/{type}/groups/{group_id}/teams` | ✅ | Teams in a specific conference or division |
| `/seasons/{year}/types/{type}/groups/{group_id}/standings` | ⚠️ | Standings for a specific conference or division |

## Event (Game) Endpoints

!!! note "Dynamic Parameter"
    Replace `{event_id}` with the specific event ID.

| Endpoint | Status | Description |
|----------|--------|-------------|
| `/events/{event_id}` | 🔄 | Information about a specific event |
| `/events/{event_id}/competitions/{event_id}` | 🔄 | Competition details |
| `/events/{event_id}/competitions/{event_id}/odds` | ⚠️ | Odds for the event - Returns array of odds data grouped by odds provider |
| `/events/{event_id}/competitions/{event_id}/plays` | 🔄 | Play-by-play data - Returns array of plays in sequence order with detailed information |
| `/events/{event_id}/competitions/{event_id}/probabilities` | ⚠️ | Win probability data - Returns array showing win probability changes throughout the game |
| `/events/{event_id}/competitions/{event_id}/competitors/{competitor_id}/roster` | 🔄 | Team roster for the event |
| `/events/{event_id}/competitions/{event_id}/competitors/{competitor_id}/records` | ⚠️ | Team records for the event |
| `/events/{event_id}/competitions/{event_id}/officials` | ❌ | Officials for the event |

## Rankings Endpoints

!!! note "Dynamic Parameters"
    Replace `{year}` with the season year and `{ranking_id}` with the ranking system ID (e.g., 1 for AP Top 25, 2 for Coaches Poll).

| Endpoint | Status | Description |
|----------|--------|-------------|
| `/seasons/{year}/rankings` | ✅ | List of available rankings systems for the season |
| `/seasons/{year}/rankings/{ranking_id}` | ⚠️ | Detailed information about a specific ranking system |
| `/seasons/{year}/types/{type}/weeks/{week}/rankings` | ✅ | Rankings for a specific week |
| `/seasons/{year}/types/{type}/weeks/{week}/rankings/{ranking_id}` | ✅ | Detailed ranking data for a specific ranking system in a specific week |

## Awards Endpoints

!!! note "Dynamic Parameters"
    Replace `{year}` with the season year and `{award_id}` with the specific award ID.

| Endpoint | Status | Description |
|----------|--------|-------------|
| `/seasons/{year}/awards` | ✅ | List of awards given for the season |
| `/seasons/{year}/awards/{award_id}` | ⚠️ | Detailed information about a specific award, including winners |

## Athlete Endpoints

!!! note "Dynamic Parameter"
    Replace `{athlete_id}` with the specific athlete ID.

| Endpoint | Status | Description |
|----------|--------|-------------|
| `/athletes/{athlete_id}` | 🔄 | Information about a specific athlete |
| `/seasons/{year}/athletes/{athlete_id}` | ⚠️ | Season-specific information about an athlete |
| `/athletes/{athlete_id}/statistics/0` | 🔄 | Athlete statistics - Returns comprehensive statistics including defensive, general and offensive categories |
| `/athletes/{athlete_id}/statisticslog` | ⚠️ | Athlete statistics log - Returns array of season entries with links to total and team-specific statistics |
| `/seasons/{year}/types/{type}/athletes/{athlete_id}/statistics` | 🔄 | Athlete statistics for a specific season type |
| `/seasons/{year}/types/{type}/athletes/{athlete_id}/projections` | ❌ | Athlete projections for a specific season type |
| `/seasons/{year}/athletes/{athlete_id}/eventlog` | ⚠️ | Athlete event log for a season |
| `/seasons/{year}/athletes/{athlete_id}/notes` | ❌ | Notes about an athlete for a season |

## Futures Endpoints

| Endpoint | Status | Description |
|----------|--------|-------------|
| `/seasons/{year}/futures` | ✅ | List of futures/betting markets for the season |
| `/seasons/{year}/futures/{futures_id}` | ⚠️ | Detailed information for a specific futures market |

## Venue Endpoints

!!! note "Dynamic Parameters"
    Replace `{venue_id}` with the specific venue ID.

| Endpoint | Status | Description |
|----------|--------|-------------|
| `/venues/{venue_id}` | ⚠️ | Information about a specific venue |

## Franchise Endpoints

!!! note "Dynamic Parameters"
    Replace `{franchise_id}` with the specific franchise ID.

| Endpoint | Status | Description |
|----------|--------|-------------|
| `/franchises/{franchise_id}` | ⚠️ | Information about a specific franchise |

## College Endpoints

!!! note "Dynamic Parameters"
    Replace `{college_id}` with the specific college ID.

| Endpoint | Status | Description |
|----------|--------|-------------|
| `/colleges/{college_id}` | ⚠️ | General information about a specific college |

## Common Query Parameters

The following query parameters can be used with many endpoints:

| Parameter | Description | Example |
|-----------|-------------|---------|
| `limit` | Number of items to return | `?limit=100` |
| `active` | Filter for active items | `?active=true` |
| `lang` | Language for the response | `?lang=en` |
| `region` | Region for the response | `?region=us` |
| `season` | Specific season for the data | `?season=2025` |

## Common Data Structures

### Pagination

!!! example "Pagination Response Example"
    Most list endpoints return paginated results with the following structure:
    
    ```json
    {
      "count": 200,        // Total number of items
      "pageIndex": 1,      // Current page (1-based)
      "pageSize": 25,      // Number of items per page
      "pageCount": 8,      // Total number of pages
      "items": [...]       // Array of items for the current page
    }
    ```

### Reference Structure

!!! example "Reference Structure Example"
    Many objects contain a `$ref` field pointing to more detailed data:
    
    ```json
    {
      "$ref": "http://sports.core.api.espn.com/v2/sports/basketball/leagues/mens-college-basketball/seasons/2025?lang=en&region=us"
    }
    ```

### Team Object

!!! example "Team Object Example"
    Example of basic team information:
    
    ```json
    {
      "id": "52",
      "uid": "s:40~l:41~t:52",
      "slug": "north-carolina-tar-heels",
      "location": "North Carolina",
      "name": "Tar Heels",
      "nickname": "UNC",
      "abbreviation": "UNC",
      "color": "7BAFD4",
      "alternateColor": "13294B",
      "logos": [
        {
          "href": "https://a.espncdn.com/i/teamlogos/ncaa/500/153.png",
          "width": 500,
          "height": 500,
          "alt": "North Carolina Tar Heels",
          "rel": ["full", "default"]
        }
      ]
    }
    ```

### Event Object

!!! example "Event Object Example"
    Example of basic event information:
    
    ```json
    {
      "id": "401478604",
      "uid": "s:40~l:41~e:401478604",
      "date": "2024-03-09T23:00Z",
      "name": "Duke at North Carolina",
      "shortName": "DUKE @ UNC",
      "season": {
        "year": 2025,
        "type": 2,
        "slug": "regular-season"
      },
      "competitions": [
        {
          "id": "401478604",
          "competitors": [
            {
              "id": "52",
              "homeAway": "home"
            },
            {
              "id": "150",
              "homeAway": "away"
            }
          ]
        }
      ]
    }
    ```

## Notes on Usage

!!! warning "Important Notes"
    1. Many ESPN API responses include a `$ref` field that contains a URL to more detailed data.
    2. The IDs for teams, athletes, and events can be extracted from the `$ref` URL or the `id` field in the response.
    3. The API generally returns paginated results with `count`, `pageIndex`, `pageSize`, and `pageCount` fields.
    4. For large datasets (e.g., all athletes), use the `limit` parameter to control the number of results returned.
    5. Some endpoints might not return complete data at certain times of the year (e.g., postseason data during the regular season).
    6. When extracting IDs from `$ref` URLs, be aware that they may include query parameters (`?lang=en&region=us`). These should be removed to get the clean ID.

## Authentication

!!! tip "Authentication"
    The ESPN v2 API endpoints documented here do not require authentication for basic usage. However, implementing rate limiting or user-agent identification in your client is recommended to avoid potential IP blocking.

## Example Requests

=== "Python"

    ```python
    import requests
    
    # Get information about the current NCAA Men's Basketball season
    url = "https://sports.core.api.espn.com/v2/sports/basketball/leagues/mens-college-basketball/seasons/2025"
    headers = {"User-Agent": "ESPN API Documentation Project"}
    response = requests.get(url, headers=headers)
    data = response.json()
    ```

=== "JavaScript"

    ```javascript
    // Get information about the current NCAA Men's Basketball season
    fetch("https://sports.core.api.espn.com/v2/sports/basketball/leagues/mens-college-basketball/seasons/2025", {
      headers: {
        "User-Agent": "ESPN API Documentation Project"
      }
    })
    .then(response => response.json())
    .then(data => console.log(data));
    ```

=== "Curl"

    ```bash
    # Get information about the current NCAA Men's Basketball season
    curl -X GET "https://sports.core.api.espn.com/v2/sports/basketball/leagues/mens-college-basketball/seasons/2025" \
      -H "User-Agent: ESPN API Documentation Project"
    ```
