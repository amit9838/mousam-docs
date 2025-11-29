# Quick Start Guide

## For End Users

### Installation (5 minutes)

**Fastest Method - Flathub:**
```bash
flatpak install flathub io.github.amit9838.mousam
flatpak run io.github.amit9838.mousam
```

**Alternative - Snap:**
```bash
sudo snap install mousam
mousam
```

**Alternative - From Package Manager:**
```bash
# Ubuntu/Debian
sudo apt install mousam

# Fedora
sudo dnf install mousam

# Arch
sudo pacman -S mousam
```

### First Run

1. **Open Mousam** from applications menu or run `mousam`
2. **Allow location access** (optional - for geolocation)
3. **Search your city** or use detected location
4. **View current weather** - displays instantly if cached, or fetches within 2 seconds
5. **Explore tabs**:
   - **Now**: Current conditions
   - **Hourly**: 24-hour forecast with graphs
   - **Tomorrow**: Next day weather
   - **7 Days**: Weekly forecast

### Basic Usage

| Action | How |
|--------|-----|
| **Change location** | Click location name in header → Search → Select |
| **Switch units** | Click °C/°F button in header |
| **Refresh weather** | Click refresh icon or press Ctrl+R |
| **View 24h forecast** | Click "Hourly" tab, scroll horizontally |
| **View settings** | Click menu button (☰) → Preferences |

### Tips & Tricks

```
• Your location is saved automatically
• Weather updates automatically every 30 minutes
• Click any weather icon for description details
• Hover over charts for precise hourly values
• Dark mode follows your system preferences
```

---

## For Developers

### Quick Development Setup (10 minutes)

```bash
# 1. Clone repository
git clone https://github.com/amit9838/mousam.git
cd mousam

# 2. Install dependencies (Ubuntu/Debian example)
sudo apt install -y libgtk-4-dev libadwaita-1-dev python3-dev meson ninja-build

# 3. Build
meson setup builddir
meson compile -C builddir

# 4. Run
./builddir/src/mousam/__main__.py

# 5. Or after installing
meson install -C builddir
mousam
```

### Project Structure Quick Tour

```
mousam/
├── src/mousam/
│   ├── app.py              ← Main application
│   ├── window.py           ← Main window UI
│   ├── services/           ← Business logic
│   │   ├── weather_service.py    (fetches weather)
│   │   ├── api_client.py         (API calls)
│   │   └── cache.py              (caching)
│   ├── ui/                 ← User interface
│   │   ├── pages/          (different views)
│   │   └── widgets/        (reusable components)
│   ├── models/             ← Data models
│   └── utils/              ← Helper utilities
├── data/
│   ├── resources.gresource.xml   (resources)
│   └── ui/                       (UI files)
└── meson.build             ← Build config
```

### Making Your First Change

**Example: Change app title**

1. **Find the title**:
```bash
grep -r "Mousam" src/
```

2. **Edit in window.py or app.py**:
```python
# Before:
self.set_title("Mousam")

# After:
self.set_title("My Weather App")
```

3. **Rebuild & test**:
```bash
meson compile -C builddir
./builddir/src/mousam/__main__.py
```

### Adding New Weather Metric

**Step-by-step example: Add "feels like" temperature**

**1. Data Model** (`models/weather.py`):
```python
@dataclass
class CurrentWeather:
    temperature: float
    apparent_temperature: float  # Add this
```

**2. API** (`services/api_client.py`):
```python
HOURLY_VARIABLES = [
    "temperature_2m",
    "apparent_temperature",  # Add this
]
```

**3. UI** (`ui/widgets/weather_card.py`):
```python
self.feels_like_label = Gtk.Label()
self.feels_like_label.set_text(
    f"Feels like: {weather.apparent_temperature}°C"
)
```

**4. Test**:
```bash
meson compile -C builddir
./builddir/src/mousam/__main__.py
```

### Debugging Tips

```bash
# Run with debug output
MOUSAM_DEBUG=1 mousam

# Run in GDB (with breakpoint on crash)
gdb python3 -ex "run -m mousam"

# Inspect GTK UI live
GTK_DEBUG=interactive mousam

# Check what's happening
strace -e openat mousam 2>&1 | grep config
```

### Running Tests

```bash
# Install test dependencies
pip install pytest pytest-cov

# Run all tests
pytest tests/ -v

# Run with coverage
pytest --cov=src/mousam tests/

# Run specific test
pytest tests/test_weather_service.py::TestWeatherService::test_fetch -v
```

---

## Architecture Quick Reference

### How Weather Gets to Your Screen

```
1. You click "Delhi"
   ↓
2. LocationService finds coordinates (28.6139, 77.2090)
   ↓
3. WeatherService checks cache (has fresh data?)
   ├─ YES → Return immediately (50ms)
   └─ NO ↓
4. APIClient sends HTTP request to api.open-meteo.com
   ↓
5. Server returns JSON with weather data
   ↓
6. Data transformed to Python objects (models)
   ↓
7. Cache stores data for next 30 minutes
   ↓
8. UI updated with temperature, charts, forecast
```

### Key Components

| Component | Does What | Location |
|-----------|-----------|----------|
| **app.py** | App lifecycle | src/mousam/app.py |
| **window.py** | Main window | src/mousam/window.py |
| **weather_service.py** | Fetch/cache weather | services/ |
| **api_client.py** | HTTP requests | services/ |
| **cache.py** | Local storage | services/ |
| **Weather model** | Data structure | models/ |
| **UI widgets** | Display elements | ui/widgets/ |

### Key APIs

**Open-Meteo Forecast API:**
```
GET https://api.open-meteo.com/v1/forecast
  ?latitude=28.6&longitude=77.2
  &hourly=temperature_2m,precipitation
  &daily=temperature_2m_max,temperature_2m_min
```

