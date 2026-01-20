# PagerDuty-Style On-Call Shift Scheduler

Generate professional on-call shift schedules in Excel format with visual representation. Uses PagerDuty-style layer definitions where each layer represents a single time window that rotates daily.

## Key Features

- **Excel output** - Detailed schedule with all shifts, dates, and assignments
- **Visual output** - Color-coded PNG chart showing the schedule visually
- **7 layers configuration** - 4 layers for Mon-Thu (08:00-18:00), 3 layers for Friday (08:00-13:00)
- **Shifted rotation** - Each layer has the same 10 users shifted by 1 position
- **Exact dates** - Real calendar dates from start to end date
- **3-month default** - Configurable date range (default: today + 3 months)

## Layer Configuration

### Monday to Thursday (4 layers)
- **Layer 1**: 08:00-10:30
- **Layer 2**: 10:30-13:00
- **Layer 3**: 13:00-15:30
- **Layer 4**: 15:30-18:00

### Friday (3 layers)
- **Layer 5**: 08:00-09:30
- **Layer 6**: 09:30-11:00
- **Layer 7**: 11:00-13:00

### Rotation Logic

All 7 layers use the same 10 users, but **shifted backwards by 1 position** per layer:

```
Layer 1: Utente 1, Utente 2, Utente 3, ..., Utente 10
Layer 2: Utente 10, Utente 1, Utente 2, ..., Utente 9
Layer 3: Utente 9, Utente 10, Utente 1, ..., Utente 8
Layer 4: Utente 8, Utente 9, Utente 10, ..., Utente 7
Layer 5: Utente 7, Utente 8, Utente 9, ..., Utente 6
Layer 6: Utente 6, Utente 7, Utente 8, ..., Utente 5
Layer 7: Utente 5, Utente 6, Utente 7, ..., Utente 4
```

Each layer rotates its team members **daily**.

## Installation

```bash
python3 -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate
pip install -r requirements.txt
```

## Usage

### Basic Commands

```bash
# Generate schedule with defaults (today + 3 months)
python oncall_scheduler.py --config layers_2_5h_shifts.yaml

# Custom date range
python oncall_scheduler.py --config layers_2_5h_shifts.yaml \
  --start-date 2026-01-20 --end-date 2026-04-20

# Custom output filename
python oncall_scheduler.py --config layers_2_5h_shifts.yaml \
  --output my_schedule.xlsx

# Specific month
python oncall_scheduler.py --config layers_2_5h_shifts.yaml \
  --start-date 2026-02-01 --end-date 2026-03-01 \
  --output february_2026.xlsx
```

### Output Files

The tool generates **two files**:

1. **Excel file** (`.xlsx`) - Complete schedule with:
   - Date (exact calendar date)
   - Day (Monday, Tuesday, etc.)
   - Layer (Layer 1, Layer 2, etc.)
   - Time Window (e.g., 08:00 - 10:30)
   - On-Call Person (Utente 1, Utente 2, etc.)
   - Color-coded by person

2. **Visual chart** (`.png`) - Graphical representation with:
   - Timeline view (vertical: time, horizontal: dates)
   - Color bars showing who's on-call
   - Person names on each shift bar
   - Legend with all team members
   - Limited to first 30 days for readability

### Parameters

- `--config`: Path to YAML configuration file (required)
- `--output`: Output xlsx filename (default: oncall_schedule.xlsx)
- `--start-date`: Start date YYYY-MM-DD (default: today)
- `--end-date`: End date YYYY-MM-DD (default: start + 3 months)

## Example Rotation

### Week 1 (Jan 20-24, 2026)

**Monday, Jan 20:**
- 08:00-10:30: Utente 1 (Layer 1)
- 10:30-13:00: Utente 10 (Layer 2)
- 13:00-15:30: Utente 9 (Layer 3)
- 15:30-18:00: Utente 8 (Layer 4)

**Tuesday, Jan 21:**
- 08:00-10:30: Utente 2 (Layer 1)
- 10:30-13:00: Utente 1 (Layer 2)
- 13:00-15:30: Utente 10 (Layer 3)
- 15:30-18:00: Utente 9 (Layer 4)

