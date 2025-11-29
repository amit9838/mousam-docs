# Developer Setup & Contribution Guide

## Local Development Environment

### Prerequisites Checklist

- [ ] Python 3.10 or newer
- [ ] Git installed
- [ ] GTK 4.5+ development libraries
- [ ] Libadwaita 1.6+ development libraries
- [ ] Meson build system
- [ ] Text editor or IDE with Python support

---

## Step-by-Step Development Setup

### 1. Clone Repository

```bash
# HTTPS (recommended for public repos)
git clone https://github.com/amit9838/mousam.git
cd mousam

# Or SSH (if you have SSH key configured)
git clone git@github.com:amit9838/mousam.git
cd mousam
```

### 2. Install System Dependencies

#### Ubuntu/Debian
```bash
# Update package lists
sudo apt update

# Install development libraries
sudo apt install -y \
    build-essential \
    python3 \
    python3-dev \
    python3-pip \
    python3-gi \
    python3-gi-cairo \
    gir1.2-gtk-4.0 \
    gir1.2-adw-1 \
    libgtk-4-dev \
    libadwaita-1-dev \
    libglib2.0-dev \
    libgobject-introspection1.0-dev \
    meson \
    ninja-build \
    git

# Additional runtime dependencies
sudo apt install -y \
    python3-requests \
    fonts-liberation
```

#### Fedora
```bash
sudo dnf install -y \
    gcc \
    python3-devel \
    python3-pip \
    python3-gobject \
    gtk4-devel \
    libadwaita-devel \
    gobject-introspection-devel \
    meson \
    ninja-build \
    git

# Additional runtime dependencies
sudo dnf install -y \
    python3-requests
```

#### Arch Linux
```bash
sudo pacman -S \
    base-devel \
    python \
    python-pip \
    gobject-introspection \
    gtk4 \
    libadwaita \
    meson \
    ninja \
    git

# Additional packages
sudo pacman -S \
    python-requests
```

#### macOS (using Homebrew)
```bash
# Install Homebrew if not already installed
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# Install dependencies
brew install \
    python@3.11 \
    gtk4 \
    libadwaita \
    meson \
    ninja \
    git

# Install Python packages
pip3 install -r requirements-dev.txt
```

### 3. Install Python Development Dependencies

```bash
# Navigate to project directory
cd mousam

# Create virtual environment (recommended)
python3 -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate

# Install development dependencies
pip install -r requirements-dev.txt
```

**requirements-dev.txt** should contain:
```
requests>=2.28.0        # HTTP client for API
PyGObject>=3.44.0       # GTK Python bindings
pytest>=7.0             # Testing framework
pytest-cov>=4.0         # Test coverage
black>=23.0             # Code formatter
pylint>=2.16.0          # Linter
mypy>=1.0               # Type checker
```

### 4. Build the Project

```bash
# Clean previous builds (if any)
rm -rf builddir

# Configure build
meson setup builddir

# Build
meson compile -C builddir

# Run directly without installing
python3 -m mousam
# Or
PYTHONPATH=builddir python3 src/mousam/__main__.py
```

### 5. Install Locally (Optional)

```bash
# Install to home directory (~/.local)
meson install -C builddir --skip-subprojects

# Add to PATH if not already
export PATH="$PATH:$HOME/.local/bin"

# Now run
mousam
```

---

## Project Structure for Developers

