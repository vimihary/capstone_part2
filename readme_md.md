# Metro Interstate Traffic Analytics Pipeline & CLI Application

A reproducible end-to-end Python data processing pipeline and CLI analytics tool built for analyzing Metro Interstate Traffic Volume data.

---

## 📁 Project Structure

```text
capstone_part2/
│
├── figures/                                   # Generated output charts and visualisations
│   ├── 01_hourly_traffic_weekday_vs_weekend.png
│   ├── 02_temperature_vs_traffic.png
│   └── 03_traffic_by_weather_main.png
│
├── Metro_Interstate_Traffic_Volume.csv        # Raw input dataset
├── cleaned_traffic_data.csv                   # Cleaned output dataset post-pipeline execution
├── featured_traffic_data.csv                  # Dataset post-feature engineering
│
├── pipeline.py                                # Data ingestion, schema validation, and cleaning module
├── feature_engineering.py                     # Feature engineering, scaling, and target creation module
├── visualizations.py                          # Matplotlib figure generation module
├── traffic_app.py                             # Command-line analytics interface application
│
├── pipeline.log                               # Auditable execution log file
└── README.md                                  # Project documentation
```

---

## ⚙️ Environment & Dependencies

Ensure Python 3.8+ and the following required libraries are installed:

```bash
pip install pandas numpy matplotlib seaborn
```

---

## 🚀 How to Run the Project

Execute the Python scripts sequentially to run the data processing pipeline, engineer features, export visualizations, and interact with the CLI application.

### Step 1: Run Data Cleaning Pipeline
Validates dataset schema, cleans string categories, converts date/time fields, removes duplicate rows, and imputes physical outliers (e.g., 0 K temperatures and extreme rainfall) using monthly medians:
```bash
python pipeline.py
```

### Step 2: Run Feature Engineering
Generates cyclical time encodings (sin/cos), scales numerical features, creates binary weather indicators, and computes quartile-based traffic congestion categories:
```bash
python feature_engineering.py
```

### Step 3: Generate Visualizations
Creates and saves high-resolution traffic analysis charts directly to the `figures/` directory:
```bash
python visualizations.py
```

### Step 4: Run CLI Application Queries
Interact with and query the processed traffic dataset via command-line arguments:

* **Query traffic for a specific date and time:**
  ```bash
  python traffic_app.py query-datetime --datetime "2012-10-02 09:00:00"
  ```

* **Compare weekday vs. weekend traffic summary statistics:**
  ```bash
  python traffic_app.py compare-daytype
  ```

* **Get low-traffic travel recommendations for a given day:**
  ```bash
  python traffic_app.py recommend-travel --date "2012-10-02"
  ```

---

## 🪵 Logging Architecture & Configuration

### Overview & Setup
* **Module-Level Loggers:** Each library script instantiates its own independent logger using `logger = logging.getLogger(__name__)`. The bare root logger is never called inside library code.
* **Dual Handler Setup:** Handlers are configured in entry-point execution scripts to stream logs simultaneously to the console standard output (`StreamHandler`) and append to a log file (`FileHandler` writing to `pipeline.log`).
* **Standard Formatter:** Output logs strictly follow the minimal required schema:
  ```text
  %(asctime)s - %(levelname)s - %(name)s - %(message)s
  ```

### Log Levels & Usage

| Level | Purpose & Pipeline Context | Representative Log Output |
| :--- | :--- | :--- |
| **`DEBUG`** | Fine-grained intermediate values and threshold logic useful for troubleshooting. Active only during debug mode execution. | `2026-10-05 14:04:25 - DEBUG - feature_engineering - Congestion quartile thresholds computed: 25%=1193.0, 50%=3380.0, 75%=4933.0` |
| **`INFO`** | Milestone progress events, successful data loading, file generation, and CLI commands. | `2026-10-05 13:52:50 - INFO - __main__ - Successfully loaded raw file 'Metro_Interstate_Traffic_Volume.csv'. Loaded 48204 rows and 9 columns.` |
| **`WARNING`** | Recoverable data issues and cleaning adjustments, such as dropped duplicates, filled nulls, or imputed outliers. | `2026-10-05 13:52:50 - WARNING - __main__ - Outlier Imputation: Replaced 4 invalid temperature rows (0 K) in month 1 with monthly median (266.83 K).` |
| **`ERROR`** | Critical pipeline failures or malformed user CLI input parameters, logged gracefully without dumping raw tracebacks. | `2026-10-05 15:21:12 - ERROR - __main__ - Invalid input format for query-datetime 'invalid-date-string': Unknown datetime string format...` |

### Status Reporting Policy
* `print()` statements are restricted exclusively to outputting requested CLI query response values directly to the end-user terminal.
* All internal status reporting, progress tracking, warning alerts, and error diagnostics are managed through the Python `logging` module.