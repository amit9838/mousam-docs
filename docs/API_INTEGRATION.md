# API & Integration Documentation

## Open-Meteo Weather API Integration

### Overview

Mousam integrates with the **Open-Meteo API**, a free, open-source weather API that provides global weather forecasts without requiring authentication.

**API Endpoint**: `https://api.open-meteo.com/v1/`

**Key Features**:
- No authentication required
- Free for non-commercial use
- Global coverage (11km global model + 1km local models)
- Hourly and daily resolution forecasts
- Up to 16-day forecast range
- Historical data available

---

## API Endpoints Used

### 1. Forecast Endpoint

Returns weather forecast data for a specific location.

**Endpoint**: `GET /forecast`

#### Request Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `latitude` | float | Yes | - | Latitude (-90 to 90) |
| `longitude` | float | Yes | - | Longitude (-180 to 180) |
| `hourly` | string | No | - | Comma-separated hourly variables |
| `daily` | string | No | - | Comma-separated daily variables |
| `temperature_unit` | string | No | celsius | Temperature scale (celsius/fahrenheit) |
| `windspeed_unit` | string | No | kmh | Wind speed unit (kmh/mph/ms/kn) |
| `timezone` | string | No | UTC | IANA timezone string (e.g., Asia/Kolkata) |

#### Response Structure

```json
{
  "latitude": 28.6139,
  "longitude": 77.2090,
  "generationtime_ms": 156.3,
  "utc_offset_seconds": 19800,
  "timezone": "Asia/Kolkata",
  "timezone_abbreviation": "IST",
  "elevation": 216.0,
  
  "hourly": {
    "time": ["2025-01-01T00:00", "2025-01-01T01:00", ...],
    "temperature_2m": [25.5, 24.3, 23.8, ...],
    "relative_humidity_2m": [60, 65, 70, ...],
    "precipitation": [0.0, 0.0, 0.1, ...],
    "weather_code": [0, 0, 3, ...],
    "wind_speed_10m": [5.2, 4.8, 6.1, ...],
    "wind_direction_10m": [180, 175, 190, ...],
    "uv_index": [0.0, 0.0, 0.5, ...],
    "cloud_cover": [10, 15, 25, ...]
  },
  
  "daily": {
    "time": ["2025-01-01", "2025-01-02", ...],
    "temperature_2m_max": [32.1, 31.8, 30.5, ...],
    "temperature_2m_min": [18.2, 17.8, 19.1, ...],
    "precipitation_sum": [0.0, 0.3, 1.5, ...],
    "weather_code": [0, 3, 61, ...],
    "wind_speed_10m_max": [8.5, 9.2, 7.8, ...]
  },
  
  "current_units": {
    "temperature": "°C",
    "windspeed": "km/h"
  }
}
```

#### Example Request

```bash
curl -G "https://api.open-meteo.com/v1/forecast" \
  --data-urlencode "latitude=28.6139" \
  --data-urlencode "longitude=77.2090" \
  --data-urlencode "hourly=temperature_2m,precipitation,weather_code,wind_speed_10m,cloud_cover" \
  --data-urlencode "daily=temperature_2m_max,temperature_2m_min,precipitation_sum,weather_code" \
  --data-urlencode "temperature_unit=celsius" \
  --data-urlencode "timezone=Asia/Kolkata"
```

#### Weather Codes (WMO)

| Code | Condition | Description |
|------|-----------|-------------|
| 0 | Clear sky | Sunny, no clouds |
| 1-3 | Mainly clear | Some clouds but mostly clear |
| 45-48 | Foggy | Fog or mist |
| 51-67 | Drizzle/Rain | Light to moderate precipitation |
| 71-77 | Snow | Snow events |
| 80-82 | Rain showers | Sudden rain |
| 85-86 | Snow showers | Sudden snow |
| 95-99 | Thunderstorm | Thunderstorm with hail |

### 2. Geocoding Endpoint

Search for locations by name or get coordinates for a location.

**Endpoint**: `GET /geocoding?name=query`

#### Request Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `name` | string | Yes | Location name to search (min 1 character) |
| `count` | integer | No | Number of results (default: 10) |
| `language` | string | No | Language code for results |

#### Response Structure

