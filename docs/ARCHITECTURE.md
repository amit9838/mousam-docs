# Architecture & Design Documentation

## In-Depth System Architecture

### 1. Layered Architecture Overview

Mousam follows a **layered architecture** pattern with clear separation of concerns:

```
┌──────────────────────────────────────────────────────────┐
│ PRESENTATION LAYER                                       │
│ ┌──────────────┐  ┌──────────────┐  ┌──────────────┐   │
│ │ Main Window  │  │ Dialog Views │  │ Preferences  │   │
│ └──────────────┘  └──────────────┘  └──────────────┘   │
└──────────────────────────────────────────────────────────┘
                         ▲
                         │ GTK Signals
                         │
┌──────────────────────────────────────────────────────────┐
│ BUSINESS LOGIC LAYER                                     │
│ ┌──────────────┐  ┌──────────────┐  ┌──────────────┐   │
│ │Weather       │  │Location      │  │Preferences   │   │
│ │Service       │  │Service       │  │Manager       │   │
│ └──────────────┘  └──────────────┘  └──────────────┘   │
└──────────────────────────────────────────────────────────┘
                         ▲
                         │
┌──────────────────────────────────────────────────────────┐
│ DATA ACCESS LAYER                                        │
│ ┌──────────────┐  ┌──────────────┐  ┌──────────────┐   │
│ │API Client    │  │Cache Manager │  │Config Store  │   │
│ └──────────────┘  └──────────────┘  └──────────────┘   │
└──────────────────────────────────────────────────────────┘
                         ▲
                         │
┌──────────────────────────────────────────────────────────┐
│ EXTERNAL INTERFACES                                      │
│ ┌──────────────┐  ┌──────────────┐  ┌──────────────┐   │
│ │Open-Meteo    │  │File System   │  │System APIs   │   │
│ │API           │  │(XDG)         │  │(D-Bus, etc)  │   │
│ └──────────────┘  └──────────────┘  └──────────────┘   │
└──────────────────────────────────────────────────────────┘
```

### 2. Component Interaction Diagrams

#### Current Weather Display Flow

```
User Launches App
       │
       ▼
┌─────────────────┐
│MouseamApp       │ Initialize GTK
│(app.py)         │ Load resources
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│MouseamWindow    │ Create main UI
│(window.py)      │ Load settings
└────────┬────────┘
         │
         ▼
┌─────────────────────────────────────┐
│LocationService                      │
│- Load last selected location        │
│- Or use default location            │
└────────┬────────────────────────────┘
         │
         ▼
┌─────────────────────────────────────┐
│WeatherService.get_weather()         │
│ 1. Check cache validity             │
│ 2. If cache miss: fetch from API    │
│ 3. Validate and transform data      │
│ 4. Return Weather model             │
└────────┬────────────────────────────┘
         │
         ├─────────────────────────────┐
         │                             │
         ▼                             ▼
    Cache Hit?               Cache Miss?
         │                        │
         ▼                        ▼
    Return from            ┌──────────────┐
    Cache                  │APIClient     │
         │                 │.fetch()      │
         │                 │- HTTP req    │
         │                 │- Parse JSON  │
         │                 └──────┬───────┘
         │                        │
         │        ┌───────────────┤
         │        │               │
         │        ▼               ▼
         │    Success?        Error
         │        │             │
         │        ▼             ▼
         │     Store in    Fallback to
         │     Cache       Stale Cache
         │        │             │
         └────────┴─────────────┘
                  │
                  ▼
         ┌────────────────────┐
         │Transform to Models │
         │- Current Weather   │
         │- Hourly Forecast   │
         │- Daily Forecast    │
         └────────┬───────────┘
                  │
                  ▼
         ┌────────────────────┐
         │Update UI Components│
         │- Temperature card  │
         │- Metrics display   │
         │- Forecast charts   │
         └────────┬───────────┘
                  │
                  ▼
         Display Weather


```

