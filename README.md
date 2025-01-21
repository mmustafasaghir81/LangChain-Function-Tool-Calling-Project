# 📚 Project : LangChain Function/Tool Calling
# 💡 Step 1: Install Required Libraries
To start, install the required library for interacting with Vertex AI's generative models.


```bash
!pip install google-cloud-aiplatform
```

# 💡 Step 2: Configure API Client for Vertex AI
Import necessary modules and configure the API client with your Google Cloud project and API key

```bash
from google.colab import userdata
from google.cloud import aiplatform  # Vertex AI's library

# Configure the API client with your Google API Key
api_key = userdata.get("GOOGLE_API_KEY")
aiplatform.init(project="your-project-id", location="us-central1", api_endpoint="us-central1-aiplatform.googleapis.com")
```

# 💡 Step 3: Initialize the Generative AI Model
Set up the Google Generative AI API with your API key and define the necessary tools for fan control.

```bash
import google.generativeai as genai
from google.colab import userdata

# Configure your API key (ensure you've set it in userdata or manually here)
genai.configure(api_key=userdata.get("GOOGLE_API_KEY"))

# Define the fan control functions
def enable_fan():
    """Turn on the fan system."""
    print("FANBOT: Fan enabled.")

def set_fan_speed(speed: str):
    """Set the fan speed. Fan must be enabled for this to work."""
    print(f"FANBOT: Fan speed set to {speed}.")

def stop_fan():
    """Stop the fan system."""
    print("FANBOT: Fan turned off.")

# List of tools (functions)
fan_controls = [enable_fan, set_fan_speed, stop_fan]

# System instruction for the AI model
instruction = "You are a helpful fan system bot. You can turn the fan on and off, and you can set the speed. Do not perform any other tasks."

# Initialize the model (Ensure this is done correctly with a proper model ID)
model = genai.GenerativeModel(
    "models/gemini-1.5-pro", tools=fan_controls, system_instruction=instruction
)

# Start the chat session to get the 'chat' object
chat = model.start_chat()
```

# 💡 Step 4: Define Tool Configuration Helper Function
Create a helper function to define tool configurations based on the mode of function calling.

```bash
from google.cloud.aiplatform import gapic as aiplatform_gapic
from collections.abc import Iterable

def tool_config_from_mode(mode: str, fns: Iterable[str] = ()):
    """Create a tool config with the specified function calling mode."""
    return {
        "function_calling_config": {
            "mode": mode,
            "allowed_function_names": fns
        }
    }
```
# 💡 Step 5: Ask About Capabilities
Use the tool_config to restrict function calls and interact with the model.

```bash
# Tool config for mode 'none' (no function calls will be made)
tool_config = tool_config_from_mode("none")

# Send a message to the model to ask about its capabilities
response = chat.send_message(
    "Hello fan-bot, what can you do?", tool_config=tool_config
)

# Print the response text from the model
print(response.text)
```
# 💡 Step 6: Automate Function Calling
Enable automatic function calls and send a message to trigger an action.

```bash
tool_config = tool_config_from_mode("auto")

# Send a message that will trigger automatic function calls (like turning on the fan)
response = chat.send_message("Turn on the fan!", tool_config=tool_config)

# Print the response and verify the function was called
print(response.parts[0])

# Reset the chat history to avoid unnecessary function calls
chat.rewind()
```
# 💡 Step 7: Control Specific Functions
Allow specific functions to be called and send a message to test the functionality.

```bash
# List of available functions for this message
available_fns = ["set_fan_speed", "stop_fan"]

# Tool configuration to allow specific functions
tool_config = tool_config_from_mode("any", available_fns)

# Send a message to set the fan speed to high
response = chat.send_message("Set the fan speed to HIGH!", tool_config=tool_config)

# Print the response from the model
print(response.parts[0])
```

# 💡 Step 8: Retry on Rate Limiting
Implement a retry mechanism to handle rate limits gracefully.

```bash
import time

def send_message_with_retry(auto_chat, message, tool_config, max_retries=3):
    for i in range(max_retries):
        try:
            return auto_chat.send_message(message, tool_config=tool_config)
        except genai.TooManyRequests:
            if i < max_retries - 1:
                sleep_time = 2 ** i  # Exponential backoff
                print(f"Rate limit exceeded. Retrying in {sleep_time} seconds...")
                time.sleep(sleep_time)
            else:
                raise

# Retry sending the message
send_message_with_retry(chat, "It's awful dark in here...", tool_config)
```

# 💡 Step 9: Schedule Fan Shutdown
Create a function to automatically turn off the fan at 6:00 AM and schedule it in a background thread.

```bash
import datetime
import time

# Function to simulate turning off the fan
def stop_fan():
    print("FANBOT: The fan has been turned OFF.")

def turn_off_fan_in_morning():
    """Automatically turns off the fan at 6:00 AM."""
    while True:
        current_time = datetime.datetime.now()
        # Check if it's 6:00 AM
        if current_time.hour == 6 and current_time.minute == 0:
            stop_fan()  # Turn off the fan
            print("FANBOT: It's 6:00 AM, turning off the fan automatically.")
            time.sleep(60)  # Sleep for a minute to avoid repeating it multiple times in the same minute
        time.sleep(10)  # Check every 10 seconds if it's time to turn off the fan

# Run the function in a separate thread for continuous checking
import threading
threading.Thread(target=turn_off_fan_in_morning, daemon=True).start()
```

# 💡 Step 10: Verify the Shutdown Function
Run the shutdown function immediately to verify its behavior.

```bash
import datetime
import time

# Function to simulate turning off the fan
def stop_fan():
    print("FANBOT: The fan has been turned OFF.")

def turn_off_fan_in_morning():
    """Automatically turns off the fan at 6:00 AM."""
    current_time = datetime.datetime.now()
    print(f"Current Time: {current_time.strftime('%H:%M:%S')}")

    if current_time.hour == 6 and current_time.minute == 0:
        stop_fan()  # Turn off the fan
        print("FANBOT: It's 6:00 AM, turning off the fan automatically.")
    else:
        print("FANBOT: It's not 6:00 AM yet, checking again soon.")

# Run the function once to immediately get an output (no continuous checking)
turn_off_fan_in_morning()
```

# Result 


Current Time: 07:00:39
FANBOT: It's not 6:00 AM yet, checking again soon.

