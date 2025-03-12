import RPi.GPIO as GPIO
import time
import requests

# GPIO Pin configuration
SOIL_MOISTURE_PIN = 17  # GPIO pin for soil moisture sensor
PUMP_PIN = 27  # GPIO pin for water pump relay

# ThingSpeak API Configuration (Optional - For Cloud Monitoring)
THINGSPEAK_API_KEY = "your_thingspeak_api_key"
THINGSPEAK_URL = "https://api.thingspeak.com/update"

# Moisture threshold (Adjustable based on soil type)
MOISTURE_THRESHOLD = 400  # Example threshold value

# Setup GPIO
GPIO.setwarnings(False)
GPIO.setmode(GPIO.BCM)
GPIO.setup(SOIL_MOISTURE_PIN, GPIO.IN)
GPIO.setup(PUMP_PIN, GPIO.OUT)

def read_soil_moisture():
    moisture_value = GPIO.input(SOIL_MOISTURE_PIN)  # 0: Wet, 1: Dry
    return moisture_value

def send_to_cloud(moisture):
    payload = {
        'api_key': THINGSPEAK_API_KEY,
        'field1': moisture
    }
    try:
        response = requests.get(THINGSPEAK_URL, params=payload)
        if response.status_code == 200:
            print("Data sent to ThingSpeak successfully!")
        else:
            print("Failed to send data. Response code:", response.status_code)
    except Exception as e:
        print("Error sending data to cloud:", e)

def water_plants():
    print("Soil is dry! Activating water pump...")
    GPIO.output(PUMP_PIN, GPIO.HIGH)
    time.sleep(5)  # Pump water for 5 seconds
    GPIO.output(PUMP_PIN, GPIO.LOW)
    print("Watering complete!")

print("Automated Plant Watering System Active")
try:
    while True:
        soil_moisture = read_soil_moisture()
        print(f"Soil Moisture Level: {'Dry' if soil_moisture else 'Wet'}")
        if soil_moisture:  # If soil is dry
            water_plants()
        send_to_cloud(soil_moisture)
        time.sleep(30)  # Check every 30 seconds
except KeyboardInterrupt:
    print("Shutting down...")
    GPIO.cleanup()

