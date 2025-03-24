# Authentication

This guide explains authentication options for the ESPN API for NCAA Men's Basketball data.

## Public Access

Most NCAA Men's Basketball endpoints in the ESPN API are publicly accessible without authentication. You can make requests to these endpoints directly without any authentication token or API key.

Example of a public endpoint request:

```bash
curl -s "https://sports.core.api.espn.com/v2/sports/basketball/leagues/mens-college-basketball/seasons/2024/teams" | jq
```

## Access Limitations

While most data is publicly available, be aware of the following limitations:

1. **Rate limiting** applies to all requests, including unauthenticated ones
2. **Some endpoints or data fields** may require authentication or special permissions
3. **Commercial applications** may require additional licensing agreements with ESPN

## Authentication Methods

If you do need to authenticate for certain endpoints or use cases, the ESPN API supports the following authentication methods:

### API Key Authentication

For some ESPN APIs and products, API key authentication is used. If you have been provided with an API key:

1. Include the key in the request headers:

```bash
curl -s -H "X-API-Key: YOUR_API_KEY" "https://sports.core.api.espn.com/v2/sports/basketball/leagues/mens-college-basketball/seasons/2024/teams" | jq
```

2. Or append it as a query parameter:

```bash
curl -s "https://sports.core.api.espn.com/v2/sports/basketball/leagues/mens-college-basketball/seasons/2024/teams?apikey=YOUR_API_KEY" | jq
```

!!! note
    The specific parameter name may vary based on the API product. Check your API key documentation for the exact parameter name.

### OAuth Authentication

For applications requiring user-specific data or performing actions on behalf of users, OAuth 2.0 may be used:

1. Register your application with ESPN Developer Portal
2. Implement the OAuth 2.0 authorization flow to obtain access tokens
3. Include the access token in request headers:

```bash
curl -s -H "Authorization: Bearer YOUR_ACCESS_TOKEN" "https://sports.core.api.espn.com/v2/sports/basketball/leagues/mens-college-basketball/user-specific-endpoint" | jq
```

## Security Best Practices

When working with authentication:

1. **Never expose API keys or tokens** in client-side code or public repositories
2. **Store credentials securely** using environment variables or a secure key management service
3. **Implement token refresh** mechanisms for OAuth tokens
4. **Use HTTPS** for all API requests
5. **Implement proper error handling** for authentication failures

Example of secure credential handling:

```javascript
// Node.js example using environment variables
require('dotenv').config(); // Load environment variables from .env file

const apiKey = process.env.ESPN_API_KEY;
if (!apiKey) {
  throw new Error('API key not found in environment variables');
}

async function fetchTeams() {
  const response = await fetch(
    'https://sports.core.api.espn.com/v2/sports/basketball/leagues/mens-college-basketball/seasons/2024/teams',
    {
      headers: {
        'X-API-Key': apiKey
      }
    }
  );
  
  if (!response.ok) {
    if (response.status === 401) {
      throw new Error('Authentication failed: Invalid API key');
    } else if (response.status === 403) {
      throw new Error('Authentication failed: Insufficient permissions');
    }
    throw new Error(`API request failed with status ${response.status}`);
  }
  
  return await response.json();
}
```

## Authentication Errors

Common authentication-related error codes:

- **401 Unauthorized**: Authentication failed (invalid or missing credentials)
- **403 Forbidden**: Authentication succeeded but access is denied due to insufficient permissions

Example error response for authentication failures:

```json
{
  "error": {
    "status": 401,
    "message": "Authentication required",
    "detail": "This endpoint requires valid authentication"
  }
}
```

## Commercial Usage

If you're developing a commercial application that uses ESPN API data:

1. **Review the ESPN Developer Terms of Service**
2. **Contact ESPN licensing** for commercial usage agreements
3. **Obtain proper authentication credentials** for your approved use case

## Next Steps

- If you're building an application that needs to handle authentication, see the [Best Practices](best-practices.md) guide for implementation recommendations
- For information on managing request volume with authenticated requests, see the [Rate Limiting](rate-limiting.md) guide

## Support

If you encounter authentication issues or have questions about access requirements for specific endpoints:

1. Check if the endpoint requires authentication in its documentation
2. Review your authentication implementation for common issues
3. Contact ESPN Developer Support for assistance with authentication problems

Remember that most NCAA Men's Basketball data endpoints are accessible without authentication, so you can build many applications without worrying about authentication at all. 