**Wednesday, Jan 22:**
- 08:00-10:30: Utente 3 (Layer 1)
- 10:30-13:00: Utente 2 (Layer 2)
- 13:00-15:30: Utente 1 (Layer 3)
- 15:30-18:00: Utente 10 (Layer 4)

**Thursday, Jan 23:**
- 08:00-10:30: Utente 4 (Layer 1)
- 10:30-13:00: Utente 3 (Layer 2)
- 13:00-15:30: Utente 2 (Layer 3)
- 15:30-18:00: Utente 1 (Layer 4)

**Friday, Jan 24:**
- 08:00-09:30: Utente 5 (Layer 5)
- 09:30-11:00: Utente 4 (Layer 6)
- 11:00-13:00: Utente 3 (Layer 7)

This ensures:
- **Daily rotation**: Each person rotates to the next day
- **Layer shift**: Each layer starts with a different person (backwards shift)
- **Fair distribution**: Everyone gets similar coverage across time slots

## Configuration File Structure

Edit `layers_2_5h_shifts.yaml` to customize:

```yaml
schedule:
  name: "Your Schedule Name"
  start_date: "2026-01-20"
  duration_months: 3
  
  layers:
    layer1:
      name: "Layer 1"
      time_window:
        start: "08:00"
        end: "10:30"
      days: ["monday", "tuesday", "wednesday", "thursday"]
      rotation_team:
        - "Utente 1"
        - "Utente 2"
        # ... up to Utente 10
    
    layer2:
      # Same structure, shifted team list
      # ...
```

## Visual Output Details

The PNG chart shows:
- **X-axis**: Calendar dates with day names
- **Y-axis**: Time (08:00 to 18:00)
- **Color bars**: Each person has a unique color
- **Names**: Person names displayed on shift bars
- **Legend**: All team members with their colors
- **Title**: Schedule name and date range

For schedules longer than 30 days, only the first 30 days are visualized (Excel contains full schedule).

## Use Cases

### Quarterly Planning
```bash
# Q1 2026
python oncall_scheduler.py --config layers_2_5h_shifts.yaml \
  --start-date 2026-01-01 --end-date 2026-04-01 --output q1_2026.xlsx

# Q2 2026
python oncall_scheduler.py --config layers_2_5h_shifts.yaml \
  --start-date 2026-04-01 --end-date 2026-07-01 --output q2_2026.xlsx
```

### Monthly Schedules
```bash
# January
python oncall_scheduler.py --config layers_2_5h_shifts.yaml \
  --start-date 2026-01-01 --end-date 2026-02-01 --output jan_2026.xlsx

# February
python oncall_scheduler.py --config layers_2_5h_shifts.yaml \
  --start-date 2026-02-01 --end-date 2026-03-01 --output feb_2026.xlsx
```

### Preview Mode
```bash
# Generate just 2 weeks to preview
python oncall_scheduler.py --config layers_2_5h_shifts.yaml \
  --start-date 2026-01-20 --end-date 2026-02-03 --output preview.xlsx
```

## Tips

- **Visual preview**: Generate short periods (1-2 weeks) first to verify rotation
- **Excel for details**: Use Excel file for complete schedule and planning
- **PNG for sharing**: Use PNG chart for quick visualization and presentations
- **Date alignment**: Start dates on Monday for cleaner weekly views
- **Configuration**: Keep YAML file in version control to track team changes

## Troubleshooting

### No shifts on certain days
- Check `days` field in YAML (monday, tuesday, etc.)
- Verify date range includes those days

### Wrong rotation
- Verify `rotation_team` lists are correctly shifted by 1 per layer
- Check that all 10 users are present in each layer

### Visual too crowded
- Use shorter date ranges (1-2 weeks) for detailed view
- Full schedule always available in Excel file

### Colors hard to distinguish
- Script uses optimized color palette
- Check person names on bars and legend

## Requirements

- Python 3.7+
- openpyxl (Excel generation)
- pyyaml (Configuration parsing)
- python-dateutil (Date calculations)
- matplotlib (Visual chart generation)
