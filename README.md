# Emerging-Systems-Architecture-and-Technologies
#### Table of contents
* [Summary](#Summary)
* [Highlights](#Highlights)
* [Improvements](#Improvements)
* [Skills Gained](#Skills-Gained)
* [Coding Best Practices](#Coding_Best_Practices)
* [Thermostat prototype code](Thermostat.py)
* [Morse code translation code](Milestone3.py)

---

### Summary
For the final project in this course, I was to prototype a smart thermometer. The thermostat utilizes the [Python Statemachine](https://python-statemachine.readthedocs.io/en/latest/readme.html#contributing) 2.5.0 library to function as a state machine with three states: Off, Heat, and Cool. A single button cycles through states (Off → Heat → Cool → Off) via the cycle event. It handles temperature control by integrating a button for temperature increment by 1°F and another button for temperature decrement by 1°F. These buttons utilize the [gpiozero](https://gpiozero.readthedocs.io/en/latest/) Python library for seamless interrupt-driven callbacks. The [LCD is integrated via I2C](https://docs.circuitpython.org/projects/charlcd/en/latest/) protocol and placed on a virtual board along with the [AHT20](https://docs.circuitpython.org/projects/ahtx0/en/latest/) temperature sensor. The LEDs represent signals sent to the HVAC system. Each LED uses a pulse-width-modulated object for precise power and frequency control via the gpiozero library. The state machine is configured to flash the blue LED when the temperature is above the target temperature to indicate the system is cooling. The blue LED is solid when the temperature is at or below the target temperature, meaning the system is in standby. The opposite is true for the red LED to indicate whether the system is actively heating or in standby heat. State, temperature, and setpoint data are transmitted every 30 seconds to the simulated server via UART in the prototype. This is received with another program on the Raspberry Pi in another terminal. The thermostat is prototyped on a Raspberry Pi 4 with a breadboard for solderless connections to integrate the peripherals. 

For Milestone 3, I prototyped a Morse code signaling system on a Raspberry Pi 4. It utilizes the Python Statemachine library to manage states for dots (red LED, 500ms), dashes (blue LED, 1500ms), and pauses (250ms dot/dash, 750ms letter, 3000ms word). A single button toggles between "SOS" and a custom message via the processButton event. The LCD is integrated via the I2C protocol to display the active message. Red/blue LEDs use gpiozero for precise on/off control during transmissions. A Morse dictionary converts characters to code, with threading enabling continuous transmission and LCD updates without blocking. The system runs indefinitely until CTRL-C, with graceful cleanup.

---

### Highlights
The [state machine logic](Thermostat.py) for the thermostat prototype is minimalistic, resulting in fewer required resources for an embedded system. The updateLights method is concise and accurately sets the LED lights to the proper indicator using minimal code and basic logic. I overcame several inconsistencies with the buttons. I noticed rapid pressing created a race condition for the getFahrenheit method between manageMyDisplay and the updateLights methods. Using a thread lock on the getFahrenheit allowed the processor to queue the method for sequential calls. I also noticed bounce on the button release since the signal from the button is pulled low when pressed. This was easily fixed by utilizing the bouncetime parameter of the [gpiozero](https://gpiozero.readthedocs.io/en/stable/api_input.html) library for the buttons.

In the [Morse code translator](Milestone3.py), the transmit method efficiently parses messages into words, letters, and Morse symbols using nested loops and a comprehensive MorseDict, ensuring accurate character-to-code conversion. It correctly sequences state transitions (doDot, doDash, doDDP, doLP, doWP) with precise timing via sleep() in entry actions, maintaining standard Morse cadence.

---

### Improvements
The Morse code state machine in Milestone3.py redundantly sends events twice (e.g., self.send("doDot") twice), which could be streamlined to single transitions. LED blinking via gpiozero's blink() in on_enter states may not integrate perfectly with state exits; using PWMLED.pulse() as in Thermostat.py would allow better cancellation on state change. Add error handling for sensor reads and serial writes to prevent crashes from I2C/UART failures.

---

### Skills Gained
I mastered state machine design for embedded control, including event-driven transitions and entry/exit actions. I gained proficiency in GPIOZero for buttons, PWMLEDs, and interrupts, Adafruit libraries for the AHT20 sensor, and I2C LCD. Threading with locks for concurrent operations and serial UART communication are useful tools I now feel comfortable with. I also added Morse code encoding and timed LED signaling to my skill set. Tools I now use: Python statemachine, gpiozero, adafruit_ahtx0, adafruit_character_lcd, and pyserial, strengthening my embedded Python toolkit.

---

### Coding Best Practices
I ensured maintainability with modular classes (e.g., ManagedDisplay, TemperatureMachine/CWMachine), and clear method names. I improved readability through detailed comments and consistent structure. I enhanced adaptability by parameterizing setpoints, messages, and timings, using threads for non-blocking display/serial tasks, and implementing graceful shutdown with end flags and cleanup.