#### API Call Detail

```
APIClient.fetch_forecast()
│
├─ Build URL
│  url = "https://api.open-meteo.com/v1/forecast"
│  params = {
│    "latitude": 28.6139,
│    "longitude": 77.2090,
│    "hourly": "temperature_2m,precipitation,..."
│    "daily": "temperature_2m_max,..."
│    "timezone": "Asia/Kolkata"
│  }
│
├─ Execute Request
│  try:
│    response = requests.get(url, params=params, timeout=10)
│  except requests.Timeout:
│    retry_with_backoff()  # Exponential backoff
│
├─ Parse Response
│  {
│    "latitude": 28.6139,
│    "hourly": {
│      "time": ["2025-01-01T00:00", ...],
│      "temperature_2m": [25.5, 24.3, ...],
│      "precipitation": [0, 0.1, ...],
│      ...
│    },
│    "daily": {
│      "temperature_2m_max": [32, 31, ...],
│      "temperature_2m_min": [18, 17, ...],
│      ...
│    }
│  }
│
└─ Return to Caller
   weather_data = response.json()
```

### 3. State Management

#### Application State Lifecycle

```
START
  │
  ├─ Load User Settings
  │  └─ Cache TTL config
  │  └─ Temperature unit preference
  │  └─ Saved locations
  │
  ├─ Initialize Services
  │  ├─ WeatherService
  │  ├─ LocationService
  │  ├─ CacheManager
  │  └─ PreferencesManager
  │
  ├─ Show Main Window
  │  ├─ Restore window size/position
  │  ├─ Load last selected location
  │  └─ Fetch initial weather
  │
  ├─ EVENT LOOP
  │  ├─ Handle User Input
  │  │  ├─ Location change → Fetch new weather
  │  │  ├─ Units toggle → Refresh display
  │  │  ├─ Refresh click → Force API call
  │  │  └─ Settings change → Update app behavior
  │  │
  │  ├─ Handle Async Operations
  │  │  ├─ API responses → Update UI
  │  │  ├─ Cache updates → Persist to disk
  │  │  └─ Timer events → Periodic refresh
  │  │
  │  └─ Handle System Events
  │     ├─ Network state change
  │     ├─ System time change
  │     └─ Theme preference change
  │
  └─ SHUTDOWN
     ├─ Save window state
     ├─ Save current location
     ├─ Cleanup resources
     └─ Exit cleanly
```

### 4. Data Flow Patterns

#### Pattern 1: Synchronous Data Retrieval (Cached)

```
Request Weather
    │
    ▼
CacheManager.get(location_id)
    │
    ├─ Cache Hit (fresh) ──────────────► Return cached data
    │                                          │
    │                                          ▼
    │                                    Update UI immediately
    │                                          │
    │                                    (Background refresh)
    │                                          │
    │                                    └──────┐
    └─ Cache Miss ────────────────────────────┘
        │
        ▼
    APIClient.fetch()
        │
        ├─ Success ──► Parse ──► Store in cache ──► Update UI
        │
        └─ Failure
            │
            ├─ Expired cache? ──► Show with warning
            └─ No cache? ──► Show error dialog
```

#### Pattern 2: Async Background Update

```
Timer fires (30 min interval)
    │
    ▼
Emit "time_to_refresh" signal
    │
    ▼
Create background thread
    │
    ├─ Main thread: Continue responsive
    │
    └─ Worker thread:
        │
        ├─ Fetch fresh data
        │
        ├─ Update cache
        │
        └─ Emit "data_ready" signal
           │
           ▼
        Back to main thread
        (via GLib.idle_add)
           │
           ▼
        Update UI
```

### 5. Service Architecture Details

#### WeatherService

