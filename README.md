from flask import Flask, render_template_string, request
from adafruit_servokit import ServoKit
import time

app = Flask(__name__)

# Initialize PCA9685 (16 channels)
# The Pi 5 uses I2C bus 1 by default
kit = ServoKit(channels=16)

# Define servo channels based on your setup
# 0-3: MG996R (Arm joints), 4: MG996S (Gripper/End effector)
servos = {
    "base": 0,
    "shoulder": 1,
    "elbow": 2,
    "wrist": 3,
    "gripper": 4
}

# HTML Template for the Mobile Interface
HTML_TEMPLATE = """
<!DOCTYPE html>
<html>
<head>
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Robotic Arm Control</title>
    <style>
        body { font-family: Arial; text-align: center; background: #222; color: white; }
        .slider-container { margin: 20px; padding: 15px; border: 1px solid #444; border-radius: 10px; }
        input[type=range] { width: 80%; height: 25px; }
        label { font-size: 1.2em; display: block; margin-bottom: 10px; }
    </style>
</head>
<body>
    <h1>Arm Controller</h1>
    {% for name, channel in servos.items() %}
    <div class="slider-container">
        <label>{{ name.capitalize() }}: <span id="val-{{ channel }}">90</span>°</label>
        <input type="range" min="0" max="180" value="90" 
               oninput="updateServo({{ channel }}, this.value)">
    </div>
    {% endfor %}

    <script>
        function updateServo(channel, angle) {
            document.getElementById('val-' + channel).innerText = angle;
            fetch(`/set_servo?channel=${channel}&angle=${angle}`);
        }
    </script>
</body>
</html>
"""

@app.route('/')
def index():
    return render_template_string(HTML_TEMPLATE, servos=servos)

@app.route('/set_servo')
def set_servo():
    channel = int(request.args.get('channel'))
    angle = int(request.args.get('angle'))
    
    # Safety Check & Execution
    if 0 <= angle <= 180:
        kit.servo[channel].angle = angle
        return f"Channel {channel} set to {angle}", 200
    return "Invalid Angle", 400

if __name__ == '__main__':
    # Run the server on your local network
    app.run(host='0.0.0.0', port=5000, debug=True)
