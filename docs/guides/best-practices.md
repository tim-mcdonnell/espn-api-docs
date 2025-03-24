# Best Practices for the ESPN API

This guide outlines best practices for working with the ESPN API for NCAA Men's Basketball data. Following these recommendations will help you build more efficient, reliable, and maintainable applications.

## Efficient API Usage

### 1. Minimize Request Volume

- **Batch related requests** when possible instead of making many small individual requests
- **Cache responses** based on their expected freshness - some data (like team lists) changes infrequently
- **Implement conditional requests** using HTTP headers like `If-Modified-Since` or `If-None-Match` when supported

### 2. Optimize Query Parameters

- **Use specific filters** rather than retrieving large datasets and filtering client-side
- **Limit response size** by requesting only the data you need
- **Use pagination effectively** by adjusting the `limit` parameter based on your needs

### 3. Handle Rate Limiting Gracefully

- **Track rate limit headers** in responses to monitor your usage
- **Implement backoff strategies** when you approach or exceed rate limits
- **Distribute requests evenly** over time rather than sending them in bursts
- **Prioritize critical requests** when working within rate limits

## Resilient Integration

### 1. Implement Robust Error Handling

- **Check HTTP status codes** before processing responses
- **Handle 429 (Too Many Requests)** errors by backing off and retrying
- **Parse error responses** to extract meaningful information
- **Log API errors** with sufficient context for troubleshooting

```javascript
async function fetchWithRetry(url, options = {}, maxRetries = 3) {
  let retries = 0;
  while (retries < maxRetries) {
    try {
      const response = await fetch(url, options);
      
      // Handle rate limiting
      if (response.status === 429) {
        const retryAfter = response.headers.get('Retry-After') || 5;
        await new Promise(resolve => setTimeout(resolve, retryAfter * 1000));
        retries++;
        continue;
      }
      
      // Handle other errors
      if (!response.ok) {
        const errorData = await response.json().catch(() => ({}));
        throw new Error(`API error: ${response.status} ${JSON.stringify(errorData)}`);
      }
      
      return await response.json();
    } catch (error) {
      if (retries >= maxRetries - 1) throw error;
      retries++;
      // Exponential backoff
      await new Promise(resolve => setTimeout(resolve, 1000 * Math.pow(2, retries)));
    }
  }
}
```

### 2. Design for API Changes

- **Validate response data** rather than assuming structure
- **Build abstractions** around direct API calls to isolate your application from API changes
- **Version your dependencies** on the API in documentation and code
- **Have fallback strategies** for when specific endpoints or features are unavailable

### 3. Monitor API Health

- **Track response times** to detect performance issues
- **Set up alerts** for unexpected error rates or outages
- **Implement circuit breakers** to prevent cascading failures
- **Maintain an audit log** of critical API interactions

## Performance Optimization

### 1. Efficient Data Processing

- **Process data incrementally** when working with large datasets through pagination
- **Use streaming parsers** for large JSON responses when appropriate
- **Normalize and store related data** to minimize redundant requests

### 2. Optimize Network Usage

- **Enable compression** in requests (Accept-Encoding: gzip)
- **Use HTTP/2** when available to reduce connection overhead
- **Implement connection pooling** for multiple requests

### 3. Caching Strategies

