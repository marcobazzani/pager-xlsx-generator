# PagerDuty-Style On-Call Shift Scheduler

Generate professional on-call shift schedules with automated rotation management. Create Excel spreadsheets, visual PNG charts, and iCalendar files for your team's on-call coverage.

## Key Features

- **📊 Excel Output** - Detailed schedule with dates, times, assignments, and on-call status formulas
- **📈 Visual Charts** - Color-coded PNG timeline showing the complete schedule
- **📅 iCalendar Export** - Generate .ics files for each team member to import into their calendar
- **⚙️ Flexible Configuration** - YAML-based setup for multiple layers and time windows
- **🔄 Smart Rotation** - Automatic daily rotation with per-layer offsets
- **📆 Date Ranges** - Configurable periods (default: 3 months from today)
- **⏰ Day-Specific Times** - Different time windows for different days (e.g., shorter Friday shifts)
- **👻 Dummy Shifts** - Optional shifts that consume rotation but aren't visible (for fairness)

## Quick Start

### Prerequisites

- Python 3.7 or higher
- pip (Python package manager)

### Installation

#### macOS / Linux

```bash
# Clone the repository
git clone https://github.com/marcobazzani/pager-xlsx-generator.git
cd pager-xlsx-generator

# Create virtual environment
python3 -m venv venv
source venv/bin/activate

# Install dependencies
pip install -r requirements.txt
```

#### Windows

```cmd
# Clone the repository
git clone https://github.com/marcobazzani/pager-xlsx-generator.git
cd pager-xlsx-generator

# Create virtual environment
python -m venv venv
venv\Scripts\activate

# Install dependencies
pip install -r requirements.txt
```

### Basic Usage

```bash
# Generate schedule with default settings (today + 3 months)
python oncall_scheduler.py --config your_config.yaml

# Generate with iCalendar files for each team member
python oncall_scheduler.py --config your_config.yaml --generate-ics

# Using absolute dates
python oncall_scheduler.py --config your_config.yaml \
  --start-date 2026-01-20 --end-date 2026-04-20

# Using relative dates (much easier!)
python oncall_scheduler.py --config your_config.yaml \
  --start-date today --end-date +2w

# Mix absolute and relative
python oncall_scheduler.py --config your_config.yaml \
  --start-date 2026-01-20 --end-date +3m
```

**Note:** On Windows, replace `\` line continuation with `^` or write the command on one line.

### Output Files

All outputs are automatically organized in a folder named after your configuration file:

```
your_config/
├── your_config.xlsx          # Excel spreadsheet with formulas
├── your_config.png          # Visual timeline chart
└── ics_files/               # (if --generate-ics used)
    ├── Team_Member_01.ics
    ├── Team_Member_02.ics
    └── ...
```

**Example:** Using `layers_2_shifts_4users.yaml` creates:
- Folder: `layers_2_shifts_4users/`
- Excel: `layers_2_shifts_4users/layers_2_shifts_4users.xlsx`
- PNG: `layers_2_shifts_4users/layers_2_shifts_4users.png`
- ICS: `layers_2_shifts_4users/ics_files/`

### Date Formats

The tool supports two date formats to make scheduling easier:

#### Absolute Dates (Traditional)
Use the format `YYYY-MM-DD`:
- `2026-01-20` - January 20, 2026
- `2026-12-31` - December 31, 2026

#### Relative Dates (Recommended)
Use relative notation for easier date calculations:
- `today` - Today's date
- `+14` or `+14d` - 14 days from reference date
- `+2w` - 2 weeks from reference date
- `+3m` - 3 months from reference date
- `+1y` - 1 year from reference date

**Reference Date:**
- For `--start-date`: relative to today
- For `--end-date`: relative to start date (if provided) or today

**Examples:**
```bash
# Next 2 weeks starting today
--start-date today --end-date +2w

# Start in 1 week, run for 3 months
--start-date +1w --end-date +3m

# Start next Monday (calculate manually), run for 2 weeks
--start-date 2026-01-26 --end-date +2w

# Start today, run for 90 days
--start-date today --end-date +90d
```
```

## Configuration

Create a YAML configuration file to define your schedule. Two example configurations are included:

### Simple 2-Layer Configuration

For basic coverage with fewer team members:

