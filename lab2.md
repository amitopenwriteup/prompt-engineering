# Comprehensive Hands-on Lab: Resilient AI-Driven Kubernetes Troubleshooting Agent

## Overview

In this lab, you will combine Python development with Prompt Engineering methodologies to build a resilient, AI-powered Kubernetes troubleshooting tool. You will configure a Linux virtual environment, secure credentials, implement exponential backoff retry logic to handle API throttling, and execute **Zero-Shot**, **Single-Shot**, and **Multi-Shot Chain-of-Thought (CoT)** prompts using the `google-genai` SDK.

---

## Lab Objectives

By the end of this lab, you will be able to:

1. Provision an isolated Python environment and manage dependencies on Linux.
2. Securely store and load Gemini API credentials using `.env` files.
3. Build a Python script that gracefully handles API capacity errors (`503 UNAVAILABLE`).
4. Programmatically apply Zero, Single, and Multi-Shot CoT prompt techniques to generate, standardize, and diagnose Kubernetes workloads (`Deployments`, `StatefulSets`, `CrashLoopBackOff` errors).

---

## Prerequisites

* Linux system (Ubuntu/Debian or RHEL/Fedora).
* Python 3.10+ installed (`python3 --version`).
* Gemini API key from Google AI Studio.

---

1. **Prepare Directory & Virtual Environment:** Isolate dependencies on the host system.
Open your Linux terminal and install necessary package manager dependencies:

```bash
sudo apt update && sudo apt install -y python3-venv python3-pip

```

Create a project directory and set up a virtual environment named `venv`:

```bash
mkdir -p ~/k8s_ai_lab && cd ~/k8s_ai_lab
python3 -m venv venv
source venv/bin/activate
pip install --upgrade pip

```


2. **Install SDK & Secure Credentials:** Set up environment variables and file permissions.
Install the official Google GenAI SDK and `python-dotenv`:

```bash
pip install google-genai python-dotenv

```

Store your API key securely inside a `.env` file and restrict its read permissions:

```bash
cat << 'EOF' > .env
GEMINI_API_KEY="your_actual_api_key_here"
EOF

chmod 600 .env

```

Create a `.gitignore` file to protect secret keys and local dependencies:

```bash
cat << 'EOF' > .gitignore
venv/
.env
__pycache__/
EOF

```


3. **Build the Prompt Engineering Engine:** Implement retry logic and N-Shot CoT prompting.
Create a script named `k8s_troubleshooter.py`. This script handles `503 UNAVAILABLE` capacity spikes using exponential backoff and executes **Zero-Shot**, **Single-Shot**, and **Multi-Shot Chain-of-Thought** logic.