```json
{
  "results": [
    {
      "id": 1252431,
      "name": "New York",
      "latitude": 40.7128,
      "longitude": -74.0060,
      "elevation": 10.0,
      "feature_code": "PPL",
      "country_code": "US",
      "country": "United States",
      "admin1_id": 3651335,
      "admin1": "New York",
      "timezone": "America/New_York",
      "population": 8336817,
      "country_population": 331002651
    },
    {
      "id": 5106834,
      "name": "New York City",
      "latitude": 40.7127,
      "longitude": -74.0059,
      "elevation": 9.0,
      ...
    }
  ],
  "generationtime_ms": 50.2
}
```

#### Example Request

```bash
curl -G "https://api.open-meteo.com/v1/geocoding" \
  --data-urlencode "name=Delhi" \
  --data-urlencode "count=5"
```

---

## Implementation in Mousam

### API Client Class

```python
# services/api_client.py

class APIClient:
    """Handles all Open-Meteo API communication"""
    
    BASE_URL = "https://api.open-meteo.com/v1"
    TIMEOUT = 10
    RETRIES = 3
    
    def fetch_forecast(self, latitude: float, longitude: float,
                      hourly_vars: List[str],
                      daily_vars: List[str],
                      timezone: str = "UTC",
                      temperature_unit: str = "celsius") -> dict:
        """
        Fetch weather forecast from Open-Meteo
        
        Args:
            latitude: Location latitude
            longitude: Location longitude
            hourly_vars: List of hourly variables to fetch
            daily_vars: List of daily variables to fetch
            timezone: IANA timezone string
            temperature_unit: celsius or fahrenheit
            
        Returns:
            dict: Raw API response
            
        Raises:
            APIError: On network or parsing errors
            
        Example:
            >>> forecast = client.fetch_forecast(
            ...     28.6139, 77.2090,
            ...     ["temperature_2m", "precipitation"],
            ...     ["temperature_2m_max"]
            ... )
        """
        
        # Build request parameters
        params = {
            "latitude": latitude,
            "longitude": longitude,
            "timezone": timezone,
            "temperature_unit": temperature_unit,
        }
        
        if hourly_vars:
            params["hourly"] = ",".join(hourly_vars)
        if daily_vars:
            params["daily"] = ",".join(daily_vars)
        
        # Retry with exponential backoff
        for attempt in range(self.RETRIES):
            try:
                response = requests.get(
                    f"{self.BASE_URL}/forecast",
                    params=params,
                    timeout=self.TIMEOUT
                )
                response.raise_for_status()
                return response.json()
                
            except requests.Timeout as e:
                if attempt < self.RETRIES - 1:
                    wait_time = 2 ** attempt
                    time.sleep(wait_time)
                    continue
                raise APIError(f"Timeout after {self.RETRIES} retries")
                
            except requests.RequestException as e:
                raise APIError(f"Network error: {e}")
                
            except json.JSONDecodeError:
                raise APIError("Invalid JSON response from API")
    
    def search_locations(self, query: str, count: int = 10) -> List[dict]:
        """
        Search for locations by name
        
        Args:
            query: Location name to search
            count: Number of results to return
            
        Returns:
            List of location dictionaries
            
        Example:
            >>> results = client.search_locations("Delhi", 5)
            >>> for loc in results:
            ...     print(f"{loc['name']}: {loc['latitude']}, {loc['longitude']}")
        """
        
        try:
            response = requests.get(
                f"{self.BASE_URL}/geocoding",
                params={
                    "name": query,
                    "count": count,
                    "language": "en"
                },
                timeout=self.TIMEOUT
            )
            response.raise_for_status()
            data = response.json()
            return data.get("results", [])
            
        except requests.RequestException as e:
            raise APIError(f"Location search failed: {e}")
```

### Variables Mapping

**Hourly Variables Fetched:**
```python
HOURLY_VARIABLES = [
    "temperature_2m",           # Current temperature at 2 meters
    "relative_humidity_2m",     # Relative humidity
    "apparent_temperature",     # Feels-like temperature
    "precipitation",            # Precipitation amount
    "weather_code",             # WMO weather code
    "wind_speed_10m",          # Wind speed at 10 meters
    "wind_direction_10m",      # Wind direction
    "uv_index",                # UV index
    "cloud_cover",             # Cloud coverage percentage
    "pressure_msl",            # Mean sea level pressure
    "visibility",              # Visibility distance
    "wind_gusts_10m",          # Wind gust speed
]

DAILY_VARIABLES = [
    "temperature_2m_max",       # Daily maximum temperature
    "temperature_2m_min",       # Daily minimum temperature
    "precipitation_sum",        # Daily precipitation sum
    "precipitation_probability_max",  # Max precipitation probability
    "weather_code",             # WMO weather code
    "wind_speed_10m_max",      # Daily max wind speed
    "wind_direction_10m_dominant",   # Dominant wind direction
    "uv_index_max",            # Daily max UV index
    "sunshine_duration",        # Hours of sunshine
]
```