```yaml
schedule:
  name: "My On-Call Schedule"
  description: "Two 5-hour shifts"
  start_date: "2026-01-20"
  duration_months: 3
  
  layers:
    layer1:
      name: "Morning Shift"
      time_windows:
        monday:
          start: "08:00"
          end: "13:00"
        tuesday:
          start: "08:00"
          end: "13:00"
        # ... other days
      rotation_team:
        - "Team Member 1"
        - "Team Member 2"
        - "Team Member 3"
        - "Team Member 4"
    
    layer2:
      name: "Afternoon Shift"
      time_windows:
        monday:
          start: "13:00"
          end: "18:00"
        # ... other days
        friday:
          start: "13:00"
          end: "18:00"
          dummy: true  # Consumes rotation but not shown
      rotation_team:
        - "Team Member 4"
        - "Team Member 3"
        - "Team Member 2"
        - "Team Member 1"
```

### Advanced Multi-Layer Configuration

For complex coverage with multiple time slots:

```yaml
schedule:
  name: "Business Hours Coverage"
  description: "Multiple shifts per day"
  start_date: "2026-01-20"
  duration_months: 3
  
  layers:
    layer1:
      name: "Morning Early"
      time_windows:
        monday:
          start: "08:00"
          end: "10:30"
        # Different times per day supported
        friday:
          start: "08:00"
          end: "09:30"
      rotation_team:
        - "User 1"
        - "User 2"
        # ... more users
```

### Configuration Options

- **schedule.name**: Display name for the schedule
- **schedule.description**: Description text
- **schedule.start_date**: Default start date (YYYY-MM-DD)
- **schedule.duration_months**: Default duration in months
- **layers**: Dictionary of layer configurations
  - **name**: Layer display name
  - **time_windows**: Per-day time configuration
    - **dayname.start**: Start time (HH:MM)
    - **dayname.end**: End time (HH:MM)
    - **dayname.dummy**: Optional, if true shift consumes rotation but isn't shown
  - **rotation_team**: List of team member names (rotates daily)

### Rotation Logic

- Each layer rotates through its `rotation_team` list **daily**
- Different layers can have different or shifted team orders for fair distribution
- Dummy shifts count for rotation but don't appear in outputs (useful for balance)

## Excel Output Details

The generated Excel file includes:

- **Date**: Calendar date for the shift
- **Day**: Day of week (Monday, Tuesday, etc.)
- **Start Time**: Shift start time
- **End Time**: Shift end time
- **Hours**: Calculated duration (End - Start)
- **On-Call Person**: Team member assigned
- **On-Call Status**: Formula that shows "On-Call" when NOW() is within the shift time
- **Empty rows**: Between different dates for readability
- **Color coding**: Each person has a unique color
- **Real-time status**: Excel formula highlights current on-call shift

## Visual Chart Details

The PNG timeline shows:

- **Horizontal axis**: Calendar dates with day names
- **Vertical axis**: Time of day (dynamically scaled to your shift times)
- **Color bars**: Each person has a unique color
- **Text labels**: Person names on each shift bar
- **Legend**: All team members with their colors
- **Full schedule**: Shows all days in your date range

## iCalendar Export

Use `--generate-ics` to create calendar files:

```bash
python oncall_scheduler.py --config your_config.yaml --generate-ics
```

This creates one `.ics` file per team member in the `ics_files/` folder:
- Compatible with Google Calendar, Outlook, Apple Calendar, etc.
- Each file contains all shifts for that person
- Includes 15-minute reminder alarms
- Can be imported or subscribed to

## Command-Line Options

```
--config PATH           YAML configuration file (required)
--start-date DATE       Start date: YYYY-MM-DD, relative (+2w, +3m), or "today"
--end-date DATE         End date: YYYY-MM-DD or relative from start date
--generate-ics          Generate iCalendar files for each team member
```

**Date Format Details:**
- **Absolute**: `2026-01-20`, `2026-12-31`
- **Relative**: `today`, `+7d`, `+2w`, `+3m`, `+1y`
- **Units**: `d` (days), `w` (weeks), `m` (months), `y` (years)
- **Reference**: `--start-date` is relative to today, `--end-date` is relative to start date

**Output Files:**
- Automatically generated from config filename
- Example: `my_schedule.yaml` → `my_schedule/my_schedule.xlsx`
- All outputs go in folder named after config file

## Use Cases

### Quick Preview (Relative Dates)

```bash
# Next 2 weeks
python oncall_scheduler.py --config my_config.yaml \
  --start-date today --end-date +2w

# This week
python oncall_scheduler.py --config my_config.yaml \
  --start-date today --end-date +7d

# Next 30 days
python oncall_scheduler.py --config my_config.yaml \
  --start-date today --end-date +30d
```

### Quarterly Planning (Absolute Dates)

