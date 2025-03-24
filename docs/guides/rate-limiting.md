# Rate Limiting

This guide explains how rate limiting works in the ESPN API for NCAA Men's Basketball and provides strategies for working effectively within these limits.

## Understanding Rate Limits

The ESPN API implements rate limiting to ensure fair usage and maintain service stability. When you exceed these limits, requests will be rejected until your quota refreshes.

### Rate Limit Structure

Rate limits for the API are based on:

1. **Requests per minute (RPM)**: The number of requests allowed per minute
2. **Requests per day (RPD)**: The total number of requests allowed per day
3. **Requests per endpoint**: Some specific endpoints may have their own separate limits

!!! note
    Exact rate limit values are subject to change. Current limits are not published, but you can monitor your usage through response headers.

### Rate Limit Headers

The API includes rate limit information in response headers:

- `X-RateLimit-Limit`: Maximum number of requests allowed in the current period
- `X-RateLimit-Remaining`: Number of requests remaining in the current period
- `X-RateLimit-Reset`: Time when the current limit window resets (Unix timestamp)

Example:
```
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 87
X-RateLimit-Reset: 1619049600
```

## Handling Rate Limiting

### Detecting Rate Limit Errors

When you exceed rate limits, the API will respond with:

- HTTP status code `429 Too Many Requests`
- Response body with error details
- `Retry-After` header indicating when to retry (in seconds)

Example rate limit error response:
```json
{
  "error": {
    "status": 429,
    "message": "Rate limit exceeded. Please wait before making additional requests.",
    "detail": "You have exceeded the rate limit of 100 requests per minute"
  }
}
```

### Best Practices for Avoiding Rate Limits

1. **Implement caching**
   - Cache responses locally to reduce repeated requests
   - Set appropriate cache expiration based on data volatility
   - Use conditional requests with `If-Modified-Since` or `If-None-Match` headers

2. **Use efficient request patterns**
   - Batch related requests when possible
   - Request only the data you need
   - Avoid polling endpoints at high frequencies

3. **Implement rate tracking**
   - Monitor rate limit headers in responses
   - Track your usage against limits
   - Slow down requests as you approach limits

4. **Space out requests**
   - Distribute requests evenly over time
   - Avoid sending bursts of concurrent requests
   - Implement request queuing with controlled throughput

## Implementation Strategies

### Basic Rate Limit Handling

```javascript
async function fetchWithRateLimitHandling(url) {
  try {
    const response = await fetch(url);
    
    // Track rate limit info from headers
    const rateLimit = {
      limit: parseInt(response.headers.get('X-RateLimit-Limit') || '0'),
      remaining: parseInt(response.headers.get('X-RateLimit-Remaining') || '0'),
      reset: parseInt(response.headers.get('X-RateLimit-Reset') || '0')
    };
    
    // Log rate limit status (optional)
    console.log(`Rate limit: ${rateLimit.remaining}/${rateLimit.limit}`);
    
    if (response.status === 429) {
      const retryAfter = parseInt(response.headers.get('Retry-After') || '60');
      console.log(`Rate limited. Retrying after ${retryAfter} seconds`);
      
      // Wait for the specified retry time
      await new Promise(resolve => setTimeout(resolve, retryAfter * 1000));
      
      // Retry the request
      return fetchWithRateLimitHandling(url);
    }
    
    if (!response.ok) {
      throw new Error(`API error: ${response.status}`);
    }
    
    return await response.json();
  } catch (error) {
    console.error('Request failed:', error);
    throw error;
  }
}
```

### Advanced Request Throttling

```javascript
class ThrottledApiClient {
  constructor(options = {}) {
    this.baseUrl = 'https://sports.core.api.espn.com/v2/sports/basketball/leagues/mens-college-basketball';
    this.requestsPerMinute = options.requestsPerMinute || 60;
    this.requestQueue = [];
    this.processing = false;
    
    // Track rate limits
    this.rateLimit = {
      limit: Infinity,
      remaining: Infinity,
      reset: 0
    };
  }
  
  async request(endpoint, params = {}) {
    return new Promise((resolve, reject) => {
      // Add to queue
      this.requestQueue.push({
        endpoint,
        params,
        resolve,
        reject
      });
      
      // Start processing queue if not already processing
      if (!this.processing) {
        this.processQueue();
      }
    });
  }
  
  async processQueue() {
    if (this.requestQueue.length === 0) {
      this.processing = false;
      return;
    }
    
    this.processing = true;
    
    // Calculate delay between requests to stay under rate limit
    const delayMs = 60000 / this.requestsPerMinute;
    
    // If we're close to rate limit, wait until reset
    if (this.rateLimit.remaining < 5 && this.rateLimit.reset > 0) {
      const waitTime = (this.rateLimit.reset * 1000) - Date.now();
      if (waitTime > 0) {
        await new Promise(resolve => setTimeout(resolve, waitTime));
      }
    }
    
    const { endpoint, params, resolve, reject } = this.requestQueue.shift();
    
    try {
      // Construct URL with query parameters
      const url = new URL(`${this.baseUrl}${endpoint}`);
      Object.keys(params).forEach(key => {
        url.searchParams.append(key, params[key]);
      });
      
      // Make the request
      const response = await fetch(url.toString());
      
      // Update rate limit tracking
      this.rateLimit = {
        limit: parseInt(response.headers.get('X-RateLimit-Limit') || this.rateLimit.limit),
        remaining: parseInt(response.headers.get('X-RateLimit-Remaining') || this.rateLimit.remaining),
        reset: parseInt(response.headers.get('X-RateLimit-Reset') || this.rateLimit.reset)
      };
      
      // Handle rate limiting
      if (response.status === 429) {
        const retryAfter = parseInt(response.headers.get('Retry-After') || '60');
        console.warn(`Rate limited. Waiting ${retryAfter} seconds before continuing.`);
        
        // Put the request back in the queue
        this.requestQueue.unshift({ endpoint, params, resolve, reject });
        
        // Wait for the retry-after period
        await new Promise(resolve => setTimeout(resolve, retryAfter * 1000));
      } else if (!response.ok) {
        const error = await response.json().catch(() => ({ message: 'Unknown error' }));
        reject(new Error(`API error: ${response.status} - ${error.message}`));
      } else {
        resolve(await response.json());
      }
    } catch (error) {
      reject(error);
    }
    
    // Wait for the calculated delay before processing next request
    await new Promise(resolve => setTimeout(resolve, delayMs));
    
    // Process next request
    this.processQueue();
  }
}

// Usage example:
const api = new ThrottledApiClient({ requestsPerMinute: 30 });
api.request('/seasons/2024/teams', { limit: 50 })
  .then(data => console.log(data))
  .catch(error => console.error(error));
```