```
class WeatherService:
    """Orchestrates weather data retrieval and caching"""
    
    - api_client: APIClient
    - cache_manager: CacheManager
    - location_service: LocationService
    
    Methods:
    
    async get_weather(location: Location) -> Weather:
        """
        Fetches weather, using cache if available.
        
        Flow:
        1. Build cache key from location ID
        2. Check cache.get(key)
        3. If fresh: return cached
        4. Else: call API
        5. Transform API response
        6. Update cache
        7. Return weather data
        """
        
    async refresh_weather(location: Location) -> Weather:
        """
        Force refresh from API, bypass cache.
        
        Used when:
        - User clicks refresh button
        - Manual update requested
        - Cache explicitly cleared
        """
        
    async get_hourly_forecast(location: Location, hours: int) -> HourlyForecast:
        """Get detailed hourly breakdown"""
        
    async get_daily_forecast(location: Location, days: int) -> List[DailyForecast]:
        """Get daily weather summary"""
```

#### CacheManager

```
class CacheManager:
    """Manages local weather data persistence"""
    
    Storage structure:
    ~/.config/mousam/cache/
    ├── weather_cache.json       # Current and hourly
    └── forecast_cache.json      # Daily forecasts
    
    Cache Entry Format:
    {
        "location_id": "28.6139_77.2090",
        "data": { ... weather data ... },
        "timestamp": 1700000000,
        "ttl": 1800,  # 30 minutes
        "is_expired": false
    }
    
    Methods:
    
    def get(key: str) -> Optional[Dict]:
        """
        Check if cached data is fresh
        
        1. Load from disk
        2. Calculate age
        3. Compare to TTL
        4. Mark if expired
        5. Return if valid
        """
        
    def set(key: str, data: Dict, ttl: int = 1800):
        """Store data with TTL"""
        
    def is_expired(timestamp: float, ttl: int) -> bool:
        """Check if cache entry passed TTL"""
        
    def clear_all():
        """Clear all cached data"""
        
    def cleanup_expired():
        """Remove old expired entries"""
```

#### LocationService

```
class LocationService:
    """Manages location selection and geolocation"""
    
    Methods:
    
    def get_saved_locations() -> List[Location]:
        """Load user's saved locations from disk"""
        
    def save_location(location: Location) -> bool:
        """Persist location to storage"""
        
    def search_locations(query: str) -> List[Location]:
        """
        Search for locations by name
        
        Uses Open-Meteo Geocoding API:
        GET /geocoding?name=New%20York
        
        Returns:
        [
            {
                "id": 5128581,
                "name": "New York",
                "latitude": 40.7128,
                "longitude": -74.0060,
                "country": "United States",
                "timezone": "America/New_York"
            },
            ...
        ]
        """
        
    def get_current_location() -> Optional[Location]:
        """
        Get device's current location
        
        Attempts (in order):
        1. Last saved location
        2. Geolocation API (if permitted)
        3. Default fallback location
        """
```

### 6. Error Handling Strategy

#### Exception Hierarchy

```
MouseamError (base)
├── APIError
│   ├── NetworkError
│   │   ├── ConnectionTimeout
│   │   ├── ConnectionRefused
│   │   └── DNSResolutionError
│   ├── HTTPError
│   │   ├── ClientError (4xx)
│   │   ├── ServerError (5xx)
│   │   └── RateLimitError
│   └── DataParseError
├── CacheError
│   ├── CacheCorrupted
│   ├── CacheWriteFailed
│   └── InvalidCacheEntry
├── LocationError
│   ├── LocationNotFound
│   └── GeolocationDenied
└── ConfigError
    ├── SettingsCorrupted
    └── PermissionDenied
```

#### Error Recovery Strategies

```
APIError
  │
  ├─ NetworkError
  │  ├─ Retry with exponential backoff
  │  │  (1s, 2s, 4s, 8s)
  │  │
  │  └─ After 3 failures:
  │     ├─ Use cached data if available
  │     ├─ Display offline indicator
  │     └─ Show "Retry" button
  │
  ├─ RateLimitError (429)
  │  ├─ Wait and retry
  │  │  (Extract Retry-After header)
  │  │
  │  └─ Queue request for later
  │
  └─ DataParseError
     ├─ Log error details
     ├─ Use fallback data
     └─ Alert user to app issue

CacheError
  └─ Attempt auto-recovery:
     ├─ Rebuild cache from API
     ├─ If fails: disable caching
     └─ Continue with live data only
```

