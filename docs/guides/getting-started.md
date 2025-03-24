# Getting Started with the ESPN API

This guide will help you get started with using the ESPN API for NCAA Men's Basketball data. It covers the basic concepts, tools, and steps needed to begin making requests and processing responses.

## Overview

The ESPN API provides programmatic access to NCAA Men's Basketball data, including teams, players, games, statistics, and more. The API follows REST principles, returning JSON-formatted responses that can be consumed by any programming language or tool that supports HTTP requests.

## Prerequisites

Before you begin using the API, you should have:

- Basic understanding of REST APIs and HTTP requests
- Familiarity with JSON data format
- A tool for making HTTP requests (curl, Postman, or your preferred programming language)

## Making Your First Request

Let's start by making a simple request to get information about the current NCAA Men's Basketball season.

### Using curl

```bash
curl -s "https://sports.core.api.espn.com/v2/sports/basketball/leagues/mens-college-basketball/seasons/2024" | jq
```

### Using JavaScript

```javascript
async function getCurrentSeason() {
  const response = await fetch(
    "https://sports.core.api.espn.com/v2/sports/basketball/leagues/mens-college-basketball/seasons/2024"
  );
  const data = await response.json();
  console.log(data);
  return data;
}

getCurrentSeason();
```

### Using Python

```python
import requests

def get_current_season():
    url = "https://sports.core.api.espn.com/v2/sports/basketball/leagues/mens-college-basketball/seasons/2024"
    response = requests.get(url)
    return response.json()

data = get_current_season()
print(data)
```

## Understanding the Response

The API returns data in JSON format. Here's an example response from the seasons endpoint:

```json
{
  "$ref": "http://sports.core.api.espn.com/v2/sports/basketball/leagues/mens-college-basketball/seasons/2024?lang=en&region=us",
  "id": "2024",
  "name": "2023-24",
  "type": 2,
  "year": 2024,
  "startDate": "2023-11-06T08:00Z",
  "endDate": "2024-04-08T06:59Z",
  "types": {
    "$ref": "http://sports.core.api.espn.com/v2/sports/basketball/leagues/mens-college-basketball/seasons/2024/types?lang=en&region=us"
  }
}
```

Each response provides:

- Unique identifiers for the resource
- Basic information about the resource
- References to related resources (using the `$ref` field)
- Embedded related resources when requested

## Working with References

The ESPN API uses references to link related resources. References appear as URLs in the `$ref` field or as embedded objects. You can follow these references to retrieve additional data.

For example, to get the season types for the 2024 season:

```bash
curl -s "https://sports.core.api.espn.com/v2/sports/basketball/leagues/mens-college-basketball/seasons/2024/types" | jq
```

## Pagination

Endpoints that return collections of data use pagination to limit response size. Paginated responses include:

- `count`: Total number of items
- `pageIndex`: Current page number (starting from 1)
- `pageSize`: Number of items per page
- `pageCount`: Total number of pages
- `items`: Array of data items for the current page

Example request with pagination:

```bash
curl -s "https://sports.core.api.espn.com/v2/sports/basketball/leagues/mens-college-basketball/seasons/2024/teams?limit=10&offset=0" | jq
```

To navigate through pages, adjust the `limit` and `offset` parameters:
- `limit`: Number of items to return per page
- `offset`: Number of items to skip

## Common Query Parameters

Most endpoints support these common parameters:

- `lang`: Language code (e.g., `en` for English)
- `region`: Region code (e.g., `us` for United States)
- `limit`: Maximum number of items to return in paginated responses
- `offset`: Number of items to skip in paginated responses

## Working with Dates

When working with date-based endpoints, you can filter results using the `dates` parameter:

```bash
# Get events for a specific date (January 10, 2024)
curl -s "https://sports.core.api.espn.com/v2/sports/basketball/leagues/mens-college-basketball/seasons/2024/events?dates=20240110" | jq

# Get events for a date range (January 10-17, 2024)
curl -s "https://sports.core.api.espn.com/v2/sports/basketball/leagues/mens-college-basketball/seasons/2024/events?dates=20240110-20240117" | jq
```

## Error Handling

The API returns standard HTTP status codes to indicate success or failure:

- `200 OK`: Request succeeded
- `400 Bad Request`: Invalid request parameters
- `404 Not Found`: Resource not found
- `429 Too Many Requests`: Rate limit exceeded
- `500 Internal Server Error`: Server-side error

Error responses include a JSON object with details about the error.

## Best Practices

1. **Cache responses** when appropriate to reduce API calls
2. **Use specific endpoints** rather than retrieving large datasets and filtering client-side
3. **Implement proper error handling** to gracefully handle API errors
4. **Respect rate limits** by implementing backoff strategies
5. **Keep query parameters minimal** to improve response times

## Next Steps

Now that you understand the basics, explore specific endpoints for:

- [Teams](../endpoints/teams.md) - Get team information and statistics
- [Athletes](../endpoints/athletes.md) - Access player data and performance
- [Events](../endpoints/events.md) - Retrieve game details and results
- [Seasons](../endpoints/seasons.md) - Work with season-specific data

For more advanced usage, check out these guides:
- [Authentication](authentication.md)
- [Rate Limiting](rate-limiting.md)
- [Best Practices](best-practices.md)

## Troubleshooting

If you encounter issues:

1. Verify your request URL and parameters
2. Check if you're hitting rate limits
3. Ensure you're handling responses correctly
4. Review the [Status Codes](../reference/status-codes.md) documentation 