```
mousam/
├── src/
│   └── mousam/
│       ├── __main__.py              # Entry point
│       ├── __init__.py              # Package init
│       ├── app.py                   # Main application
│       ├── window.py                # Main window
│       ├── ui/
│       │   ├── pages/              # Different views
│       │   │   ├── current.py
│       │   │   ├── hourly.py
│       │   │   └── daily.py
│       │   ├── widgets/            # Reusable components
│       │   │   ├── weather_card.py
│       │   │   ├── charts.py
│       │   │   └── location_picker.py
│       │   └── dialogs/            # Dialog windows
│       │       └── settings.py
│       ├── services/               # Business logic
│       │   ├── weather_service.py   # Weather fetching
│       │   ├── api_client.py        # API communication
│       │   ├── cache.py             # Caching
│       │   ├── location_service.py  # Location management
│       │   └── preferences.py       # Settings management
│       ├── models/                 # Data models
│       │   ├── weather.py
│       │   └── location.py
│       └── utils/                  # Utilities
│           ├── constants.py
│           ├── formatters.py
│           ├── helpers.py
│           └── icons.py
├── data/
│   ├── resources.gresource.xml     # Resource file
│   ├── ui/                         # UI blueprint files
│   │   ├── window.blp
│   │   └── pages/
│   ├── icons/                      # App icons
│   │   ├── mousam.svg
│   │   └── weather-icons/
│   └── styles/
│       └── custom.css
├── tests/
│   ├── __init__.py
│   ├── test_weather_service.py
│   ├── test_api_client.py
│   ├── test_cache.py
│   └── test_ui.py
├── build-aux/
│   └── flatpak/
│       └── io.github.amit9838.mousam.yml
├── meson.build                     # Build configuration
├── meson_options.txt              # Build options
├── pyproject.toml                 # Python project metadata
├── requirements.txt               # Runtime dependencies
├── requirements-dev.txt           # Development dependencies
├── README.md                      # Main documentation
├── ARCHITECTURE.md                # Architecture docs
├── API_INTEGRATION.md             # API documentation
└── LICENSE                        # GPLv3 license
```

---

## Common Development Tasks

### Running the Application

```bash
# From project directory with build
./builddir/src/mousam/__main__.py

# Or after installing
mousam

# With debug output
MOUSAM_DEBUG=1 mousam

# With Python debugger
python3 -m pdb src/mousam/__main__.py
```

### Making Code Changes

#### Example: Add a new weather metric

**1. Update data model** (`models/weather.py`):
```python
@dataclass
class CurrentWeather:
    temperature: float
    humidity: int
    # Add new field:
    air_quality_index: int  # New metric
```

**2. Update API client** (`services/api_client.py`):
```python
HOURLY_VARIABLES = [
    "temperature_2m",
    "relative_humidity_2m",
    # Add to fetch:
    "air_quality",  # WMO Air Quality Index
]
```

**3. Update UI widget** (`ui/widgets/weather_card.py`):
```python
class WeatherCard(Gtk.Box):
    def __init__(self):
        # ... existing code ...
        self.aqi_label = Gtk.Label()
        self.append(self.aqi_label)
    
    def set_weather(self, weather):
        # ... existing code ...
        self.aqi_label.set_text(f"AQI: {weather.air_quality_index}")
```

**4. Rebuild and test**:
```bash
meson compile -C builddir
./builddir/src/mousam/__main__.py
```

### Running Tests

```bash
# Run all tests
pytest tests/ -v

# Run specific test file
pytest tests/test_weather_service.py -v

# Run specific test
pytest tests/test_weather_service.py::TestWeatherService::test_fetch -v

# Run with coverage report
pytest tests/ --cov=src/mousam --cov-report=html

# View coverage report
open htmlcov/index.html
```

### Code Quality Checks

```bash
# Format code with Black
black src/ tests/

# Check formatting without changing
black src/ tests/ --check

# Lint code
pylint src/mousam

# Type checking
mypy src/mousam

# All checks together
black --check src/ && pylint src/ && mypy src/
```

### Git Workflow

```bash
# Check status
git status

# Stage changes
git add src/mousam/new_feature.py

# Create commit
git commit -m "Add new weather metric: air quality index"

# Push to fork
git push origin feature/air-quality

# Create Pull Request on GitHub
# (Instructions appear in terminal)
```

---

## Debug Techniques

### Enable Debug Logging