### 7. Threading & Concurrency

#### Thread Model

```
Main Thread (GTK Event Loop)
│
├─ Handles all UI updates
├─ Processes user input
├─ Runs GTK signal handlers
└─ Triggers worker threads
   
Worker Threads (for I/O operations)
│
├─ APIClient.fetch()
│  │ (Network I/O - non-blocking)
│  └─ Result → GLib.idle_add() → Main thread
│
├─ CacheManager.read/write()
│  │ (Disk I/O - non-blocking)
│  └─ Result → GLib.idle_add() → Main thread
│
└─ LocationService.search()
   │ (Network I/O - non-blocking)
   └─ Result → GLib.idle_add() → Main thread

Critical: Never call GTK functions from worker threads!
Use: GLib.idle_add(func, arg) to queue back to main thread
```

#### Concurrency Pattern

```python
def on_refresh_button_clicked(self):
    """Handle refresh button - non-blocking"""
    
    # Start background task
    thread = threading.Thread(
        target=self._fetch_weather_async,
        daemon=True
    )
    thread.start()
    
    # Main thread continues immediately
    self.show_loading_indicator()

def _fetch_weather_async(self):
    """Run in background, never touch UI directly"""
    
    try:
        weather = self.service.fetch_weather(
            self.current_location
        )
        
        # Queue UI update back to main thread
        GLib.idle_add(
            self._update_weather_ui,
            weather
        )
        
    except APIError as e:
        GLib.idle_add(
            self._show_error_dialog,
            str(e)
        )

def _update_weather_ui(self, weather):
    """Called in main thread - safe to update UI"""
    self.temperature_label.set_text(
        f"{weather.current.temp}°C"
    )
```

### 8. Memory Management

#### Resource Lifecycle

```
Application Start
    │
    ├─ Load Resources
    │  ├─ .gresource (UI files, icons, CSS)
    │  ├─ Configuration files
    │  └─ Theme assets
    │
    ├─ Cache Loaded Data
    │  ├─ Recent locations (in memory)
    │  ├─ User preferences (in memory)
    │  └─ Weather data (in memory)
    │
    └─ Create UI Objects
       ├─ Main window
       ├─ GTK widgets
       └─ Signal handlers

Runtime Memory Usage:
- Current weather object: ~1-2 MB
- Forecast data (hourly): ~2-3 MB
- Daily forecast: ~0.5 MB
- UI elements: ~5-10 MB
- Total resident: ~20-30 MB

Application Shutdown
    │
    ├─ Disconnect signals
    ├─ Cleanup resources
    ├─ Save application state
    └─ Exit GTK main loop
```

---

## Design Patterns Used

### 1. Model-View-Controller (MVC)

```
Model (Data Layer):
- Weather, Location, Forecast dataclasses
- Represent domain entities
- No UI or business logic

View (Presentation Layer):
- GTK widgets and windows
- Display data
- Generate signals from user input

Controller (Business Logic):
- WeatherService, LocationService
- Process user actions
- Update models
- Notify views of changes
```

### 2. Dependency Injection

```python
# Example: Services are injected
class MouseamWindow(Gtk.ApplicationWindow):
    def __init__(self, weather_service, location_service):
        self.weather_service = weather_service
        self.location_service = location_service
        
        # Services can be mocked for testing
        # Decouples components
        # Easy to extend
```

### 3. Observer Pattern

```python
# GTK signals implement observer pattern
self.location_combo.connect(
    "changed",  # Signal name
    self.on_location_changed  # Observer method
)

# When location changes, callback is invoked
def on_location_changed(self, combo):
    """Observer gets notified of change"""
    self.fetch_weather_for_selected_location()
```