### Implementing Exponential Backoff

When dealing with rate limits or server errors, implement exponential backoff to gradually increase wait time between retries:

```python
import requests
import time
import random

def fetch_with_backoff(url, max_retries=5, base_delay=1):
    retries = 0
    
    while retries < max_retries:
        try:
            response = requests.get(url)
            
            # Track rate limits
            rate_limit = {
                'limit': int(response.headers.get('X-RateLimit-Limit', 0)),
                'remaining': int(response.headers.get('X-RateLimit-Remaining', 0)),
                'reset': int(response.headers.get('X-RateLimit-Reset', 0))
            }
            
            # Handle rate limiting
            if response.status_code == 429:
                retry_after = int(response.headers.get('Retry-After', base_delay))
                print(f"Rate limited. Retry after: {retry_after} seconds")
                time.sleep(retry_after)
                retries += 1
                continue
                
            # Handle other errors
            if response.status_code >= 500:
                raise Exception(f"Server error: {response.status_code}")
                
            response.raise_for_status()
            return response.json()
            
        except Exception as e:
            retries += 1
            
            if retries >= max_retries:
                raise Exception(f"Maximum retries reached: {e}")
                
            # Calculate exponential backoff delay
            delay = base_delay * (2 ** retries) + random.uniform(0, 1)
            print(f"Retry {retries}/{max_retries} after {delay:.2f} seconds")
            time.sleep(delay)
    
    raise Exception("Failed after maximum retries")
```

## Strategies for High-Volume Applications

If you need to make a large number of requests, consider these strategies:

### 1. Distribute Across Time

For batch or background processing, distribute requests across a longer timeframe:

```javascript
async function batchProcess(items, processFn, batchSize = 10, delayBetweenBatches = 60000) {
  const batches = [];
  
  // Split items into batches
  for (let i = 0; i < items.length; i += batchSize) {
    batches.push(items.slice(i, i + batchSize));
  }
  
  // Process batches with delay between them
  for (let i = 0; i < batches.length; i++) {
    const batch = batches[i];
    
    // Process current batch
    await Promise.all(batch.map(processFn));
    
    // Wait between batches, except for the last one
    if (i < batches.length - 1) {
      console.log(`Batch ${i+1}/${batches.length} complete. Waiting ${delayBetweenBatches/1000} seconds...`);
      await new Promise(resolve => setTimeout(resolve, delayBetweenBatches));
    }
  }
}

// Example usage:
const teamIds = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15];

async function processTeam(teamId) {
  const url = `https://sports.core.api.espn.com/v2/sports/basketball/leagues/mens-college-basketball/seasons/2024/teams/${teamId}`;
  return fetchWithRateLimitHandling(url);
}

// Process 3 teams at a time, with 1 minute between batches
batchProcess(teamIds, processTeam, 3, 60000);
```

### 2. Implement a Request Scheduler

For ongoing operations, schedule requests to maintain a consistent rate:

```javascript
class RequestScheduler {
  constructor(requestsPerMinute) {
    this.interval = 60000 / requestsPerMinute;
    this.queue = [];
    this.timer = null;
  }
  
  schedule(requestFn) {
    return new Promise((resolve, reject) => {
      this.queue.push({ 
        requestFn, 
        resolve, 
        reject 
      });
      
      if (!this.timer) {
        this.processQueue();
      }
    });
  }
  
  processQueue() {
    if (this.queue.length === 0) {
      this.timer = null;
      return;
    }
    
    const { requestFn, resolve, reject } = this.queue.shift();
    
    requestFn()
      .then(resolve)
      .catch(reject)
      .finally(() => {
        this.timer = setTimeout(() => {
          this.processQueue();
        }, this.interval);
      });
  }
}

// Usage
const scheduler = new RequestScheduler(30); // 30 requests per minute

function fetchTeam(teamId) {
  return scheduler.schedule(() => {
    return fetch(`https://sports.core.api.espn.com/v2/sports/basketball/leagues/mens-college-basketball/seasons/2024/teams/${teamId}`)
      .then(response => response.json());
  });
}

// Now you can call fetchTeam as needed without worrying about rate limits
```

## Conclusion

By understanding and respecting rate limits, you can build applications that interact reliably with the ESPN API. Implement appropriate strategies for your use case:

- For occasional API usage: Basic rate limit handling is sufficient
- For regular usage: Advanced throttling and caching are recommended
- For high-volume applications: Request scheduling and distribution are essential

Remember, good API citizenship helps ensure the service remains stable and available for everyone.

If you have questions about rate limits or need help addressing specific issues, refer to the [Best Practices Guide](best-practices.md) or contact ESPN API support. 