### Error Handling

```python
class APIError(Exception):
    """Base exception for API errors"""
    pass

class NetworkError(APIError):
    """Network connectivity error"""
    pass

class TimeoutError(APIError):
    """Request timeout"""
    pass

class RateLimitError(APIError):
    """API rate limit exceeded"""
    pass

class InvalidResponseError(APIError):
    """Invalid/malformed response"""
    pass

# Usage in fetch_forecast:
try:
    data = client.fetch_forecast(...)
except RateLimitError:
    # Wait and retry later
    schedule_retry(delay=60)
except NetworkError:
    # Use cached data if available
    return get_cached_weather()
except APIError as e:
    # Log and show error to user
    logger.error(f"API error: {e}")
    show_error_dialog(str(e))
```

---

## Caching Strategy

### Cache Structure

```python
# ~/.config/mousam/cache/weather_cache.json

{
  "cache_entries": {
    "28.6139_77.2090": {  # Key: latitude_longitude
      "location": {
        "name": "Delhi",
        "latitude": 28.6139,
        "longitude": 77.2090,
        "timezone": "Asia/Kolkata"
      },
      "data": {
        # Full forecast data
      },
      "timestamp": 1704067200,  # When cached
      "ttl": 1800,             # 30 minutes
      "is_expired": false
    }
  }
}
```

### Cache Implementation

```python
# services/cache.py

import json
import time
from pathlib import Path
from datetime import datetime

class CacheManager:
    """Manages local weather data persistence"""
    
    def __init__(self, cache_dir: Path = None):
        if cache_dir is None:
            cache_dir = Path.home() / ".config" / "mousam" / "cache"
        
        self.cache_dir = cache_dir
        self.cache_file = cache_dir / "weather_cache.json"
        self.cache_dir.mkdir(parents=True, exist_ok=True)
        self._load_cache()
    
    def _load_cache(self):
        """Load cache from disk"""
        if self.cache_file.exists():
            try:
                with open(self.cache_file, 'r') as f:
                    self._cache = json.load(f)
            except (json.JSONDecodeError, IOError):
                self._cache = {"cache_entries": {}}
        else:
            self._cache = {"cache_entries": {}}
    
    def get(self, location_key: str) -> Optional[dict]:
        """
        Retrieve cached weather if fresh
        
        Args:
            location_key: "latitude_longitude" string
            
        Returns:
            Weather data dict if cache hit and not expired, None otherwise
            
        Example:
            >>> cached = cache.get("28.6139_77.2090")
            >>> if cached:
            ...     print(f"Using cached data from {cached['timestamp']}")
        """
        
        if location_key not in self._cache["cache_entries"]:
            return None
        
        entry = self._cache["cache_entries"][location_key]
        
        # Check if expired
        age = time.time() - entry["timestamp"]
        if age > entry["ttl"]:
            entry["is_expired"] = True
            return None
        
        return entry["data"]
    
    def get_with_expiry(self, location_key: str) -> tuple[Optional[dict], bool]:
        """
        Get cached data, including expired entries
        
        Returns:
            (data, is_expired) tuple
            
        Useful for showing stale data while refreshing
        """
        
        if location_key not in self._cache["cache_entries"]:
            return None, False
        
        entry = self._cache["cache_entries"][location_key]
        age = time.time() - entry["timestamp"]
        is_expired = age > entry["ttl"]
        
        return entry["data"], is_expired
    
    def set(self, location_key: str, data: dict, ttl: int = 1800):
        """
        Store weather data with TTL
        
        Args:
            location_key: "latitude_longitude" string
            data: Weather forecast data
            ttl: Time-to-live in seconds (default: 30 minutes)
            
        Example:
            >>> cache.set("28.6139_77.2090", forecast_data)
        """
        
        self._cache["cache_entries"][location_key] = {
            "data": data,
            "timestamp": time.time(),
            "ttl": ttl,
            "is_expired": False
        }
        
        self._save_cache()
    
    def _save_cache(self):
        """Persist cache to disk"""
        try:
            with open(self.cache_file, 'w') as f:
                json.dump(self._cache, f, indent=2)
        except IOError as e:
            logger.error(f"Failed to save cache: {e}")
    
    def clear_all(self):
        """Clear all cached entries"""
        self._cache = {"cache_entries": {}}
        self._save_cache()
    
    def cleanup_expired(self):
        """Remove expired cache entries"""
        current_time = time.time()
        expired_keys = []
        
        for key, entry in self._cache["cache_entries"].items():
            age = current_time - entry["timestamp"]
            if age > entry["ttl"]:
                expired_keys.append(key)
        
        for key in expired_keys:
            del self._cache["cache_entries"][key]
        
        if expired_keys:
            self._save_cache()
```