```python
import requests
import json
import time
import os

class ESPNAPIClient:
    def __init__(self, cache_dir=".cache", cache_ttl=3600):
        self.base_url = "https://sports.core.api.espn.com/v2/sports/basketball/leagues/mens-college-basketball"
        self.cache_dir = cache_dir
        self.cache_ttl = cache_ttl
        os.makedirs(cache_dir, exist_ok=True)
    
    def _get_cache_path(self, endpoint):
        import hashlib
        cache_key = hashlib.md5(endpoint.encode()).hexdigest()
        return os.path.join(self.cache_dir, f"{cache_key}.json")
    
    def _get_cached_response(self, endpoint):
        cache_path = self._get_cache_path(endpoint)
        if not os.path.exists(cache_path):
            return None
        
        # Check if cache is expired
        file_modified = os.path.getmtime(cache_path)
        if time.time() - file_modified > self.cache_ttl:
            return None
            
        with open(cache_path, 'r') as f:
            return json.load(f)
    
    def _cache_response(self, endpoint, data):
        cache_path = self._get_cache_path(endpoint)
        with open(cache_path, 'w') as f:
            json.dump(data, f)
    
    def get(self, endpoint, params=None, force_refresh=False):
        # Check cache first unless forced to refresh
        if not force_refresh:
            cached_data = self._get_cached_response(endpoint)
            if cached_data:
                return cached_data
        
        # Make the API request
        url = f"{self.base_url}{endpoint}"
        response = requests.get(url, params=params)
        response.raise_for_status()
        data = response.json()
        
        # Cache the response
        self._cache_response(endpoint, data)
        return data
```

Consider different caching strategies for different types of data:

- **Short-lived cache (minutes)**: Current game scores, live statistics
- **Medium-lived cache (hours)**: Daily schedules, standings
- **Long-lived cache (days)**: Team information, historical data
- **Versioned cache**: Cache data until a new version is detected

## Security Considerations

### 1. Protect Credentials

- **Never expose API keys** in client-side code
- **Use environment variables** for storing sensitive configuration
- **Implement proper authentication** when required

### 2. Validate Input and Output

- **Sanitize user input** before using it in API requests
- **Validate API responses** before processing to prevent injection attacks
- **Implement content security policies** for web applications

## User Experience Best Practices

### 1. Progressive Loading

- **Show loading indicators** for data that's being fetched
- **Implement progressive enhancement** to display partial data while loading more
- **Prioritize above-the-fold content** in your loading sequence

### 2. Graceful Degradation

- **Provide fallback content** when API data is unavailable
- **Cache critical data locally** to support offline or degraded experiences
- **Communicate API issues** to users in a helpful way

### 3. Performance Perception

- **Preload predictable data** before users need it
- **Update UI optimistically** before API responses when appropriate
- **Batch UI updates** to prevent excessive re-rendering

## Implementation Examples

### Efficient Team Data Retrieval

```javascript
// Less efficient approach - retrieving all teams then filtering
async function getTeamsByConference(year, conferenceId) {
  // Don't do this - it retrieves all teams
  const response = await fetch(`https://sports.core.api.espn.com/v2/sports/basketball/leagues/mens-college-basketball/seasons/${year}/teams?limit=400`);
  const data = await response.json();
  
  // Client-side filtering is less efficient
  return data.items.filter(team => team.conferenceId === conferenceId);
}

// More efficient approach - using the API's filtering capabilities
async function getTeamsByConference(year, conferenceId) {
  // Use the groups endpoint to get teams by conference directly
  const response = await fetch(`https://sports.core.api.espn.com/v2/sports/basketball/leagues/mens-college-basketball/seasons/${year}/groups/${conferenceId}/teams`);
  return await response.json();
}
```

### Handling Pagination Efficiently

```javascript
// Retrieve all items across multiple pages
async function getAllItems(baseUrl, pageSize = 100) {
  let allItems = [];
  let pageIndex = 1;
  let hasMorePages = true;
  
  while (hasMorePages) {
    const url = `${baseUrl}?limit=${pageSize}&offset=${(pageIndex - 1) * pageSize}`;
    const response = await fetch(url);
    const data = await response.json();
    
    allItems = [...allItems, ...data.items];
    
    hasMorePages = pageIndex < data.pageCount;
    pageIndex++;
  }
  
  return allItems;
}
```

## Conclusion

Following these best practices will help you build applications that use the ESPN API efficiently and reliably. Remember that good API citizenship benefits everyone by ensuring the service remains responsive and available.

For further guidance, refer to:
- [Getting Started Guide](getting-started.md)
- [Rate Limiting Documentation](rate-limiting.md)
- [Authentication Guide](authentication.md) 