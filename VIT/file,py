import requests
import json
import time
import random  # For simulating sensor noise/errors and comms

# --- Configuration ---
API_BASE = "https://roverdata2-production.up.railway.app/api"
ROVER_SPEED = 5
OBSTACLE_THRESHOLD = 0.2  # Adjust based on sensor readings
RECHARGE_START = 0.05
RECHARGE_END = 0.8
COMM_LOSS_THRESHOLD = 0.1
COMM_REGAIN_THRESHOLD = 0.1
INTERMITTENT_COMM_DELAY = 2  # seconds (Simulate comms delay)
SENSOR_NOISE_LEVEL = 0.05  # Percentage of noise to add to sensor readings

# --- Helper Functions ---
def simulate_sensor_noise(value):
    """Adds random noise to a sensor reading."""

    noise = random.uniform(-SENSOR_NOISE_LEVEL, SENSOR_NOISE_LEVEL) * value
    return value + noise

def simulate_comm_delay():
    """Simulates intermittent communication delays."""

    time.sleep(random.uniform(0, INTERMITTENT_COMM_DELAY))

def check_battery_status(status):
    """Checks battery level and returns True if recharging is needed."""

    battery_level = status.get("battery_level", 1.0)  # Default to 1.0 if not available
    if battery_level <= RECHARGE_START:
        print("Battery low, initiating recharge.")
        return True
    return False

def is_communication_lost(status):
    """Checks battery level and returns True if communication is lost."""

    battery_level = status.get("battery_level", 1.0)
    if battery_level < COMM_LOSS_THRESHOLD:
        print("Communication lost due to low battery.")
        return True
    return False

def can_communicate(status):
    """Checks battery level and returns True if communication is regained."""

    battery_level = status.get("battery_level", 1.0)
    if battery_level > COMM_REGAIN_THRESHOLD:
        print("Communication regained.")
        return True
    return False

# --- API Interaction Functions ---
def start_session():
    """Starts a new session and returns the session ID."""

    try:
        response = requests.post(f"{API_BASE}/session/start")
        response.raise_for_status()
        data = response.json()
        session_id = data.get("session_id")
        if session_id:
            print(f"Session started! Your session ID: {session_id}")
            return session_id
        else:
            print("Error: Could not retrieve session ID.")
            return None
    except requests.exceptions.RequestException as e:
        print(f"Error starting session: {e}")
        return None

def move_rover(session_id, direction):
    """Moves the rover in the specified direction."""

    if direction not in ["forward", "backward", "left", "right"]:
        print("Error: Invalid direction.")
        return False

    try:
        response = requests.post(
            f"{API_BASE}/rover/move?session_id={session_id}&direction={direction}"
        )
        response.raise_for_status()
        print(f"Rover moved {direction}.")
        return True
    except requests.exceptions.RequestException as e:
        print(f"Error moving rover: {e}")
        return False

def stop_rover(session_id):
    """Stops the rover's movement."""

    try:
        response = requests.post(f"{API_BASE}/rover/stop?session_id={session_id}")
        response.raise_for_status()
        print("Rover stopped.")
        return True
    except requests.exceptions.RequestException as e:
        print(f"Error stopping rover: {e}")
        return False

def get_rover_status(session_id):
    """Gets the current status of the rover."""

    try:
        response = requests.get(f"{API_BASE}/rover/status?session_id={session_id}")
        response.raise_for_status()
        data = response.json()
        return data
    except requests.exceptions.RequestException as e:
        print(f"Error getting rover status: {e}")
        return None

def get_sensor_data(session_id):
    """Fetches the latest sensor data from the rover."""

    try:
        response = requests.get(f"{API_BASE}/rover/sensor-data?session_id={session_id}")
        response.raise_for_status()
        data = response.json()
        return data
    except requests.exceptions.RequestException as e:
        print(f"Error getting sensor data: {e}")
        return None

# --- Navigation and Decision-Making (To be completed by you) ---
def navigate(session_id, sensor_data):
    """
    Implements the rover's navigation logic.

    This is where you'll use sensor data to decide which way to move.
    """

    #  Example (very basic obstacle avoidance):
    front_obstacle = False
    #  Replace with actual sensor data analysis
    #  if sensor_data and sensor_data.get("ultrasonic_front") < OBSTACLE_THRESHOLD:
    #      front_obstacle = True
    if front_obstacle:
        stop_rover(session_id)
        #  Implement more sophisticated avoidance (e.g., turn)
        turn_direction = random.choice(["left", "right"])
        move_rover(session_id, turn_direction)
        time.sleep(1)  # Turn for 1 second
        move_rover(session_id, "forward")  # Try moving forward again
    else:
        move_rover(session_id, "forward")

