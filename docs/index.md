# Mousam - Weather at a Glance

A modern, lightweight GTK4/Libadwaita-based desktop weather application for Linux that provides real-time weather information, detailed forecasts, and beautiful data visualizations using the free Open-Meteo API.

**Current Version:** 1.4.0  
**License:** GPLv3  
**Repository:** https://github.com/amit9838/mousam  
**Flathub Store:** https://flathub.org/en/apps/io.github.amit9838.mousam

---

## Table of Contents

1. [Overview](#overview)
2. [Key Features](#key-features)
3. [Technology Stack](#technology-stack)
4. [Architecture Overview](#architecture-overview)
5. [Installation Guide](#installation-guide)
6. [Build from Source](#build-from-source)
7. [Configuration](#configuration)
8. [Usage Guide](#usage-guide)
9. [API Integration](#api-integration)
10. [Development Guide](#development-guide)
11. [Contributing](#contributing)
12. [Support & Credits](#support--credits)

---

## Overview

Mousam is a production-grade desktop weather application designed for GNOME desktop environments and other GTK-compatible platforms. It follows GNOME Human Interface Guidelines (HIG) and delivers a seamless user experience with responsive design, dark mode support, and adaptive layouts for different screen sizes.

The application fetches real-time weather data from the **Open-Meteo API** - a free, open-source weather API that provides high-quality forecasts without requiring API keys, making it ideal for open-source projects.

### Target Users

- Linux desktop users who want a native weather application
- Developers interested in GTK4/Libadwaita application development
- Contributors to the open-source ecosystem

### Why Mousam?

- **Zero Configuration**: No API keys required - works out of the box
- **Privacy-Focused**: All data is fetched from free, open-source weather models
- **Modern UI**: Built with GTK4 and Libadwaita following GNOME design standards
- **Lightweight**: Minimal dependencies, fast performance
- **Cross-Desktop**: Works on GNOME, KDE, and other Linux desktops

---

## Key Features

### Core Weather Information
- **Real-time Display**: Current temperature, humidity, wind speed, UV index, pressure
- **Multiple Measurement Units**: Switch between metric (Celsius, m/s) and imperial (Fahrenheit, mph) systems
- **Rich Data Points**:
  - Apparent temperature (feels-like temperature)
  - Wind direction with cardinal indicators
  - Atmospheric pressure
  - Visibility distance
  - Solar radiation measurements
  - Precipitation probability
  - Cloud coverage percentage

### Forecasting Capabilities
- **Hourly Forecast**: 24-hour graphical forecast showing:
  - Temperature trend graph
  - Precipitation probability timeline
  - Wind speed and direction visualization
  - Hourly breakdown for detailed planning

- **Daily Forecast**: Tomorrow's weather prediction
  - High/low temperatures
  - Daily summary and conditions
  - Expected precipitation

- **Extended Forecast**: 7-day weather outlook
  - Daily temperature highs and lows
  - Weather conditions summary
  - Precipitation forecasts

### User Interface Features
- **Adaptive Layout**: Responsive design for various screen sizes
- **Dark Mode Support**: Seamless switching between light and dark themes
- **Location Management**: Add multiple weather locations
- **Graphical Visualization**: 
  - Beautiful weather icons
  - Interactive charts for temperature trends
  - Wind direction diagrams
  - Precipitation probability indicators

---

## Technology Stack

### Language & Framework
- **Language**: Python 3.10+
- **GUI Framework**: GTK 4
- **UI Toolkit**: Libadwaita 1.6+
- **Build System**: Meson

### Key Dependencies

| Component | Version | Purpose |
|-----------|---------|---------|
| GTK 4 | ≥4.5 | Core GUI rendering engine |
| Libadwaita | ≥1.6 | GNOME-style widgets and theming |
| PyGObject | Latest | GTK/GObject Python bindings |
| python3-requests | Latest | HTTP client for API calls |
| Meson | ≥0.60 | Build system and build configuration |
| Blueprint | ≥0.16 | UI markup language compiler |

### External Services
- **Open-Meteo API**: Free weather forecast API
  - No authentication required
  - Global coverage with 11km global + 1km mesoscale models
  - Hourly resolution forecasts
  - Up to 16-day forecast range

### Desktop Integration
- **GNOME Platform**: Latest GNOME runtime
- **Freedesktop Standards**: 
  - D-Bus integration
  - XDG base directory specification
  - .desktop files for application discovery
  - AppData metadata for app stores

---

## Architecture Overview

### High-Level Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    Mousam Application                        │
├─────────────────────────────────────────────────────────────┤
│                                                               │
│  ┌────────────────┐  ┌────────────────┐  ┌──────────────┐  │
│  │  GUI Layer     │  │ Business Logic │  │   Settings   │  │
│  │ (Libadwaita)   │  │   & Services   │  │  & Storage   │  │
│  └────────────────┘  └────────────────┘  └──────────────┘  │
│                              │                                 │
│  ┌────────────────┐  ┌───────┴──────┐  ┌──────────────┐  │
│  │ UI Components  │  │ Cache Manager│  │ Local Config │  │
│  │  - Windows     │  │ Weather Data │  │ User Prefs   │  │
│  │  - Dialogs     │  └───────┬──────┘  └──────────────┘  │
│  │  - Widgets     │          │                             │
│  └────────────────┘  ┌───────▼──────┐                    │
│                      │  API Client   │                    │
│                      │ (Open-Meteo)  │                    │
│                      └───────┬──────┘                    │
└──────────────────────────────┼──────────────────────────────┘
                               │
                    ┌──────────▼─────────┐
                    │   External APIs    │
                    │  - Open-Meteo API  │
                    │  - System APIs     │
                    └────────────────────┘
```

### Component Structure

```
mousam/
├── src/
│   └── mousam/
│       ├── __main__.py              # Application entry point
│       ├── app.py                   # Main application class
│       ├── window.py                # Main window implementation
│       ├── ui/                      # UI components
│       │   ├── pages/              # Different views/pages
│       │   ├── widgets/            # Reusable UI components
│       │   └── dialogs/            # Dialog windows
│       ├── services/               # Business logic services
│       │   ├── weather_service.py   # Weather data fetching
│       │   ├── api_client.py        # API communication
│       │   ├── cache.py             # Caching layer
│       │   └── location_service.py  # Location management
│       ├── models/                 # Data models
│       │   ├── weather.py           # Weather data structures
│       │   └── location.py          # Location data structures
│       ├── utils/                  # Utility functions
│       │   ├── constants.py         # Constants and configurations
│       │   ├── formatters.py        # Data formatting utilities
│       │   └── helpers.py           # General helpers
│       └── config/                 # Configuration
│           └── settings.py          # User preferences
├── data/
│   ├── resources.gresource.xml     # Resource definitions
│   ├── ui/                         # Blueprint UI files
│   └── icons/                      # Application icons
├── meson.build                     # Meson build configuration
└── pyproject.toml                  # Python project metadata
```

### Data Flow

```
User Input (Location Selection)
         │
         ▼
┌─────────────────────┐
│ LocationService     │ ◄─── Load saved locations
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│ WeatherService      │
│ - Fetch forecast    │
│ - Process data      │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│ APIClient           │ ───► Open-Meteo API
│ HTTP Requests       │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│ CacheManager        │ ◄─── Store locally
│ - Cache data        │
│ - Validate TTL      │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│ WeatherViewModel    │ ◄─── Format for display
│ - Format values     │
│ - Prepare graphs    │
└──────────┬──────────┘
           │
           ▼
Display UI & Charts
```

---

## Installation Guide

### Supported Platforms

- **Linux (Primary)**
  - Ubuntu 20.04 LTS and later
  - Fedora 33 and later
  - Debian 11 and later
  - Arch Linux and derivatives
  - Any Linux distribution with GTK4 and Libadwaita support

### Installation Methods

#### 1. Flathub (Recommended for Most Users)

The easiest way to install with automatic updates:

```bash
flatpak install flathub io.github.amit9838.mousam
```

**Launch:**
```bash
flatpak run io.github.amit9838.mousam
```

**Benefits:**
- Automatic updates
- Sandbox isolation
- Works across distributions
- No system library conflicts

#### 2. Snap Package

If you prefer Snap:

```bash
sudo snap install mousam
```

**Launch:**
```bash
mousam
```

#### 3. Debian/Ubuntu Package (Unofficial)

Maintained by community contributor @hsbasu:

```bash
# Add repository and install
# (Instructions available at: https://github.com/hsbasu/mousam-deb)
```

#### 4. Native Distribution Packages

Check your distribution's package manager:

- **Ubuntu/Debian**: `apt search mousam`
- **Fedora**: `dnf search mousam`
- **Arch**: `pacman -Ss mousam`

### Uninstallation

**Flatpak:**
```bash
flatpak uninstall io.github.amit9838.mousam
```

**Snap:**
```bash
sudo snap remove mousam
```

---

## Build from Source

### Prerequisites

Before building, ensure you have the required development tools and libraries:

#### Ubuntu/Debian
```bash
sudo apt install -y \
    python3-dev \
    python3-pip \
    python3-requests \
    build-essential \
    meson \
    ninja-build \
    libgtk-4-dev \
    libadwaita-1-dev \
    libglib2.0-dev \
    libgobject-introspection1.0-dev \
    gir1.2-gtk-4.0 \
    gir1.2-adw-1
```

#### Fedora
```bash
sudo dnf install -y \
    python3-devel \
    python3-pip \
    python3-requests \
    gcc \
    meson \
    ninja-build \
    gtk4-devel \
    libadwaita-devel \
    gobject-introspection-devel
```

#### Arch Linux
```bash
sudo pacman -S \
    base-devel \
    python \
    python-requests \
    meson \
    ninja \
    gtk4 \
    libadwaita
```

### Build Steps

#### 1. Clone Repository
```bash
git clone https://github.com/amit9838/mousam.git
cd mousam
```

#### 2. Configure Build
```bash
rm -rf builddir
meson setup -Dprefix=$HOME/.local builddir
```

**Build Configuration Options:**
- `-Dprefix=$HOME/.local`: Install to home directory (no sudo required)
- `-Dprefix=/usr`: Install system-wide (requires sudo for install)
- `-Dbuildtype=debug`: Build with debug symbols
- `-Dbuildtype=release`: Optimized build (default)

#### 3. Compile
```bash
meson compile -C builddir --verbose
```

**Build flags:**
- `--verbose`: Show detailed build output
- `-j4`: Use 4 parallel jobs (adjust based on CPU cores)

#### 4. Install
```bash
# To home directory
meson install -C builddir

# Or system-wide (requires sudo)
sudo meson install -C builddir
```

#### 5. Run Application
```bash
# If installed to $HOME/.local
$HOME/.local/bin/mousam

# Or if in PATH
mousam

# Or run from build directory without installing
./builddir/src/mousam/mousam
```

### Development Build (No Installation)

For rapid development iteration:

```bash
# Setup
meson setup builddir

# Build
meson compile -C builddir

# Run directly from build directory
PYTHONPATH=builddir ./builddir/src/mousam/__main__.py
```

### Clean Build

To perform a complete rebuild:

```bash
rm -rf builddir
meson setup builddir
meson compile -C builddir
meson install -C builddir
```

### Troubleshooting Build Issues

**Error: "gtk4-devel not found"**
```bash
# Update package lists first
sudo apt update  # or dnf update / pacman -Sy
# Then install dependencies
```

**Error: "python3-gobject not installed"**
```bash
# Additional PyGObject bindings
pip3 install PyGObject
```

**Error: "meson: command not found"**
```bash
pip3 install meson ninja
```

**Permission denied during install:**
```bash
# Use --user or setup with $HOME/.local prefix
meson setup -Dprefix=$HOME/.local builddir
```

---

## Configuration

### User Configuration

Configuration files are stored in `~/.config/mousam/` (XDG Base Directory):

```
~/.config/mousam/
├── settings.json          # User preferences
├── locations.json         # Saved locations
└── cache/                 # Weather data cache
    ├── weather_cache.db
    └── forecast_cache.db
```

### Settings Management

#### Temperature Units
- Default: Metric (Celsius)
- Options: Metric (C, m/s), Imperial (F, mph)
- Set via: Application UI → Preferences

#### Location Storage
- Locations are saved automatically after selection
- Format: JSON with latitude, longitude, name
- Accessible via: GUI location management dialog

#### Cache Configuration
- Cache TTL (Time-to-Live): 30 minutes
- Manual refresh: Click refresh button or use keyboard shortcut (Ctrl+R)
- Clear cache: Application menu → Clear Cache

### Advanced Configuration (Developers)

Edit `~/.config/mousam/settings.json`:

```json
{
  "theme": "auto",
  "units": "metric",
  "auto_location": false,
  "update_interval_minutes": 30,
  "cache_enabled": true,
  "api_timeout_seconds": 10
}
```

### Environment Variables

```bash
# Enable debug logging
MOUSAM_DEBUG=1 mousam

# Custom config directory
MOUSAM_CONFIG_DIR=/path/to/config mousam

# Disable cache
MOUSAM_CACHE_ENABLED=0 mousam
```

---

## Usage Guide

### First Launch

1. **Open Application**: Click "Mousam" in applications menu or run `mousam` terminal
2. **Grant Permissions**: If using Flatpak, approve location access if desired
3. **Set Location**: 
   - Allow geolocation (if permitted) OR
   - Manually search and select city
4. **View Weather**: Application displays current conditions and forecasts

### Main Interface

**Header Bar:**
- Application menu (three horizontal lines)
- Refresh button (circular arrows icon)
- Location selector
- Units toggle button

**Main View:**
- Current weather conditions (large temperature display)
- Weather icon and description
- Key metrics (humidity, wind speed, pressure, etc.)
- Multiple tabs/sections for different forecasts

**Sidebar/Navigation:**
- Location management
- Preferences
- About and help

### Common Tasks

#### Change Location
1. Click location name in header bar
2. Search for new city
3. Select from results
4. View instantly updates

#### Switch Temperature Units
1. Click units toggle (°C/°F) in header
2. All values update immediately
3. Preference is saved

#### View Hourly Forecast
1. Navigate to "Hourly" tab
2. Scroll horizontally through 24-hour period
3. View temperature and precipitation graphs
4. Wind direction and speed indicators

#### View Daily/Weekly Forecast
1. Click "Tomorrow" or "7 Days" tab
2. View day-by-day breakdown
3. See high/low temperatures and conditions

#### Refresh Weather Data
- Manual: Click refresh button or press Ctrl+R
- Automatic: Updates every 30 minutes (configurable)

#### Add Multiple Locations
1. Click "+" button in location panel
2. Search and select new city
3. Switch between locations via dropdown

---

## API Integration

### Open-Meteo API Overview

Mousam uses the **Open-Meteo API**, a free, open-source weather API:

**API Endpoint**: https://api.open-meteo.com/v1/

**Key Characteristics:**
- No authentication required (no API key needed)
- Free for non-commercial use
- Global coverage with multiple weather models
- High temporal resolution (hourly data)
- High spatial resolution (1km local models)
- Integrates data from national weather services

### API Endpoints Used

#### Weather Forecast Endpoint

```
GET /forecast
Parameters:
  latitude (float)      - Location latitude
  longitude (float)     - Location longitude
  hourly (string)       - Comma-separated list of variables to fetch
  daily (string)        - Daily aggregates to fetch
  temperature_unit      - celsius or fahrenheit
  timezone              - IANA timezone string
```

**Example Request:**
```bash
curl "https://api.open-meteo.com/v1/forecast?latitude=28.6139&longitude=77.2090&hourly=temperature_2m,precipitation,windspeed_10m&daily=temperature_2m_max,temperature_2m_min&timezone=Asia/Kolkata"
```

### Data Fetching Flow

```python
# Flow in weather_service.py
1. LocationService provides latitude/longitude
2. APIClient constructs request URL
3. CacheManager checks if fresh data exists
4. If cache miss:
   a. APIClient sends HTTP request to Open-Meteo
   b. Parse JSON response
   c. Store in CacheManager with TTL
5. Transform API data to Weather model
6. Return to UI layer for display
```

### Error Handling & Retry Logic

**Automatic Retry Strategy:**
- Network timeout: Retry up to 3 times with exponential backoff
- Rate limiting (429): Wait and retry
- API errors (5xx): Retry with increasing delays
- Invalid input (4xx): Fail immediately with user-friendly error

**Fallback Mechanism:**
- If API fails and cache exists: Display cached data (even if expired)
- Display warning badge if data is stale
- Offer manual refresh option

### API Rate Limiting

Open-Meteo Public API rate limits:
- **Requests per minute**: 60 (typically enough for normal usage)
- **Concurrent connections**: 1 per IP
- **Timeout**: 30 seconds per request

### Caching Strategy

**Cache Levels:**

1. **Memory Cache** (current session):
   - Fastest access
   - Cleared on app exit

2. **Disk Cache** (`~/.config/mousam/cache/`):
   - Survives app restarts
   - TTL: 30 minutes
   - Automatic cleanup of expired entries

3. **Open-Meteo Aggregated Data**:
   - Server-side caching improves response times
   - Multiple requests for same location share cache

---

## Development Guide

### Project Structure Explained

#### Entry Point: `src/mousam/__main__.py`
```python
# Application bootstrap
# - Imports and dependencies check
# - Creates and runs main Gtk.Application
# - Handles command-line arguments
```

#### Main Application: `src/mousam/app.py`
```python
class MouseamApp(Gtk.Application):
    """Core application lifecycle management"""
    - do_startup(): Initialize app resources
    - do_activate(): Create and show main window
    - do_shutdown(): Cleanup resources
    - Signal handlers for system events
```

#### Window: `src/mousam/window.py`
```python
class MouseamWindow(Gtk.ApplicationWindow):
    """Main window UI and layout"""
    - Build header bar and navigation
    - Create content area with pages
    - Connect UI signals to handlers
    - Manage window state (size, position)
```

### Key Services

#### WeatherService (`services/weather_service.py`)
**Responsibilities:**
- Orchestrate data fetching
- Handle location-to-coordinates conversion
- Coordinate with API and cache layers
- Transform raw data to models

**Key Methods:**
```python
async get_weather(location: str) -> Weather
    """Fetch current weather and forecast"""

async refresh_weather(location: Location) -> Weather
    """Force refresh from API"""

async search_locations(query: str) -> List[Location]
    """Search for cities/locations"""
```

#### APIClient (`services/api_client.py`)
**Responsibilities:**
- HTTP communication with Open-Meteo
- URL construction
- Request/response handling
- Error handling and retries

**Key Methods:**
```python
fetch_forecast(lat: float, lon: float, 
               hourly: List[str],
               daily: List[str]) -> dict
    """Fetch raw forecast data from API"""

search_geocoding(query: str) -> List[dict]
    """Search for locations (uses Nominatim)"""
```

#### CacheManager (`services/cache.py`)
**Responsibilities:**
- Local data persistence
- TTL validation
- Cache invalidation
- Performance optimization

**Key Methods:**
```python
get_cached(key: str) -> Optional[Weather]
    """Retrieve cached weather if fresh"""

set_cache(key: str, data: Weather, ttl: int = 1800)
    """Store weather data with TTL"""

clear_expired()
    """Remove expired cache entries"""
```

### Data Models

#### Weather Model (`models/weather.py`)
```python
@dataclass
class Weather:
    """Complete weather information"""
    current: CurrentWeather
    hourly: HourlyForecast
    daily: DailyForecast
    location: Location
    last_updated: datetime
    is_cached: bool
```

#### Location Model (`models/location.py`)
```python
@dataclass
class Location:
    """Geographic location with name"""
    name: str
    latitude: float
    longitude: float
    timezone: str
    country: str
```

### UI Components

#### Widgets Pattern
```python
# Reusable UI components in ui/widgets/
class WeatherCard(Gtk.Box):
    """Display weather metrics"""
    
class HourlyChart(Gtk.DrawingArea):
    """Temperature/precipitation graph"""
    
class LocationPicker(Gtk.Dialog):
    """Search and select locations"""
```

#### Pages Pattern
```python
# Different views in ui/pages/
class CurrentWeatherPage(Gtk.Box):
    """Now tab - current conditions"""
    
class HourlyForecastPage(Gtk.Box):
    """Hourly tab - 24-hour forecast"""
    
class DailyForecastPage(Gtk.Box):
    """7-day tab - extended forecast"""
```

### Signal Handling

**GTK Signal Connection Pattern:**
```python
# In window.py
def on_location_changed(self, combo_box):
    """Handle location selection change"""
    location = combo_box.get_selected_item()
    self.weather_service.fetch_weather(location)

# Connect in __init__:
self.location_combo.connect("changed", self.on_location_changed)
```

### Async Operations

**Threading for Long-Running Tasks:**
```python
import threading
import asyncio

def on_refresh_clicked(self, button):
    """Non-blocking weather refresh"""
    thread = threading.Thread(target=self._fetch_async)
    thread.daemon = True
    thread.start()

def _fetch_async(self):
    """Run in background thread"""
    weather = self.weather_service.fetch_weather(...)
    GLib.idle_add(self._update_ui, weather)
```

### Building and Testing

**Run with Debug Output:**
```bash
PYTHONPATH=src python3 -m mousam
```

**Run Unit Tests:**
```bash
pytest tests/ -v
```

**Run with Debug Logging:**
```bash
MOUSAM_DEBUG=1 python3 -m mousam
```

**Check Code Quality:**
```bash
# Linting
pylint src/mousam/

# Type checking
mypy src/mousam/

# Code formatting
black src/mousam/
```

### Creating a New Feature

**Example: Add new weather metric**

1. **Update Data Model** (`models/weather.py`):
```python
@dataclass
class CurrentWeather:
    # ... existing fields ...
    new_metric: float  # Add new field
```

2. **Update API Client** (`services/api_client.py`):
```python
hourly_variables = [
    # ... existing ...
    "new_metric_2m",  # Add to API request
]
```

3. **Update UI Widget** (`ui/widgets/weather_card.py`):
```python
new_metric_label.set_text(
    f"{weather.current.new_metric} units"
)
```

4. **Test Changes**:
```bash
meson compile -C builddir
./builddir/src/mousam/__main__.py
```

### Debugging Tips

**Print Debug Information:**
```python
import logging
logging.basicConfig(level=logging.DEBUG)
logger = logging.getLogger(__name__)
logger.debug(f"Weather data: {weather}")
```

**Use GDB for Crashes:**
```bash
gdb python3 -ex "run -m mousam"
```

**GTK Inspector (Live UI Debugging):**
```bash
GTK_DEBUG=interactive mousam
```

---

## Contributing

### How to Contribute

Mousam is an open-source project and welcomes contributions from the community!

### Areas for Contribution

1. **Code Features**
   - Bug fixes and improvements
   - New weather metrics or visualizations
   - Performance optimizations
   - Additional forecast types

2. **Localization**
   - Translate UI strings to new languages
   - Improve existing translations

3. **Documentation**
   - Improve code documentation
   - Write tutorials
   - Create video guides

4. **Quality Assurance**
   - Report bugs
   - Test on different systems
   - Suggest UI/UX improvements

5. **Packaging**
   - Maintain distribution packages
   - Create new package formats

### Contributing Code

**1. Fork Repository**
```bash
# On GitHub, click "Fork"
git clone https://github.com/YOUR_USERNAME/mousam.git
cd mousam
```

**2. Create Feature Branch**
```bash
git checkout -b feature/add-new-metric
```

**3. Make Changes**
- Write clean, documented code
- Follow existing code style (PEP 8)
- Add tests for new functionality
- Update documentation

**4. Commit with Clear Messages**
```bash
git commit -m "Add new weather metric: air quality index"
```

**5. Push and Create Pull Request**
```bash
git push origin feature/add-new-metric
# Then create PR on GitHub
```

### Code Style Guidelines

**Python Code Style:**
- Follow PEP 8
- Use type hints: `def fetch(lat: float) -> dict:`
- Maximum line length: 100 characters
- Use meaningful variable names

**GTK/UI Code:**
- Use Libadwaita widgets when available
- Follow GNOME HIG for layouts
- Ensure accessibility (labels, keyboard navigation)
- Test on multiple screen sizes

**Documentation:**
- Add docstrings to all functions
- Include type hints in docstrings
- Provide usage examples for public APIs

### Testing Before Submission

```bash
# Build and run
meson setup builddir
meson compile -C builddir
./builddir/src/mousam/__main__.py

# Check formatting
black src/ --check
pylint src/

# Run existing tests
pytest tests/ -v
```

### Pull Request Process

1. Update documentation if adding features
2. Add/update tests
3. Ensure all tests pass locally
4. Create clear PR description
5. Link any related issues
6. Respond to review feedback
7. Maintainers will merge when approved

---

## Support & Credits

### Getting Help

**Documentation:**
- In-app help: Application Menu → Help
- GitHub Issues: https://github.com/amit9838/mousam/issues
- GitHub Discussions: https://github.com/amit9838/mousam/discussions

**Reporting Bugs:**
1. Search existing issues to avoid duplicates
2. Provide:
   - System information (OS, GTK version)
   - Steps to reproduce
   - Expected vs actual behavior
   - Screenshots if applicable
3. Create detailed issue on GitHub

### Credits & Acknowledgments

**Weather Data:**
- **Open-Meteo**: Free, open-source weather API providing global forecasts
  - Website: https://open-meteo.com
  - Data sources: Multiple national weather services
  - License: Open Data Commons

**UI & Icons:**
- **basmilius**: Beautiful weather icons and weather SVG library
  - Repository: https://github.com/basmilius/weather-icons

**Dependencies:**
- **GNOME Project**: GTK4 and Libadwaita
- **PyGObject**: Python GTK bindings
- **Open-source community**: All other libraries and tools

### Contributors

30+ contributors have helped develop Mousam. See GitHub for full list:
https://github.com/amit9838/mousam/graphs/contributors

### Project Statistics

- **Stars**: 315+
- **Forks**: 45+
- **Contributors**: 30+
- **Active Development**: Yes
- **Last Update**: December 2024

### Support the Project

If you find Mousam useful, consider supporting the developer:

**Donate:**
- Buy Me a Coffee: https://www.buymeacoffee.com/ami9838

**Contribute:**
- Report bugs and suggest features
- Contribute code or translations
- Write documentation
- Help maintain packages

**Share:**
- Recommend to friends
- Review on Flathub/Linux app stores
- Share on social media

---

## License

Mousam is licensed under the **GNU General Public License v3.0 (GPLv3)**

See `LICENSE` file in repository for full license text.

**License Summary:**
- Free to use, modify, and distribute
- Must include license and copyright notice
- Must disclose source code
- Same license applies to modifications
- No warranty provided

---

## Additional Resources

### GNOME Development
- GNOME HIG: https://developer.gnome.org/hig/
- GTK4 Documentation: https://docs.gtk.org/gtk4/
- Libadwaita Docs: https://gnome.pages.gitlab.gnome.org/libadwaita/

### Python GTK Development
- PyGObject Tutorial: https://pygobject.readthedocs.io/
- GTK Python Examples: https://github.com/GNOME/python-gi/tree/master/examples

### Weather APIs
- Open-Meteo Docs: https://open-meteo.com/en/docs
- Weather Data Formats: https://open-meteo.com/en/docs

### Build Systems
- Meson Documentation: https://mesonbuild.com/
- Meson Python Guide: https://mesonbuild.com/meson-python/

---

**Thank you for using Mousam! Enjoy weather at a glance.** ☀️