
# Internal Developer Platform (IDP) – Parameter Capture MVP

## Overview

This document describes the step-by-step implementation of an Internal Developer Platform MVP that captures deployment parameters from a browser-based form and sends them to a Python Flask server through an Nginx reverse proxy. The server then serializes and returns the input. This setup is a foundational part of a larger IDP architecture and was built as a testable frontend-to-backend workflow.

---

## Architecture Overview

- **Frontend**: A static HTML form served from `/idp/` allowing users to input deployment details.
- **Backend**: A Python Flask server listening on port `5000`, capturing JSON data via a `/deploy` endpoint.
- **Reverse Proxy**: Nginx routes requests to the backend Flask server when `/deploy` is hit.
- **Host**: All services are deployed on the same server (e.g., `89.247.214.203`).

---

## Step-by-Step Implementation

### 1. Frontend HTML Form (`index.html`)

- Collects user input for:
  - Git repo URL
  - Application name
  - Application/container ports
  - Build tool (maven, gradle, node/npm)
- Sends a JSON payload via `fetch('/deploy')`.
- Displays response using `<pre>` formatted JSON inside a `<div id="output">`.

**Problem Solved**:
✅ Displaying backend responses directly in the browser.  
✅ Making relative API calls to avoid hardcoding IPs or ports.

---

### 2. Flask Backend (`idp_api.py`)

- Accepts POST requests on `/deploy`.
- Parses incoming JSON.
- Returns JSON with keys: `status`, `stdout`, and `stderr`.

```python
@app.route('/deploy', methods=['POST'])
def deploy():
    try:
        data = request.get_json(force=True)
    except Exception as e:
        return jsonify({
            "status": "failed",
            "stderr": f"Error parsing JSON: {str(e)}",
            "stdout": ""
        }), 400

    print("Received parameters:", data)

    return jsonify({
        "status": "success",
        "stdout": f"Received parameters: {data}",
        "stderr": ""
    }), 200
```

**Problem Solved**:
✅ Verifying and echoing back the data without Ansible yet.  
✅ Testing full frontend-to-backend connectivity.

---

### 3. Nginx Reverse Proxy Configuration

- `/idp/` serves static HTML + CSS (protected with HTTP Basic Auth).
- `/deploy` proxies to Flask server at `127.0.0.1:5000`.

**Relevant Nginx Block:**

```nginx
location /deploy {
    proxy_pass http://127.0.0.1:5000/deploy;
    proxy_http_version 1.1;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
}
```

**Problem Solved**:
✅ Prevented CORS and port mismatch issues.  
✅ Allowed browser + curl to hit Flask securely from the same base URL.

---

## Testing & Troubleshooting

- Initial 404 errors → resolved by adding `/deploy` endpoint.
- `ERR_CONNECTION_REFUSED` → resolved by configuring Nginx reverse proxy.
- `playbook not found` → fixed by skipping Ansible until integration is ready.
- Output properly rendered using `<pre>` and `JSON.stringify(result, null, 2)` in JS.

---

## Next Steps

- Begin integrating Ansible playbook execution in the backend.
- Dynamically trigger deployment automation with user-provided parameters.
- Add validation, feedback UI, and logs in front end.

---
