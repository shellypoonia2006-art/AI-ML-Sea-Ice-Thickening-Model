# AI-ML-Sea-Ice-Thickening-Model

import random
import time
import logging
from datetime import datetime

# Configure logging to display system status
logging.basicConfig(level=logging.INFO, format='%(asctime)s - %(levelname)s - %(message)s')

class SensingAgent:
    """
    Simulates the collection of raw environmental data from hardware sensors.
    In a real deployment, this would interface with GPIO pins on a microcontroller.
    """
    def __init__(self):
        # Simulation ranges for Arctic winter conditions
        self.temp_range = (-35.0, -10.0)  # Celsius
        self.wind_range = (0.0, 15.0)     # m/s
        self.salinity_range = (28.0, 35.0)# PSU (Practical Salinity Unit)
        self.solar_range = (0.0, 200.0)   # W/m^2 (0 = dark winter, >100 = spring)

    def get_state_vector(self):
        """Generates a dictionary representing the current sensor readings."""
        state = {
            "timestamp": datetime.now().isoformat(),
            "air_temp_c": round(random.uniform(*self.temp_range), 2),
            "wind_speed_ms": round(random.uniform(*self.wind_range), 2),
            "water_salinity_psu": round(random.uniform(*self.salinity_range), 2),
            "solar_radiation": round(random.uniform(*self.solar_range), 2),
            "ice_thickness_cm": round(random.uniform(50.0, 150.0), 2)
        }
        return state

class DecisionAgent:
    """
    The 'Brain' of the system. Contains the logic gates and optimization math.
    """
    def __init__(self):
        # Thresholds defined in the Feasibility Analysis
        self.MIN_WIND_SPEED = 5.0       # m/s (Required for wind power)
        self.MAX_OP_TEMP = -20.0        # Celsius (Must be colder than this)
        self.SOLAR_CUTOFF = 100.0       # W/m^2 (Ecological safeguard threshold)
        
        # Calibration weights for the Flow Rate Formula (Simulating ML weights)
        self.weight_temp = 2.5
        self.weight_salinity = 1.8

    def evaluate(self, state):
        """
        Input: State Vector from SensingAgent
        Output: Decision Dictionary (Action, FlowRate, Reason)
        """
        
        # --- GATE 1: Ecological Guardian (Spring Switch-off) ---
        if state["solar_radiation"] > self.SOLAR_CUTOFF:
            return {
                "action": "STOP",
                "flow_rate": 0,
                "reason": "CRITICAL: High Solar Radiation. Spring Bloom Protection Protocol active."
            }

        # --- GATE 2: Meteorological Viability (Energy & Thermodynamics) ---
        if state["wind_speed_ms"] < self.MIN_WIND_SPEED:
            return {
                "action": "SLEEP",
                "flow_rate": 0,
                "reason": f"Wind too low ({state['wind_speed_ms']} m/s). Cannot power pumps."
            }
        
        if state["air_temp_c"] > self.MAX_OP_TEMP:
            return {
                "action": "SLEEP",
                "flow_rate": 0,
                "reason": f"Too warm ({state['air_temp_c']} C). Risk of slush formation."
            }

        # --- GATE 3: Optimization Engine (The 'Salty Slush' Equation) ---
        # Calculate optimal flow rate to ensure brine drainage
        # Logic: Colder temps allow faster pumping; Higher salinity requires slower pumping.
        
        # Base flow calculation (Arbitrary units for simulation)
        temp_factor = abs(state["air_temp_c"]) * self.weight_temp
        salinity_penalty = state["water_salinity_psu"] * self.weight_salinity
        
        optimal_flow = temp_factor - salinity_penalty
        
        # Clamp flow rate to hardware limits (0 to 100 liters/min)
        optimal_flow = max(10, min(100, optimal_flow))

        return {
            "action": "PUMP",
            "flow_rate": round(optimal_flow, 1),
            "reason": "Optimal freezing conditions detected."
        }

class ExecutionAgent:
    """
    Simulates the physical hardware control (Valves, Clutches).
    """
    def execute(self, decision):
        if decision["action"] == "PUMP":
            logging.info(f">>> ACTUATOR: Engaging Clutch. Opening Valve to {decision['flow_rate']} L/min.")
            logging.info(f"    STATUS: Pumping... (Reason: {decision['reason']})")
        elif decision["action"] == "STOP":
            logging.warning(f">>> ACTUATOR: EMERGENCY SHUTDOWN. {decision['reason']}")
        else:
            logging.info(f">>> ACTUATOR: System Idle. {decision['reason']}")

class ArcticIceSystem:
    def __init__(self):
        self.sensor = SensingAgent()
        self.brain = DecisionAgent()
        self.actuator = ExecutionAgent()

    def run_cycle(self):
        """Runs one full cycle of the control loop."""
        print("-" * 60)
        # 1. Sense
        current_state = self.sensor.get_state_vector()
        logging.info(f"SENSOR DATA: Temp={current_state['air_temp_c']}C, "
                                        f"Solar={current_state['solar_radiation']}W/m2")

        # 2. Decide
        decision = self.brain.evaluate(current_state)

        # 3. Execute
        self.actuator.execute(decision)

# --- Main Simulation Loop ---
if __name__ == "__main__":
    print("Initializing Arctic Geoengineering AI System...")
    print("Target Zone: Beaufort Gyre")
    print("Press Ctrl+C to stop simulation.\n")
    
    system = ArcticIceSystem()

    try:
        # Simulate 10 cycles of operation
        for i in range(10):
            system.run_cycle()
            time.sleep(2) # Speed up simulation (real interval would be 10 mins)
    except KeyboardInterrupt:
        print("\nSimulation stopped by user.")        
