# energy-dashboard

Creating an interactive web application for monitoring and visualizing home energy consumption is a complex task that can be broken down into several steps. This project will include setting up a web server, a database to store energy data, real-time updates, and visualization capabilities. We'll use Flask for the web application, SQLite for storing data, and Plotly for visualization.

Here's a complete Python program that provides a basic implementation of the energy-dashboard:

```python
from flask import Flask, render_template, request, jsonify
import sqlite3
import random
import plotly.graph_objs as go
import plotly.io as pio

app = Flask(__name__)

# Database setup
DATABASE = 'energy_data.db'

def get_db_connection():
    """Create a connection to the SQLite database."""
    conn = sqlite3.connect(DATABASE)
    conn.row_factory = sqlite3.Row
    return conn

def create_table():
    """Create the energy consumption table if not exists."""
    conn = get_db_connection()
    conn.execute('''
        CREATE TABLE IF NOT EXISTS energy_consumption (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            timestamp DATETIME DEFAULT CURRENT_TIMESTAMP,
            consumption REAL
        )
    ''')
    conn.commit()
    conn.close()

create_table()

# Error handling
@app.errorhandler(404)
def not_found_error(error):
    """Handle 404 errors."""
    return render_template('404.html'), 404

@app.errorhandler(500)
def internal_error(error):
    """Handle 500 errors."""
    return render_template('500.html'), 500

@app.route('/')
def index():
    """Render the HTML page."""
    return render_template('index.html')

@app.route('/add-data', methods=['POST'])
def add_data():
    """Add data endpoint for simulated real-time data."""
    try:
        # Simulate real-time data update with random numbers
        consumption = random.uniform(0.1, 5.0)
        conn = get_db_connection()
        conn.execute('INSERT INTO energy_consumption (consumption) VALUES (?)', (consumption,))
        conn.commit()
        conn.close()
        return 'Data added successfully', 200
    except Exception as e:
        return str(e), 500

@app.route('/get-data', methods=['GET'])
def get_data():
    """Get data for visualization."""
    try:
        conn = get_db_connection()
        data = conn.execute('SELECT * FROM energy_consumption ORDER BY timestamp DESC LIMIT 100').fetchall()
        conn.close()
        return jsonify([dict(row) for row in data])
    except Exception as e:
        return str(e), 500

@app.route('/visualize', methods=['GET'])
def visualize():
    """Return plotly graph of energy consumption."""
    try:
        conn = get_db_connection()
        data = conn.execute('SELECT * FROM energy_consumption ORDER BY timestamp DESC LIMIT 100').fetchall()
        conn.close()

        timestamps = [row['timestamp'] for row in data]
        values = [row['consumption'] for row in data]

        # Create a plotly graph
        fig = go.Figure(data=go.Scatter(x=timestamps, y=values, mode='lines+markers'))
        fig.update_layout(title='Home Energy Consumption Over Time',
                          xaxis_title='Timestamp',
                          yaxis_title='Consumption (kWh)')
        graph_html = pio.to_html(fig, full_html=False)
        return graph_html
    except Exception as e:
        return str(e), 500

if __name__ == '__main__':
    app.run(debug=True)
```

### Considerations:
- **Real-time Data Simulation**: This implementation simulates data updates by generating random energy consumption values. In a real-world scenario, you'd collect this data from IoT devices or sensors.
- **Visualization**: The app uses Plotly, a popular tool for creating interactive graphs.
- **Error Handling**: Basic error handling is included for 404 and 500 errors.
- **Database**: SQLite is used for simplicity, but for a production environment, consider using a more scalable database like PostgreSQL.
- **Testing**: Test each component to ensure it functions correctly. Implement unit tests for business logic and integration tests to ensure end-to-end functionality.

### Prerequisites:
- Install the necessary libraries using `pip install flask plotly` before running the application.
- Extend security and optimize for performance if deploying to a production environment.