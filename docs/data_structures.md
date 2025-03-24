# ESPN API - Common Data Structures

This document provides detailed information about common data structures and patterns used throughout the ESPN v2 API for NCAA Men's Basketball.

## Basic Response Pattern

!!! abstract "Standard Pattern"
    Most ESPN API responses follow a consistent pattern with these common elements:

    1. Reference Objects with `$ref` fields
    2. Pagination structure for list endpoints
    3. Standardized field naming conventions
    4. Nested objects with descriptive properties

## Reference Objects

### Ref Structure

!!! example "Reference Object"
    Many responses contain reference objects that point to more detailed information. These references use the `$ref` field with a URL to the detailed resource.
    
    ```json
    {
      "$ref": "https://sports.core.api.espn.com/v2/sports/basketball/leagues/mens-college-basketball/seasons/2025"
    }
    ```
    
    Typically, the client would need to make additional HTTP requests to resolve these references. The API is designed to be deeply navigable through these references.

## Pagination Structure

!!! example "Pagination Structure"
    List endpoints return paginated results with a consistent structure. The pagination information includes count, page size, current page index, and total number of pages.
    
    ```json
    {
      "count": 363,         // Total number of items in the full dataset
      "pageIndex": 1,       // Current page (1-based indexing)
      "pageSize": 25,       // Number of items per page
      "pageCount": 15,      // Total number of pages
      "items": [...]        // Array of items for the current page
    }
    ```

!!! tip "Navigation"
    To navigate through pages, use the `page` query parameter, for example: `?page=2`
    
    To change the number of items per page, use the `limit` query parameter, for example: `?limit=50`

## Response Status Codes

The API returns standard HTTP status codes:

| Code | Description |
|------|-------------|
| 200 | Successful request |
| 400 | Bad request - typically malformed parameters |
| 401 | Unauthorized - authentication required |
| 403 | Forbidden - insufficient permissions |
| 404 | Not found - resource doesn't exist |
| 429 | Too many requests - rate limit exceeded |
| 500 | Internal server error |