### Cache Invalidation Strategy

```python
# When to invalidate cache:

# 1. Manual user refresh
def on_refresh_clicked():
    cache.delete(location_key)  # Force fresh fetch
    fetch_weather()

# 2. Location changed
def on_location_changed(new_location):
    # Old cache not invalidated (might return later)
    # Just fetch new location
    fetch_weather(new_location)

# 3. Settings changed (units)
def on_units_changed():
    # Cache is still valid, just display differently
    # No invalidation needed

# 4. Time-based expiration
# Automatic via TTL check in CacheManager.get()

# 5. App startup cleanup
def on_app_startup():
    cache_manager.cleanup_expired()  # Remove old entries
```

---

## Rate Limiting & Throttling

### Rate Limit Handling

```python
class APIClient:
    def __init__(self):
        self.rate_limiter = RateLimiter(
            max_requests=60,  # 60 requests per minute
            window=60         # 1 minute window
        )
    
    def fetch_forecast(self, ...):
        # Check rate limit before request
        if not self.rate_limiter.allow_request():
            raise RateLimitError(
                "Rate limit exceeded. Wait before retrying."
            )
        
        # Make request
        response = requests.get(...)
        
        # Check response headers for rate limit info
        if response.status_code == 429:
            retry_after = int(response.headers.get('Retry-After', 60))
            raise RateLimitError(f"Wait {retry_after} seconds")
        
        return response.json()

class RateLimiter:
    """Token bucket rate limiter"""
    
    def __init__(self, max_requests: int, window: int):
        self.max_requests = max_requests
        self.window = window
        self.requests = deque()
    
    def allow_request(self) -> bool:
        """Check if next request is allowed"""
        now = time.time()
        
        # Remove old requests outside window
        while self.requests and self.requests[0] < now - self.window:
            self.requests.popleft()
        
        # Check if below limit
        if len(self.requests) < self.max_requests:
            self.requests.append(now)
            return True
        
        return False
```

---

## Data Transformation

### API Response → Application Models

```python
from dataclasses import dataclass
from typing import List
from datetime import datetime

@dataclass
class CurrentWeather:
    """Current weather conditions"""
    temperature: float  # In °C or °F
    apparent_temperature: float
    humidity: int  # 0-100
    weather_code: int
    wind_speed: float
    wind_direction: int  # 0-360 degrees
    wind_gust: float
    uv_index: float
    cloud_cover: int  # 0-100
    pressure: float  # hPa
    visibility: float  # meters
    timestamp: datetime

@dataclass
class HourlyData:
    """Single hour of forecast data"""
    time: datetime
    temperature: float
    humidity: int
    precipitation: float
    weather_code: int
    wind_speed: float
    wind_direction: int

@dataclass
class DailyData:
    """Single day of forecast data"""
    date: datetime
    temp_max: float
    temp_min: float
    precipitation: float
    weather_code: int

class WeatherTransformer:
    """Transform API response to application models"""
    
    @staticmethod
    def transform_current(api_response: dict,
                         temperature_unit: str) -> CurrentWeather:
        """Extract current weather from API response"""
        
        hourly = api_response["hourly"]
        # Use first entry (current time)
        current_data = {
            key: values[0]
            for key, values in hourly.items()
        }
        
        return CurrentWeather(
            temperature=current_data["temperature_2m"],
            apparent_temperature=current_data["apparent_temperature"],
            humidity=current_data["relative_humidity_2m"],
            weather_code=current_data["weather_code"],
            wind_speed=current_data["wind_speed_10m"],
            wind_direction=current_data["wind_direction_10m"],
            wind_gust=current_data.get("wind_gusts_10m", 0),
            uv_index=current_data["uv_index"],
            cloud_cover=current_data["cloud_cover"],
            pressure=current_data["pressure_msl"],
            visibility=current_data.get("visibility", 10000),
            timestamp=datetime.now()
        )
    
    @staticmethod
    def transform_hourly(api_response: dict) -> List[HourlyData]:
        """Extract 24-hour forecast from API response"""
        
        hourly = api_response["hourly"]
        times = [datetime.fromisoformat(t) for t in hourly["time"][:24]]
        
        hourly_data = []
        for i, time in enumerate(times):
            hourly_data.append(HourlyData(
                time=time,
                temperature=hourly["temperature_2m"][i],
                humidity=hourly["relative_humidity_2m"][i],
                precipitation=hourly["precipitation"][i],
                weather_code=hourly["weather_code"][i],
                wind_speed=hourly["wind_speed_10m"][i],
                wind_direction=hourly["wind_direction_10m"][i]
            ))
        
        return hourly_data
    
    @staticmethod
    def transform_daily(api_response: dict) -> List[DailyData]:
        """Extract 7-day forecast from API response"""
        
        daily = api_response["daily"]
        times = [datetime.fromisoformat(t).date() for t in daily["time"][:7]]
        
        daily_data = []
        for i, date in enumerate(times):
            daily_data.append(DailyData(
                date=date,
                temp_max=daily["temperature_2m_max"][i],
                temp_min=daily["temperature_2m_min"][i],
                precipitation=daily["precipitation_sum"][i],
                weather_code=daily["weather_code"][i]
            ))
        
        return daily_data
```

