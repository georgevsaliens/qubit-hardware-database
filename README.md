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

The database is populated from a Google Sheet that contains manually curated data from research papers. Each entry includes:

- DOI (Digital Object Identifier) of the paper
- Hardware type
- Year of publication
- Number of qubits
- Performance metrics
- Summary of the work
- Additional notes

## Getting Started

### Prerequisites

- Python 3.6+
- Required packages: pandas, sqlite3, matplotlib, seaborn, plotly (for interactive visualizations)

### Installation

1. Clone this repository:
   ```bash
   git clone https://github.com/yourusername/qubit-hardware-database.git
   cd qubit-hardware-database
   ```

2. Install required packages:
   ```bash
   pip install pandas sqlite3 matplotlib seaborn plotly
   ```

3. Run the database generation script:
   ```bash
   python qubit_database.py
   ```

This will:
- Connect to the Google Sheet data source
- Process and clean the data
- Create an SQLite database (`qubit_data.db`)
- Generate example plots (if run as a script)

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

The repository includes examples of various visualizations you can create:

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

#### Interactive Plots with Plotly

```python
import plotly.express as px

# Load T1 data with joins
conn = sqlite3.connect('qubit_data.db')
query = """
SELECT m.DOI, m.Hardware_Type, m.Year, m.Number_of_Qubits,
       t.Value
FROM T1_Times t
JOIN main m ON t.DOI = m.DOI
"""
df = pd.read_sql_query(query, conn)
df['Year'] = pd.to_numeric(df['Year'], errors='coerce')
df['Number_of_Qubits'] = pd.to_numeric(df['Number_of_Qubits'], errors='coerce')
df = df.dropna(subset=['Year', 'Value'])

# Create interactive plot
fig = px.scatter(
    df,
    x='Year',
    y='Value',
    color='Hardware_Type',
    size='Number_of_Qubits',
    hover_name='DOI',
    title='T1 Coherence Time Evolution',
    labels={'Value': 'T1 Time (ms)', 'Year': 'Year'},
    height=600
)

fig.update_yaxes(type="log")
fig.show()

conn.close()
```

#### Comparing Multiple Metrics

```python
# Example: Compare gate fidelities
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

plt.figure(figsize=(12, 8))
sns.lineplot(
    data=df_fidelity,
    x='Year',
    y='Value',
    hue='Hardware_Type',
    style='Gate_Type',
    markers=True,
    dashes=True
)

plt.title('Gate Fidelities over Time by Hardware Type')
plt.xlabel('Year')
plt.ylabel('Gate Fidelity')
plt.show()

conn.close()
```

## Contributing

Contributions to the Qubit Hardware Database are welcome! Here's how you can help:

1. **Add new data**: Update the Google Sheet with data from newly published papers
2. **Improve data processing**: Enhance the data cleaning and parsing logic
3. **Add new visualizations**: Create new types of plots or analyses
4. **Enhance the database schema**: Suggest improvements to how the data is stored

Please submit pull requests with any improvements you make.

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

## License

This project is licensed under the MIT License - see the LICENSE file for details.

## Acknowledgments

- Contributors who helped collect and curate the quantum hardware data
- Researchers publishing detailed metrics on their quantum hardware implementations
- Python data science community for the tools that make this analysis possible