```bash
# Q1 schedule
python oncall_scheduler.py --config my_config.yaml \
  --start-date 2026-01-01 --end-date 2026-04-01

# Q2 schedule
python oncall_scheduler.py --config my_config.yaml \
  --start-date 2026-04-01 --end-date 2026-07-01
```

### Quarterly Planning (Relative Dates)

```bash
# Next quarter (3 months)
python oncall_scheduler.py --config my_config.yaml \
  --start-date today --end-date +3m

# Start next month, run for 3 months
python oncall_scheduler.py --config my_config.yaml \
  --start-date +1m --end-date +3m
```

### Monthly Schedules

```bash
# Q1 schedule
python oncall_scheduler.py --config my_config.yaml \
  --start-date 2026-01-01 --end-date 2026-04-01

# Q2 schedule
python oncall_scheduler.py --config my_config.yaml \
  --start-date 2026-04-01 --end-date 2026-07-01
```

### Monthly Schedules

```bash
# January (absolute dates)
python oncall_scheduler.py --config my_config.yaml \
  --start-date 2026-01-01 --end-date 2026-02-01

# February (absolute dates)
python oncall_scheduler.py --config my_config.yaml \
  --start-date 2026-02-01 --end-date 2026-03-01

# This month (relative - easier!)
python oncall_scheduler.py --config my_config.yaml \
  --start-date today --end-date +1m

# Next month
python oncall_scheduler.py --config my_config.yaml \
  --start-date +1m --end-date +1m
```

### Testing & Preview

```bash
# Test with just 1 week
python oncall_scheduler.py --config my_config.yaml \
  --start-date today --end-date +7d

# Verify rotation with 3 days
python oncall_scheduler.py --config my_config.yaml \
  --start-date today --end-date +3d
```

## Platform-Specific Notes

### macOS / Linux

- Use `python3` if your system has both Python 2 and 3
- Virtual environment activation: `source venv/bin/activate`
- Line continuation in commands: `\`
- File paths use forward slashes: `/path/to/file`

### Windows

- Use `python` (usually points to Python 3)
- Virtual environment activation: `venv\Scripts\activate`
- Line continuation in cmd: `^` or PowerShell: `` ` ``
- File paths use backslashes: `\path\to\file` or forward slashes work too

### All Platforms

To deactivate the virtual environment when done:
```bash
deactivate
```

## Troubleshooting

### "Command not found: python" or "python3"

**macOS/Linux:**
```bash
# Check if Python is installed
which python3
# or install with package manager
sudo apt install python3 python3-pip  # Ubuntu/Debian
brew install python3  # macOS with Homebrew
```

**Windows:**
- Download from [python.org](https://www.python.org/downloads/)
- Check "Add Python to PATH" during installation

### "No module named 'openpyxl'" or similar

Make sure you activated the virtual environment and installed requirements:
```bash
source venv/bin/activate  # macOS/Linux
venv\Scripts\activate     # Windows
pip install -r requirements.txt
```

### No shifts generated

- Check `time_windows` in your YAML config includes the days you want
- Verify date range includes those days of the week
- Check console output for any error messages

### Excel shows #VALUE! or formula errors

- Open the file and enable editing (Excel protected view)
- Excel Online/Mac may need formula compatibility mode
- The "On-Call Status" column uses NOW() which updates when you open the file

### Visual chart is crowded

- Colors and text overlap? The chart is optimized for the full date range
- Zoom in on the PNG file for detail
- Consider generating shorter periods for very detailed views

### iCalendar files not importing

- Make sure you used `--generate-ics` flag
- Files are in `config_name/ics_files/` folder
- Most calendar apps: File → Import → Select .ics file
- Some apps prefer drag-and-drop

## Tips & Best Practices

- **Start with a preview**: Generate a 2-week schedule first to verify your configuration
- **Version control**: Keep your YAML configuration files in git to track changes
- **Date alignment**: Start on Monday for cleaner weekly boundaries
- **Calendar integration**: Use `--generate-ics` and share .ics files with your team
- **Excel formulas**: The "On-Call Status" column automatically highlights current shifts
- **Visual sharing**: PNG charts are great for team presentations and dashboards
- **Fair distribution**: Use dummy shifts to balance rotation when coverage hours differ by day

## Dependencies

- Python 3.7+
- openpyxl - Excel file generation
- pyyaml - YAML configuration parsing
- python-dateutil - Date calculation utilities
- matplotlib - Visual chart generation

All dependencies are installed automatically via `pip install -r requirements.txt`

## License

[Include your license information here]

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

## Support

For issues, questions, or suggestions, please open an issue on GitHub.