---

## Integration Example

Complete example of fetching and displaying weather:

```python
# In weather_service.py

class WeatherService:
    def __init__(self, api_client: APIClient, cache: CacheManager):
        self.api_client = api_client
        self.cache = cache
    
    def get_weather(self, location: Location) -> dict:
        """
        Fetch weather for location with caching
        
        Example:
            >>> service = WeatherService(api_client, cache)
            >>> weather = service.get_weather(
            ...     Location("Delhi", 28.6139, 77.2090)
            ... )
            >>> print(f"Current: {weather['current'].temperature}°C")
        """
        
        location_key = f"{location.latitude}_{location.longitude}"
        
        # Try cache first
        cached = self.cache.get(location_key)
        if cached:
            return self._format_response(cached, location)
        
        try:
            # Fetch from API
            raw_response = self.api_client.fetch_forecast(
                latitude=location.latitude,
                longitude=location.longitude,
                hourly_vars=HOURLY_VARIABLES,
                daily_vars=DAILY_VARIABLES,
                timezone=location.timezone
            )
            
            # Transform data
            weather_data = {
                "current": WeatherTransformer.transform_current(raw_response),
                "hourly": WeatherTransformer.transform_hourly(raw_response),
                "daily": WeatherTransformer.transform_daily(raw_response),
                "location": location,
                "timestamp": datetime.now(),
                "raw": raw_response
            }
            
            # Cache for next time
            self.cache.set(location_key, weather_data)
            
            return self._format_response(weather_data, location)
        
        except APIError as e:
            logger.error(f"Failed to fetch weather: {e}")
            
            # Try stale cache
            stale_data, is_expired = self.cache.get_with_expiry(location_key)
            if stale_data:
                logger.info("Using stale cached data")
                return self._format_response(stale_data, location, stale=True)
            
            # No cache available
            raise WeatherFetchError(
                f"Failed to fetch weather: {e}\n"
                "Please check your internet connection."
            )
    
    def _format_response(self, data: dict, location: Location,
                        stale: bool = False) -> dict:
        """Format weather data for UI"""
        return {
            "current": data["current"],
            "hourly": data["hourly"],
            "daily": data["daily"],
            "location": location,
            "is_stale": stale
        }
```

---

## Error Codes & Status Codes

### HTTP Status Codes

| Code | Meaning | Action |
|------|---------|--------|
| 200 | OK | Success, process response |
| 400 | Bad Request | Invalid parameters, check coordinates |
| 401 | Unauthorized | API key issue (not applicable for Open-Meteo) |
| 429 | Too Many Requests | Rate limited, implement backoff |
| 500-599 | Server Error | Temporary failure, retry later |

### Custom Error Codes

| Code | Description | Recovery |
|------|-------------|----------|
| NETWORK_ERROR | No internet connection | Show offline mode, use cache |
| API_TIMEOUT | Request took too long | Retry with longer timeout |
| INVALID_RESPONSE | Malformed API response | Log error, use cached data |
| LOCATION_NOT_FOUND | Location search returned no results | Show error, try different search |
| CACHE_CORRUPTED | Cache file is invalid | Clear cache, fetch fresh |

---

**This documentation provides comprehensive API integration details for both users and developers.**