```bash
cat << 'EOF' > k8s_troubleshooter.py
import time
from dotenv import load_dotenv
from google import genai
from google.genai.errors import ServerError

# Load environment variables
load_dotenv()

# Initialize Gemini Client
client = genai.Client()
MODEL_NAME = "gemini-3.8-flash"

def call_gemini_with_retry(prompt: str, max_retries: int = 3) -> str:
    """Executes a prompt against Gemini API with backoff logic for 503 errors."""
    for attempt in range(1, max_retries + 1):
        try:
            response = client.models.generate_content(
                model=MODEL_NAME,
                contents=prompt,
            )
            return response.text
        except ServerError as e:
            if "503" in str(e):
                wait_time = attempt * 3
                print(f"[Warning] 503 Capacity Limit Hit. Retrying in {wait_time}s... (Attempt {attempt}/{max_retries})")
                time.sleep(wait_time)
            else:
                raise e
    raise RuntimeError("Failed to obtain response from Gemini API after retries.")

# --- PROMPT TEMPLATES ---

ZERO_SHOT_PROMPT = """
Task: Write a Kubernetes Deployment YAML for an Nginx web server.
Requirements:
- Replicas: 3
- Resource limits: 250m CPU, 128Mi Memory
- Container Port: 80
"""

SINGLE_SHOT_PROMPT = """
Convert the following application requirements into a Kubernetes Pod definition following company standards.

--- EXAMPLE START ---
Input: App name 'auth-service', image 'myregistry/auth:v1', port 8080.
Output:
apiVersion: v1
kind: Pod
metadata:
  name: auth-service
  labels:
    app.kubernetes.io/name: auth-service
    app.kubernetes.io/managed-by: platform-team
spec:
  securityContext:
    runAsNonRoot: true
    runAsUser: 10001
  containers:
  - name: auth-service
    image: myregistry/auth:v1
    ports:
    - containerPort: 8080
--- EXAMPLE END ---

Task:
Input: App name 'payment-gateway', image 'internal-repo/payment:v2.4', port 9090.
Output:
"""

MULTI_SHOT_COT_PROMPT = """
You are a Senior Kubernetes Site Reliability Engineer (SRE). Analyze the provided Pod details and log outputs. 
First, perform step-by-step reasoning (Chain-of-Thought) to identify the root cause, and then provide the resolution.

--- EXAMPLE 1 ---
Input:
Pod Name: payment-api-7d9b4f6-x2z9l
Status: CrashLoopBackOff
Last Logs: "Error: Secret 'db-credentials' not found in namespace 'prod'"

Reasoning (CoT):
1. The Pod status is CrashLoopBackOff, meaning the container repeatedly starts, fails, and restarts.
2. The log explicitly shows a missing dependency error: Secret 'db-credentials' does not exist in namespace 'prod'.
3. The application process fails immediately upon initialization because it cannot bind environment variables.
4. Resolution requires either creating the missing Secret or updating the Pod spec to reference the correct existing Secret.

Resolution:
Execute the following to create the missing secret:
kubectl create secret generic db-credentials --from-literal=username=admin --from-literal=password=secret123 -n prod

--- EXAMPLE 2 ---
Input:
Pod Name: auth-service-589f8c6-a1b2c
Status: CrashLoopBackOff
Exit Code: 137
Events: "OOMKilled: Kill process 1042 (node) score 980 or sacrifice child"

Reasoning (CoT):
1. Exit Code 137 combined with 'OOMKilled' indicates that the Linux kernel OOM killer terminated the container process.
2. The container attempted to consume more memory than allowed by its configured resource limit.
3. We need to increase the container's memory limits in its Deployment specification.

Resolution:
kubectl patch deployment auth-service -p '{"spec":{"template":{"spec":{"containers":[{"name":"auth-service","resources":{"limits":{"memory":"512Mi"}}}]}}}}'

--- REAL TASK ---
Input:
Pod Name: web-frontend-847c9d5-p9q8r
Status: CrashLoopBackOff
Exit Code: 1
Last Logs: "Configuration file /etc/nginx/conf.d/default.conf not readable: Permission denied"
Pod Spec snippet:
  securityContext:
    runAsUser: 2000
    runAsGroup: 3000
  volumeMounts:
  - name: config-volume
    mountPath: /etc/nginx/conf.d

Reasoning (CoT):
"""

if __name__ == "__main__":
    print("=== 1. Executing Zero-Shot Prompt ===")
    print(call_gemini_with_retry(ZERO_SHOT_PROMPT))
    
    print("\n=== 2. Executing Single-Shot Prompt ===")
    print(call_gemini_with_retry(SINGLE_SHOT_PROMPT))
    
    print("\n=== 3. Executing Multi-Shot Chain-of-Thought (CoT) Prompt ===")
    print(call_gemini_with_retry(MULTI_SHOT_COT_PROMPT))
EOF

```


4. **Execute Script & Validate Diagnostic Output:** Run script within virtual environment.
Execute the completed script in your Linux terminal:

```bash
python3 k8s_troubleshooter.py

```

Check the terminal output to verify each prompting technique:

1. **Zero-Shot Output:** Generates a standard Nginx Deployment manifest.
2. **Single-Shot Output:** Applies `securityContext` (`runAsNonRoot: true`, `runAsUser: 10001`) and standardized annotations to `payment-gateway`.
3. **Multi-Shot CoT Output:** Breaks down the `Permission denied` error, identifies that volume mount permissions conflict with `runAsUser: 2000`, and suggests patching the deployment using `fsGroup: 3000`.


---

## Troubleshooting Guide

| Symptom | Cause | Resolution |
| --- | --- | --- |
| `ModuleNotFoundError` | Virtual environment inactive or package missing. | Run `source venv/bin/activate` followed by `pip install google-genai python-dotenv`. |
| `404 NOT_FOUND` | Incorrect or legacy model string specified. | Ensure the code references `gemini-3.8-flash`. |
| `503 UNAVAILABLE` | Temporary Google API server capacity bottleneck. | The built-in `call_gemini_with_retry` function handles retries automatically using exponential backoff. |
