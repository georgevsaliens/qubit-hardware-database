# Qubit Hardware Database

A comprehensive database tracking the evolution and performance of quantum computing hardware across different architectures.

| ![image](https://github.com/user-attachments/assets/835ceb3e-3261-4f6e-8320-5a0c991cc078) |
|:--:| 
| *See example plots in "Qubit Database.ipynb" file* |

## Overview

The Qubit Hardware Database is an open-source project that collects and organizes data on quantum computing hardware from published research papers. The database tracks key performance metrics such as:

- Coherence times (T1, T2)
- Gate fidelities (single and two-qubit)
- Error rates (measurement, state preparation, SPAM)
- Gate operation times
- Qubit counts
- Operating temperatures

This repository provides tools to access, visualize, and analyze trends in quantum computing hardware development across different architectures (superconducting, trapped ion, photonic, etc.) and years.

## Data Source

The database is populated from a [Google Sheet](https://docs.google.com/spreadsheets/d/158mz7xAjDFkdbqp3O21ImE8iuDVOflKmgsl9-1Y5QcE/edit?gid=0#gid=0) that contains manually entered data from research papers. Each entry includes:

- DOI (Digital Object Identifier) of the paper
- Hardware type
- Year of publication
- Number of qubits
- Performance metrics
- Summary of the work
- Additional notes

## Using the Database

### Database Structure

The database contains:
1. A main table with basic information about each paper
2. Normalized tables for specific metrics:
   - T1_Times
   - T2_Times
   - Single_Qubit_Fidelity
   - Two_Qubit_Fidelity
   - Measurement_Error
   - State_Prep_Error
   - SPAM_Error
   - Single_Qubit_Gate_time
   - Two_Qubit_Gate_time

### Accessing Data

```python
import sqlite3
import pandas as pd

# Connect to the database
conn = sqlite3.connect('qubit_data.db')

# Simple query to get all data from main table
main_df = pd.read_sql_query("SELECT * FROM main", conn)

# Query specific metrics
t1_data = pd.read_sql_query("SELECT * FROM T1_Times", conn)

# Join metrics with main table information
query = """
SELECT m.DOI, m.Hardware_Type, m.Year, m.Number_of_Qubits,
       t.Value, t.Uncertainty
FROM T1_Times t
JOIN main m ON t.DOI = m.DOI
"""
combined_data = pd.read_sql_query(query, conn)

# Always close the connection when done
conn.close()
```

### Creating Visualizations

The repository includes examples of various visualizations you can create, all included in the "Qubit Database.ipynb" file:

#### Basic Time Series Plots

```python
import matplotlib.pyplot as plt
import seaborn as sns
import sqlite3
import pandas as pd

# Connect to the database
conn = sqlite3.connect('qubit_data.db')

# Load data with joins
def load_table_data(table_name):
    query = f"""
    SELECT m.DOI, m.Hardware_Type, m.Year, m.Number_of_Qubits,
           t.Value, t.Uncertainty
    FROM {table_name} t
    JOIN main m ON t.DOI = m.DOI
    """
    return pd.read_sql_query(query, conn)

# Example: Plot T1 Times vs Year
df = load_table_data('T1_Times')
df['Year'] = pd.to_numeric(df['Year'], errors='coerce')
df = df.dropna(subset=['Year', 'Value'])

plt.figure(figsize=(12, 6))
sns.scatterplot(
    data=df,
    x='Year',
    y='Value',
    hue='Hardware_Type',
    s=50,
    alpha=0.8
)
plt.yscale('log')
plt.title('T1 Coherence Times Over Years by Hardware Type')
plt.xlabel('Year')
plt.ylabel('T1 Time (ms, log scale)')
plt.legend(title='Hardware Type', bbox_to_anchor=(1.05, 1), loc='upper left')
plt.tight_layout()
plt.show()

conn.close()
```

#### Comparing Multiple Metrics

```python
import sqlite3
import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns
import numpy as np

conn = sqlite3.connect('qubit_data.db')

# Load both single and two qubit fidelity data
df_1q = pd.read_sql_query("""
    SELECT m.DOI, m.Hardware_Type, m.Year, t.Value
    FROM Single_Qubit_Fidelity t
    JOIN main m ON t.DOI = m.DOI
""", conn)
df_1q['Gate_Type'] = 'Single Qubit'

df_2q = pd.read_sql_query("""
    SELECT m.DOI, m.Hardware_Type, m.Year, t.Value
    FROM Two_Qubit_Fidelity t
    JOIN main m ON t.DOI = m.DOI
""", conn)
df_2q['Gate_Type'] = 'Two Qubit'

# Combine the data
df_fidelity = pd.concat([df_1q, df_2q])
df_fidelity['Year'] = pd.to_numeric(df_fidelity['Year'], errors='coerce')
df_fidelity = df_fidelity.dropna(subset=['Year', 'Value'])

# Convert fidelity to error rate (1 - fidelity)
df_fidelity['Error_Rate'] = 1 - df_fidelity['Value']

# Create figure with two subplots
fig, (ax1, ax2) = plt.subplots(1, 2, figsize=(18, 8))

# Plot 1: Original fidelity plot
sns.lineplot(
    data=df_fidelity,
    x='Year',
    y='Value',
    hue='Hardware_Type',
    style='Gate_Type',
    markers=True,
    dashes=True,
    ax=ax1
)
ax1.set_title('Gate Fidelities over Time by Hardware Type')
ax1.set_xlabel('Year')
ax1.set_ylabel('Gate Fidelity')
ax1.set_ylim(0, 1.05)  # Set y-axis to show full range

# Plot 2: Log scale error rate
sns.lineplot(
    data=df_fidelity,
    x='Year',
    y='Error_Rate',
    hue='Hardware_Type',
    style='Gate_Type',
    markers=True,
    dashes=True,
    ax=ax2
)
ax2.set_yscale('log')  # Set logarithmic scale for y-axis
ax2.set_title('Gate Error Rates (Log Scale) over Time by Hardware Type')
ax2.set_xlabel('Year')
ax2.set_ylabel('Gate Error Rate (1 - Fidelity)')
ax2.grid(True, which="both", ls="-", alpha=0.2)

# Improve legend positioning and size
handles, labels = ax2.get_legend_handles_labels()
fig.legend(handles, labels, loc='upper center', bbox_to_anchor=(0.5, 0), ncol=5)
ax1.get_legend().remove()
ax2.get_legend().remove()

plt.tight_layout(rect=[0, 0.05, 1, 0.95])
plt.suptitle('Comparison of Gate Performance Metrics', fontsize=16, y=0.98)
plt.show()


conn.close()
```

## Contributing

You can add new datapoints to this [Google Sheet](https://docs.google.com/spreadsheets/d/158mz7xAjDFkdbqp3O21ImE8iuDVOflKmgsl9-1Y5QcE/edit?gid=0#gid=0). Here is how new data should be entered:
1. For the normalized data columns if there are multiple entries seperate them by a ";" character.
2. If there are any uncertatnties in measurments add ":" character (e.g. value:uncertanty; ...)
3. For the name of the Hardware type try and keep it the same as one of the other names in the other columns

## Future Enhancements

Potential improvements for this project include:

1. **Web Interface**
   - Create a web-based dashboard for exploring the data
   - Implement interactive visualizations with filters

2. **Automated Data Collection**
   - Develop scripts to scrape papers for relevant metrics
   - Implement natural language processing to extract quantum hardware details

3. **Extended Database Schema**
   - Add more metrics (e.g., quantum volume, circuit depth capabilities)
   - Track more details about architecture (e.g., connectivity, control systems)

4. **Error Bar Support**
   - Improve handling of measurement uncertainties
   - Implement visualization of error bars on plots

5. **Statistical Analysis**
   - Add trend line calculations and projections
   - Implement statistical analysis of hardware improvement rates

6. **Real-time Updates**
   - Implement automatic database updates when the Google Sheet changes
   - Create API endpoints for accessing the latest data

7. **Cross-Platform Support**
   - Package the database for easier installation
   - Create Docker containers for consistent environments
