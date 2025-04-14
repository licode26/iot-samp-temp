# iot-samp-temp  
this is the sample project for   temperature showcase dashboard and given the alert
given by prithwiraj charchi just create sample project for iot subject


from flask import Flask, render_template, jsonify
import sqlite3
from datetime import datetime
import Adafruit_DHT  # Library to read from DHT11
from database.init_db import init_db

app = Flask(__name__)
DB_NAME = 'sensor_data.db'
TEMP_THRESHOLD = 40

# DHT Sensor configuration
DHT_SENSOR = Adafruit_DHT.DHT11
DHT_PIN = 4  # GPIO pin where the sensor is connected (change if needed)

# Collect real sensor data
def collect_sensor_data():
    humidity, temperature = Adafruit_DHT.read_retry(DHT_SENSOR, DHT_PIN)
    if humidity is not None and temperature is not None:
        timestamp = datetime.utcnow().isoformat()
        conn = sqlite3.connect(DB_NAME)
        cursor = conn.cursor()
        cursor.execute("INSERT INTO readings (temperature, humidity, timestamp) VALUES (?, ?, ?)",
                       (temperature, humidity, timestamp))
        conn.commit()
        conn.close()
        return temperature, humidity, timestamp
    else:
        return None, None, None

# Home route
@app.route('/')
def index():
    return render_template('index.html')

# API to collect and return latest data
@app.route('/api/latest', methods=['GET'])
def get_latest_data():
    temperature, humidity, timestamp = collect_sensor_data()
    if temperature is None:
        return jsonify({"error": "Failed to read from sensor"}), 500

    data = {
        "temperature": temperature,
        "humidity": humidity,
        "timestamp": timestamp,
        "alert": temperature > TEMP_THRESHOLD
    }
    return jsonify(data)

if __name__ == '__main__':
    init_db()
    app.run(debug=True)



    update the code when iot is enabled