### 4. Singleton Pattern

```python
class CacheManager:
    """Ensure only one instance exists"""
    _instance = None
    
    def __new__(cls):
        if cls._instance is None:
            cls._instance = super().__new__(cls)
        return cls._instance
    
    # Single shared cache across app
```

### 5. Repository Pattern

```python
class SettingsRepository:
    """Abstraction for configuration storage"""
    
    def save(self, key: str, value: Any):
        """Could use JSON, GSettings, DB, etc."""
        pass
    
    def load(self, key: str) -> Any:
        """Implementation detail hidden"""
        pass
```

### 6. Async Task Pattern

```python
def execute_async(func, callback):
    """
    Run long operation without blocking UI
    
    - func: Long-running function
    - callback: Called when complete
    """
    thread = threading.Thread(target=func)
    thread.daemon = True
    thread.start()
    
    # Result passed to callback in main thread
```

---

## Performance Considerations

### Optimization Techniques

#### 1. Caching

**Multi-level caching strategy:**
- **Memory cache**: Current request results
- **Disk cache**: Persistent across sessions
- **API cache**: Server-side response caching

```python
# Check before API call
cached = cache.get(location_key)
if cached and not cached.is_expired():
    return cached_data  # Instant response

# Otherwise fetch
fresh_data = api.fetch()
cache.set(location_key, fresh_data)
```

#### 2. Lazy Loading

```python
# Don't fetch 7-day forecast until user clicks tab
class DailyForecastPage:
    def __init__(self):
        self.forecast_data = None
    
    def on_page_visible(self):
        """Only load when actually needed"""
        if self.forecast_data is None:
            self.fetch_forecast()
```

#### 3. Data Pagination

```python
# Load hourly forecast in chunks
class HourlyChart:
    def load_next_hours(self, count=6):
        """Load next 6 hours of data"""
        # Reduces memory usage
        # Smoother scrolling
```

#### 4. UI Virtualization

```python
# Only render visible rows
class DailyForecastList(Gtk.ListView):
    # GTK4 automatically virtualizes list items
    # Only visible items are rendered
    # Saves memory for large lists
```

### Memory Profiling

```bash
# Monitor memory usage
python3 -m memory_profiler src/mousam/__main__.py

# Profile specific functions
@profile
def fetch_weather():
    # Function marked for profiling
    pass
```

### Performance Benchmarks

```
Operation                  Time        Notes
─────────────────────────────────────────────────
App startup                ~500ms      (first time)
Location search (10 chars)  ~200ms      network call
Weather fetch (cached)      ~50ms       memory access
Weather fetch (API)         ~1-2s       network + processing
Theme switch                ~100ms      CSS recomputation
Forecast chart render       ~300ms      Cairo drawing
```

---

## Security Considerations

### Input Validation

```python
# Sanitize location search input
def search_locations(query: str) -> List[Location]:
    # Validate input
    if not query or len(query) > 200:
        raise ValidationError("Invalid search query")
    
    # Sanitize before API call
    safe_query = urllib.parse.quote(query)
    
    # Make API request
    return api.search(safe_query)
```

### API Security

```python
# Always use HTTPS for API calls
api_url = "https://api.open-meteo.com/v1/forecast"
# Never http://

# Timeout to prevent hanging
response = requests.get(
    api_url,
    timeout=10  # 10 second timeout
)

# Validate response
if response.status_code not in [200, 204]:
    raise APIError(f"HTTP {response.status_code}")
```

### Data Privacy

- **No user tracking**: Mousam doesn't collect personal data
- **Local storage only**: Settings and cache stored locally
- **No analytics**: No telemetry or usage reporting
- **API calls**: Only to Open-Meteo for weather data
  - Location coordinates sent only to fetch weather
  - No user ID or tracking information

### XSS Prevention (for web version)