def identify_survivors(sensor_data):
    """
    Analyzes sensor data to detect and identify survivors.

    This is a complex task that might involve:
    -  Thresholding sensor values
    -  Pattern recognition
    -  Data fusion from multiple sensors
    """

    #  Placeholder - Replace with your survivor detection logic
    if sensor_data:
        #  Example: Check for a specific RFID tag or IR pattern
        #  if sensor_data.get("rfid_tag") == "SurvivorTag123":
        #      print("Survivor detected!")
        pass  # Implement your logic here

def deliver_aid(session_id):
    """
    Simulates the rover delivering aid to a survivor.

    This might involve a sequence of pre-programmed movements
    or more complex navigation to a specific location.
    """

    #  Placeholder - Replace with your aid delivery routine
    print("Simulating aid delivery.")
    time.sleep(5)  # Simulate time taken to deliver aid
    move_rover(session_id, "backward")  # Example: Move backward after
    time.sleep(2)

# --- Main Execution ---
if __name__ == "__main__":
    session_id = start_session()
    if not session_id:
        exit()

    communication_active = True  # Initially assume comms are active
    try:
        while True:  # Main control loop
            #  Simulate intermittent communication
            if communication_active:
                simulate_comm_delay()  # Introduce random delays

                #  Get rover status (including battery level)
                rover_status = get_rover_status(session_id)
                if rover_status:
                    #  Handle power constraints
                    if check_battery_status(rover_status):
                        stop_rover(session_id)
                        print("Recharging...")
                        while check_battery_status(rover_status):  # Wait for recharge
                            time.sleep(5)  # Check battery every 5 seconds
                            rover_status = get_rover_status(session_id)
                            if rover_status is None:
                                communication_active = False
                                break  # Exit recharge loop if comms lost
                        if communication_active:
                            move_rover(session_id, "forward")  # Resume movement
                    #  Check for communication loss
                    if is_communication_lost(rover_status):
                        communication_active = False
                else:
                    communication_active = False  # Assume comms lost if status fails
            else:  # Communication is lost
                print("Attempting to regain communication...")
                time.sleep(10)  # Wait before retrying
                rover_status = get_rover_status(session_id)  # Try to get status
                if rover_status and can_communicate(rover_status):
                    communication_active = True
                else:
                    #  Implement basic "limp home" behavior if completely lost
                    #  This would be very basic movement without API calls
                    #  For example, just "move_rover(None, "forward")"
                    #  But you'd need a way to stop it eventually
                    pass  # Replace with your "limp home"
            if communication_active:
                #  Get sensor data
                sensor_data = get_sensor_data(session_id)
                if sensor_data:
                    #  Simulate sensor noise
                    #  for key in sensor_data:
                    #      if isinstance(sensor_data[key], (int, float)):
                    #          sensor_data[key] = simulate_sensor_noise(sensor_data[key])
                    #  Navigation and survivor detection
                    navigate(session_id, sensor_data)
                    identify_survivors(sensor_data)
                    #  Placeholder: Trigger aid delivery based on survivor detection
                    #  if survivor_detected:
                    #      deliver_aid(session_id)
                else:
                    print("Failed to get sensor data.")
                    #  Handle the case where sensor data is not available
            else:
                print("No communication with the rover.")
    except KeyboardInterrupt:
        print("Program interrupted. Stopping rover...")
        if communication_active:
            stop_rover(session_id)
    finally:
        print("Exiting program.")
def navigate(session_id, sensor_data):
    front_distance = sensor_data.get("ultrasonic_front")
    if front_distance is not None and front_distance < OBSTACLE_THRESHOLD:  # OBSTACLE_THRESHOLD is a value you define
        stop_rover(session_id)
        turn_direction = random.choice(["left", "right"])  # Choose a random direction to turn
        move_rover(session_id, turn_direction)
        time.sleep(1)  # Turn for 1 second (adjust as needed)
        move_rover(session_id, "forward")  # Continue moving forward
    else:
        move_rover(session_id, "forward")  # If no obstacle, keep moving forward
def identify_survivors(sensor_data):
    rfid_tag = sensor_data.get("rfid_tag")
    if rfid_tag == "SurvivorTag123":  # Replace "SurvivorTag123" with the actual tag ID
        print("Survivor detected! (RFID)")
        return True
    return False
def deliver_aid(session_id):
    stop_rover(session_id)
    print("Aid delivered!")
    time.sleep(5)  # Simulate time taken to deliver aid
    move_rover(session_id, "backward")  # Move away from the survivor