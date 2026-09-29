Here is the complete, self-contained hands-on lab guide formatted in Markdown for Linux environments.

---

# Hands-on Lab: Setting Up Google GenAI SDK with Virtual Environments & Resilient API Execution on Linux

## Overview

In this lab, you will set up a clean Python virtual environment on Linux, configure secure API credential storage using environment variables, and write a resilient Python script to interact with the Google GenAI SDK.

---

## Lab Objectives

By the end of this lab, you will be able to:

1. Create and manage an isolated Python virtual environment on Linux.
2. Securely store and load API credentials using `.env` files.
3. Install and manage dependencies (`google-genai`, `python-dotenv`).
4. Implement retry logic handling temporary `503 UNAVAILABLE` capacity spikes from model endpoints.

---

## Prerequisites

* Linux system (Ubuntu/Debian, RHEL/CentOS, or similar distribution).
* Python 3.10+ installed (`python3 --version`).
* Active Gemini API Key from Google AI Studio.

---

## Step 1: System Package Verification & Directory Setup

1. Open your Linux terminal.
2. Ensure `python3-venv` is installed on your system:
```bash
sudo apt update && sudo apt install -y python3-venv python3-pip

```


*(For RHEL/Fedora-based systems, use `sudo dnf install python3-pip`)*
3. Create a dedicated directory for your project and navigate into it:
```bash
mkdir -p ~/gemini_lab && cd ~/gemini_lab

```



---

## Step 2: Virtual Environment Configuration

1. Create a Python virtual environment named `venv`:
```bash
python3 -m venv venv

```


2. Activate the virtual environment:
```bash
source venv/bin/activate

```


*(Your terminal prompt should now be prefixed with `(venv)`)*
3. Upgrade `pip` to the latest version inside the active environment:
```bash
pip install --upgrade pip

```



---

## Step 3: Install Required Dependencies

Install the official `google-genai` SDK alongside `python-dotenv` for local environment variable management:

```bash
pip install google-genai python-dotenv

```

Verify the installation:

```bash
pip list

```

---

## Step 4: Secure API Key Storage

1. Create a `.env` file in the project root:
```bash
cat << 'EOF' > .env
GEMINI_API_KEY="your_actual_api_key_here"
EOF

```


*(Replace `your_actual_api_key_here` with your actual API key)*
2. Restrict file permissions so only your Linux user can read the key:
```bash
chmod 600 .env

```


3. Create a `.gitignore` file to ensure secrets are never committed if using version control:
```bash
cat << 'EOF' > .gitignore
venv/
.env
__pycache__/
EOF

```



---

## Step 5: Create the Resilient Test Script

Create a Python script named `test_gemini.py` that includes exponential backoff retry logic to gracefully handle `503 UNAVAILABLE` capacity errors:

```bash
cat << 'EOF' > test_gemini.py
import time
from dotenv import load_dotenv
from google import genai
from google.genai.errors import ServerError

# Load environment variables from .env file
load_dotenv()

# Initialize the Gemini client
client = genai.Client()

MODEL_NAME = "gemini-3.8-flash"
PROMPT = "Confirm that my Linux Python setup with the google-genai SDK is working."

def execute_prompt_with_retry(model: str, prompt_text: str, max_retries: int = 3):
    print(f"Connecting to endpoint using model: {model}...")
    
    for attempt in range(1, max_retries + 1):
        try:
            response = client.models.generate_content(
                model=model,
                contents=prompt_text,
            )
            print("\n--- Response Received Successfully ---")
            print(response.text)
            return True
            
        except ServerError as e:
            if "503" in str(e):
                wait_time = attempt * 3
                print(f"[Warning] Server busy (503). Retrying in {wait_time}s... (Attempt {attempt}/{max_retries})")
                time.sleep(wait_time)
            else:
                print(f"[Error] Unexpected Server Error: {e}")
                raise e
        except Exception as e:
            print(f"[Error] Execution failed: {e}")
            raise e

    print("\n[Failure] Unable to complete request due to high server demand.")
    return False

if __name__ == "__main__":
    execute_prompt_with_retry(MODEL_NAME, PROMPT)
EOF

```

---

## Step 6: Execute and Verify

Run the script inside your activated virtual environment:

```bash
python3 test_gemini.py

```

### Expected Output

Upon successful execution, you should see output similar to the following:

```text
Connecting to endpoint using model: gemini-3.8-flash...

--- Response Received Successfully ---
Your Linux Python setup with the google-genai SDK is configured and working properly!

```

---

## Troubleshooting Common Issues

| Issue / Error | Root Cause | Solution |
| --- | --- | --- |
| `ModuleNotFoundError: No module named 'dotenv'` | Virtual environment not activated or package missing. | Run `source venv/bin/activate` and reinstall with `pip install python-dotenv`. |
| `404 NOT_FOUND` | Deprecated or invalid model identifier used in request. | Ensure model string is set to `"gemini-3.8-flash"`. |
| `400 INVALID_ARGUMENT` | Missing or malformed `GEMINI_API_KEY`. | Verify `.env` file formatting and ensure `load_dotenv()` is called prior to `genai.Client()`. |
| `503 UNAVAILABLE` | Temporary Google API server capacity spike. | The built-in retry loop in `test_gemini.py` will handle this automatically. |