```python
# Escape HTML special characters
def safe_display_text(text: str) -> str:
    return (
        text
        .replace("&", "&amp;")
        .replace("<", "&lt;")
        .replace(">", "&gt;")
        .replace('"', "&quot;")
    )
```

---

## Scalability & Extensibility

### Adding New Weather Metrics

```python
# Step 1: Update data model
@dataclass
class CurrentWeather:
    temperature: float
    humidity: int
    air_quality_index: int  # New field

# Step 2: Update API client
def fetch_forecast(self, lat, lon):
    hourly_vars = [
        "temperature_2m",
        "relative_humidity_2m",
        "air_quality",  # New variable
    ]
    return self.api_call(hourly=hourly_vars)

# Step 3: Create UI widget
class AirQualityCard(Gtk.Box):
    def set_data(self, aqi: int):
        self.value_label.set_text(f"AQI: {aqi}")
        self.set_color_from_aqi(aqi)

# Step 4: Update main window
class MouseamWindow:
    def __init__(self):
        self.aqi_card = AirQualityCard()
        self.main_box.append(self.aqi_card)
    
    def update_weather_display(self, weather):
        self.aqi_card.set_data(
            weather.current.air_quality_index
        )
```

### Supporting Alternative APIs

```python
# Abstract API client
class WeatherAPIClient(ABC):
    @abstractmethod
    def fetch_forecast(self, lat, lon):
        pass

# Implementations
class OpenMeteoClient(WeatherAPIClient):
    def fetch_forecast(self, lat, lon):
        # Open-Meteo specific
        pass

class WeatherAPIClient(WeatherAPIClient):
    def fetch_forecast(self, lat, lon):
        # Weather API specific
        pass

# Switch via configuration
api_client = (
    OpenMeteoClient()
    if config.API_PROVIDER == "openmeteo"
    else WeatherAPIClient()
)
```

---

## Testing Strategy

### Unit Testing

```python
# Test weather service
class TestWeatherService:
    def test_get_weather_returns_valid_model(self):
        service = WeatherService(
            api_client=MockAPIClient(),
            cache=MockCache()
        )
        weather = service.get_weather(
            Location("Delhi", 28.6, 77.2)
        )
        assert weather.current.temperature > -100
        assert weather.current.temperature < 150
    
    def test_cache_hit_returns_cached_data(self):
        service = WeatherService(
            api_client=MockAPIClient(calls=0),
            cache=MockCache(returns_valid=True)
        )
        # Should not call API
        service.get_weather(location)
        assert service.api_client.calls == 0
```

### Integration Testing

```python
# Test API integration
class TestAPIIntegration:
    def test_fetch_real_forecast(self):
        client = APIClient()
        result = client.fetch_forecast(28.6, 77.2)
        
        # Validate response structure
        assert "hourly" in result
        assert "daily" in result
        assert "latitude" in result
```

### UI Testing

```bash
# Use GTK test utilities
GTK_DEBUG=interactive mousam  # Opens inspector

# Accessibility testing
accerciser  # GTK accessibility inspector
```

---

## Deployment & Distribution

### Build Configuration

```meson
# meson.build structure
project('mousam', 'python',
  version: '1.4.0',
  description: 'Weather at a glance',
)

# Python module
py = import('python')
py_installation = py.find_installation('python3', ...)

# Define install directories
install_dir = py.get_install_dir()
data_dir = get_option('datadir')
locale_dir = get_option('localedir')

# Install application
install_subdir('src/mousam', install_dir: install_dir)
install_subdir('data/icons', install_dir: data_dir)
```

### Flatpak Manifest

```yaml
app-id: io.github.amit9838.mousam
runtime: org.gnome.Platform
runtime-version: '46'

modules:
  - name: mousam
    buildsystem: meson
    sources:
      - type: git
        url: https://github.com/amit9838/mousam.git
```

---

**This architecture ensures Mousam is maintainable, scalable, and follows industry best practices for desktop application development.**