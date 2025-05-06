# python-project
import requests
import pandas as pd

data_url = "https://raw.githubusercontent.com/CSSEGISandData/COVID-19/master/csse_covid_19_data/csse_covid_19_time_series/time_series_covid19_confirmed_global.csv"

try:
    response = requests.get(data_url)
    response.raise_for_status()  # Raise an exception for bad status codes
    csv_data = response.text
    df = pd.read_csv(io.StringIO(csv_data))
    print("Data fetched successfully!")
    print(df.head())
except requests.exceptions.RequestException as e:
    print(f"Error fetching data: {e}")
except pd.errors.EmptyDataError:
    print("Error: Fetched data is empty.")
except Exception as e:
    print(f"An unexpected error occurred: {e}")

    import pandas as pd
import io

# Assuming 'csv_data' contains the fetched CSV string
csv_data = """Province/State,Country/Region,Lat,Long,1/22/20,1/23/20
,,0,0,1,2
Hubei,China,30.9756,112.2707,444,444
"""
df = pd.read_csv(io.StringIO(csv_data))

# Melt the dataframe to long format for easier analysis
df_melted = df.melt(id_vars=['Province/State', 'Country/Region', 'Lat', 'Long'],
                    var_name='Date',
                    value_name='Confirmed')

# Convert the 'Date' column to datetime objects
df_melted['Date'] = pd.to_datetime(df_melted['Date'])

# Rename columns for clarity
df_melted.rename(columns={'Province/State': 'Province_State',
                          'Country/Region': 'Country_Region'}, inplace=True)

print("\nProcessed Data:")
print(df_melted.head())
import matplotlib.pyplot as plt
import seaborn as sns

# Assuming 'df_melted' has been created
country = "China"
country_data = df_melted[df_melted['Country_Region'] == country].groupby('Date')['Confirmed'].sum().reset_index()

plt.figure(figsize=(12, 6))
sns.lineplot(x='Date', y='Confirmed', data=country_data)
plt.title(f'COVID-19 Confirmed Cases in {country}')
plt.xlabel('Date')
plt.ylabel('Confirmed Cases')
plt.grid(True)
plt.show()
import plotly.express as px

# Assuming 'df_melted' has been created
country = "Italy"
country_data = df_melted[df_melted['Country_Region'] == country].groupby('Date')['Confirmed'].sum().reset_index()

fig = px.line(country_data, x='Date', y='Confirmed',
              title=f'COVID-19 Confirmed Cases in {country}')
fig.show()
from flask import Flask, render_template
import pandas as pd
import io
import requests
import plotly.express as px
import plotly.offline as pyo

app = Flask(__name__)

def fetch_and_process_data():
    data_url = "https://raw.githubusercontent.com/CSSEGISandData/COVID-19/master/csse_covid_19_data/csse_covid_19_time_series/time_series_covid19_confirmed_global.csv"
    response = requests.get(data_url)
    response.raise_for_status()
    csv_data = response.text
    df = pd.read_csv(io.StringIO(csv_data))
    df_melted = df.melt(id_vars=['Province/State', 'Country/Region', 'Lat', 'Long'],
                        var_name='Date',
                        value_name='Confirmed')
    df_melted['Date'] = pd.to_datetime(df_melted['Date'])
    return df_melted

@app.route('/')
def index():
    df_processed = fetch_and_process_data()
    country_data = df_processed[df_processed['Country_Region'] == 'Kenya'].groupby('Date')['Confirmed'].sum().reset_index()
    fig = px.line(country_data, x='Date', y='Confirmed', title='COVID-19 Confirmed Cases in Kenya')
    plot_div = pyo.plot(fig, output_type='div')
    return render_template('index.html', plot_div=plot_div)

if __name__ == '__main__':
    app.run(debug=True)
    <!DOCTYPE html>
<html>
<head>
    <title>COVID-19 Data Tracker</title>
</head>
<body>
    <h1>COVID-19 Data for Kenya</h1>
    <div>
        {{ plot_div | safe }}
    </div>
</body>
</html>