```python
# In any module
import logging

logger = logging.getLogger(__name__)

# In your code:
logger.debug(f"Weather data: {weather}")
logger.info(f"Fetching forecast for {location.name}")
logger.warning(f"API response slow: {response_time}s")
logger.error(f"Failed to parse response: {e}")
```

**Run with debug output:**
```bash
MOUSAM_DEBUG=1 PYTHONUNBUFFERED=1 python3 -m mousam 2>&1 | tee debug.log
```

### GTK Inspector (Live UI Debugging)

```bash
# Launch app with GTK inspector enabled
GTK_DEBUG=interactive mousam

# In the inspector window:
# - Click "+" icon to select widgets
# - Right-click any widget to inspect properties
# - Modify CSS in real-time
# - Check signal connections
```

### Python Debugger

```python
# Add breakpoint in code:
import pdb; pdb.set_trace()

# Or Python 3.7+:
breakpoint()

# When execution stops:
# (Pdb) n              # Next line
# (Pdb) s              # Step into function
# (Pdb) c              # Continue execution
# (Pdb) p variable     # Print variable
# (Pdb) l              # List code
# (Pdb) h              # Help
```

### Memory Profiling

```bash
# Install memory profiler
pip install memory-profiler

# Run with profiling
python3 -m memory_profiler src/mousam/__main__.py
```

### Network Debugging

```bash
# Monitor network requests
python3 -c "
import logging
logging.basicConfig(level=logging.DEBUG)
logging.getLogger('urllib3').setLevel(logging.DEBUG)
" && python3 -m mousam

# Or use tcpdump to capture packets
sudo tcpdump -i lo -A 'port 443'
```

---

## Creating & Running Tests

### Writing Unit Tests

```python
# tests/test_weather_service.py

import unittest
from unittest.mock import Mock, patch
from mousam.services.weather_service import WeatherService
from mousam.models.weather import CurrentWeather

class TestWeatherService(unittest.TestCase):
    def setUp(self):
        """Set up test fixtures"""
        self.mock_api = Mock()
        self.mock_cache = Mock()
        self.service = WeatherService(
            api_client=self.mock_api,
            cache=self.mock_cache
        )
    
    def test_fetch_weather_returns_valid_model(self):
        """Test that fetch returns valid weather model"""
        self.mock_cache.get.return_value = None
        self.mock_api.fetch_forecast.return_value = {
            "hourly": {
                "temperature_2m": [25.5],
                "relative_humidity_2m": [60],
            },
            "daily": {}
        }
        
        weather = self.service.get_weather(
            location=Mock(latitude=28.6, longitude=77.2)
        )
        
        assert weather is not None
        assert weather.current.temperature == 25.5
    
    def test_cache_hit_returns_cached_data(self):
        """Test that cached data is returned"""
        cached_weather = Mock()
        self.mock_cache.get.return_value = cached_weather
        
        result = self.service.get_weather(Mock())
        
        # API should not be called
        self.mock_api.fetch_forecast.assert_not_called()
```

### Running Test Suite

```bash
# Run all tests
pytest

# Run with verbose output
pytest -v

# Run specific test class
pytest tests/test_weather_service.py::TestWeatherService -v

# Run tests matching pattern
pytest -k "cache" -v

# Stop on first failure
pytest -x

# Show local variables on failure
pytest -l

# Generate coverage report
pytest --cov=src/mousam --cov-report=term-missing
```

---

## Build Variants

### Debug Build

```bash
# Build with debug symbols and no optimization
meson setup -Dbuildtype=debug -Dstrip=false builddir-debug
meson compile -C builddir-debug

# Use for debugging
gdb -ex run --args ./builddir-debug/src/mousam/__main__.py
```

### Release Build

```bash
# Optimized build for production
meson setup -Dbuildtype=release -Dstrip=true builddir-release
meson compile -C builddir-release
meson install -C builddir-release
```

### Development Build (Recommended)

```bash
# Fast build with reasonable optimization
meson setup builddir
meson compile -C builddir

# Run without installing
PYTHONPATH=builddir python3 -m mousam
```