**Returns:**
```json
{
  "hourly": {
    "time": ["2025-01-01T00:00", ...],
    "temperature_2m": [25.5, 24.3, ...]
  },
  "daily": {
    "temperature_2m_max": [32, 31, ...],
    "temperature_2m_min": [18, 17, ...]
  }
}
```

---

## Common Tasks Cheat Sheet

### Building

```bash
# Fresh build
rm -rf builddir
meson setup builddir
meson compile -C builddir

# Quick rebuild (after code changes)
meson compile -C builddir

# Install after build
meson install -C builddir

# Run from build dir
./builddir/src/mousam/__main__.py
```

### Testing & Quality

```bash
# Format code
black src/

# Check formatting
black --check src/

# Lint
pylint src/mousam

# Type check
mypy src/mousam

# All checks
black --check src/ && pylint src/ && mypy src/
```

### Git Workflow

```bash
# Create feature branch
git checkout -b feature/my-feature

# Make changes and commit
git add src/mousam/file.py
git commit -m "Add new feature description"

# Push to fork
git push origin feature/my-feature

# Create Pull Request on GitHub
```

### Debugging

```bash
# Print variable in code
logger.debug(f"Weather: {weather}")

# Add breakpoint
breakpoint()

# Run with debug output
MOUSAM_DEBUG=1 python3 -m mousam 2>&1 | tee debug.log

# Inspect live UI
GTK_DEBUG=interactive mousam
```

---

## Useful Resources

### For Users
- **Report bugs**: https://github.com/amit9838/mousam/issues
- **Ask questions**: https://github.com/amit9838/mousam/discussions
- **Documentation**: README.md in repository

### For Developers
- **GNOME HIG**: https://developer.gnome.org/hig
- **GTK4 Docs**: https://docs.gtk.org/gtk4
- **Libadwaita**: https://gnome.pages.gitlab.gnome.org/libadwaita
- **Open-Meteo API**: https://open-meteo.com/en/docs
- **Meson Docs**: https://mesonbuild.com
- **Python PyGObject**: https://pygobject.readthedocs.io

### For Contributors
- **Contribution Guide**: DEVELOPER.md
- **Architecture Docs**: ARCHITECTURE.md
- **API Docs**: API_INTEGRATION.md

---

## Troubleshooting Quick Fixes

### "Module not found" errors
```bash
pip install -r requirements-dev.txt
```

### GTK4 not found
```bash
# Ubuntu
sudo apt install libgtk-4-dev

# Fedora
sudo dnf install gtk4-devel

# Arch
sudo pacman -S gtk4
```

### Meson not found
```bash
pip install meson ninja
```

### Port/permission issues
```bash
# Run in different location
cd /tmp && mousam

# Or rebuild
rm -rf builddir && meson setup builddir
```

### Memory/performance issues
```bash
# Clear cache
rm -rf ~/.config/mousam/cache

# Restart app
pkill mousam
mousam
```

---

## Next Steps

### If You're a User
1. ✅ Install Mousam from Flathub
2. ✅ Add your favorite locations
3. ✅ Explore the UI and features
4. ✅ Report any issues or feature requests
5. ✅ Recommend to friends!

### If You're a Developer
1. ✅ Clone the repository
2. ✅ Build and run locally
3. ✅ Read ARCHITECTURE.md for system overview
4. ✅ Make a small change to understand the flow
5. ✅ Pick an issue to work on
6. ✅ Submit a Pull Request!

---

## Getting Help

### Documentation
- **README.md** - Overview and installation
- **ARCHITECTURE.md** - System design and components
- **API_INTEGRATION.md** - Open-Meteo API details
- **DEVELOPER.md** - Development setup guide
- **INTEGRATION.md** - Complete system overview

### Community
- **GitHub Issues**: Report bugs or request features
- **GitHub Discussions**: Ask questions
- **GitHub Pull Requests**: Submit improvements

### Quick Links
- Repository: https://github.com/amit9838/mousam
- Issue Tracker: https://github.com/amit9838/mousam/issues
- Discussions: https://github.com/amit9838/mousam/discussions
- Flathub: https://flathub.org/en/apps/io.github.amit9838.mousam

---

## Glossary of Terms

| Term | Meaning |
|------|---------|
| **GTK** | GUI toolkit for building interfaces |
| **Libadwaita** | GNOME UI library built on GTK4 |
| **Meson** | Build system for compiling code |
| **API** | Interface to fetch weather data |
| **Cache** | Local storage for quick access |
| **TTL** | Time-to-Live (how long data stays valid) |
| **GLib** | Core library used by GTK |
| **PyGObject** | Python bindings for GTK |
| **Flatpak** | Containerized app distribution |
| **Open-Meteo** | Free weather API provider |

---

## Keyboard Shortcuts

| Shortcut | Action |
|----------|--------|
| **Ctrl+Q** | Quit application |
| **Ctrl+R** | Refresh weather |
| **Ctrl+,** | Open preferences |
| **Ctrl+H** | Show help |
| **Ctrl+Shift+?** | Keyboard shortcuts |
| **Alt+Tab** | Switch apps |
| **Tab** | Navigate UI elements |
| **Enter** | Activate button/search |
| **Escape** | Close dialog/clear search |

---

## Tips for Smooth Experience

```
💡 Keep the app running for instant updates
💡 Enable auto-refresh for current conditions
💡 Bookmark your frequent locations
💡 Check hourly forecast for activity planning
💡 Use dark mode at night to reduce eye strain
💡 Allow geolocation for automatic location
💡 Check weather before outdoor activities
💡 Report issues to help improve the app
```

---

**Happy weather checking! 🌤️**

*Last updated: November 2024*
*Mousam v1.4.0 and later*