---

## Submitting Changes

### Pre-Submission Checklist

- [ ] Code follows PEP 8 style guide
- [ ] All tests pass: `pytest tests/ -v`
- [ ] Code formatted: `black src/`
- [ ] No linting errors: `pylint src/`
- [ ] Type hints added: `mypy src/`
- [ ] Documentation updated
- [ ] Feature tested manually
- [ ] Commit messages are clear

### Pull Request Process

1. **Fork the repository** on GitHub

2. **Create feature branch**:
```bash
git checkout -b feature/my-new-feature
```

3. **Make changes** and commit:
```bash
git commit -am "Add new feature: description"
```

4. **Push to fork**:
```bash
git push origin feature/my-new-feature
```

5. **Open Pull Request** on GitHub with:
   - Clear title
   - Description of changes
   - Link to related issues
   - Screenshots if UI changes

6. **Address review feedback**:
```bash
git add -A
git commit -am "Address review feedback"
git push
```

7. **Maintainers merge** when approved

---

## Useful Commands Reference

```bash
# Build and compile
meson setup builddir
meson compile -C builddir

# Run application
./builddir/src/mousam/__main__.py

# Install locally
meson install -C builddir

# Clean build
rm -rf builddir

# Run tests
pytest tests/ -v

# Code quality
black src/
pylint src/
mypy src/

# Git operations
git status
git add .
git commit -m "message"
git push origin branch-name

# Debug
GTK_DEBUG=interactive mousam
MOUSAM_DEBUG=1 mousam
gdb ./builddir/src/mousam/__main__.py
```

---

## Troubleshooting Development Issues

### Build Fails: "gtk4-devel not found"

```bash
# Solution: Install GTK4 development files
sudo apt install libgtk-4-dev            # Debian
sudo dnf install gtk4-devel              # Fedora
sudo pacman -S gtk4                      # Arch
```

### Import Error: "No module named 'gi'"

```bash
# Solution: Install PyGObject
pip install PyGObject

# Or via package manager
sudo apt install python3-gi               # Debian
sudo dnf install python3-gobject          # Fedora
```

### Meson not found in PATH

```bash
# Solution: Install Meson
pip install meson ninja

# Verify installation
meson --version
```

### GTK Library Not Found at Runtime

```bash
# Solution: Set library path
export LD_LIBRARY_PATH=/usr/lib/x86_64-linux-gnu:$LD_LIBRARY_PATH
python3 -m mousam
```

### App crashes with "segment fault"

```bash
# Debug with GDB
gdb python3
(gdb) run -m mousam

# Or use catchsegv
catchsegv python3 -m mousam
```

---

## Additional Resources

### Documentation
- [GNOME Platform Documentation](https://developer.gnome.org/)
- [GTK4 API Reference](https://docs.gtk.org/gtk4/)
- [Libadwaita Documentation](https://gnome.pages.gitlab.gnome.org/libadwaita/)
- [PyGObject Tutorial](https://pygobject.readthedocs.io/)

### Tools
- [Meson Documentation](https://mesonbuild.com/)
- [Blueprint Language](https://jwestman.pages.gitlab.gnome.org/blueprint-compiler/)
- [GTK Inspector](https://wiki.gnome.org/Projects/GTK/Inspector)
- [Accerciser (Accessibility)](https://wiki.gnome.org/Projects/Accerciser)

### Testing
- [Pytest Documentation](https://docs.pytest.org/)
- [Mock Library](https://docs.python.org/3/library/unittest.mock.html)

### Code Quality
- [PEP 8 Style Guide](https://www.python.org/dev/peps/pep-0008/)
- [Black Formatter](https://black.readthedocs.io/)
- [Pylint Documentation](https://pylint.readthedocs.io/)
- [MyPy Type Checking](https://mypy.readthedocs.io/)

---

**Happy coding! Welcome to the Mousam development community! 🌤️**