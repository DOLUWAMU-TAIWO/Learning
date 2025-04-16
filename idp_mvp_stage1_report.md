# Internal Developer Platform (IDP) MVP - Parameter Flow and Remote Execution
_Generated on 2025-04-12 22:13:12_

---
## Objective
The goal of this phase of the IDP MVP was to create an automated, testable workflow that allows parameters to be passed from a frontend to a Flask backend and then processed by an Ansible playbook to write them into a file on a remote virtual machine. This validates the end-to-end flow of user inputs, orchestration, and remote execution.

---
## Workflow Overview
1. A simple HTML+JS frontend collects deployment parameters from the user.
2. The frontend sends a JSON payload to a Flask `/deploy` endpoint.
3. Flask parses the parameters and triggers an Ansible playbook using `subprocess.run()`.
4. Ansible connects to a remote VM (defined in an inventory file), creates a directory `~/IDP`, and writes a `Parameters.txt` file with the passed values.
5. The file is verified and the result is returned to Flask, which sends it back to the frontend.

---
## Initial Issues and Fixes
### 1. **SSH Authentication Failure**
- **Problem:** Ansible was unable to connect to the remote VM due to missing or inaccessible SSH key.
- **Fix:**
  - Ensured a valid SSH private key existed (`~/.ssh/id_rsa` or `~/key.pem`).
  - Verified that the correct `ansible_user` and `ansible_ssh_private_key_file` were set in the inventory file.
  - Used absolute paths for SSH keys in the inventory (no `~`).

### 2. **User Mismatch in Inventory**
- **Problem:** The inventory used `ansible_user=ubuntu`, but the VM was configured for a user named `doluwamua`.
- **Fix:** Updated the inventory entry for the VM to use the correct SSH user:
```ini
[app]
gcp-vm1 ansible_host=34.159.25.70 ansible_user=doluwamua ansible_ssh_private_key_file=/home/qorelabs01/key.pem
```

### 3. **Permission Denied on Target Directory**
- **Problem:** Ansible failed to create `/home/ubuntu/IDP` because it was running as `doluwamua`, not `ubuntu`.
- **Fix:**
  - Changed the target directory to `~/IDP` or used `ansible_env.HOME` to dynamically get the user's home path.
  - Enabled `gather_facts: true` to allow access to `ansible_env.HOME`.

### 4. **Environment Not Respected by Subprocess**
- **Problem:** Flask’s `subprocess.run()` didn’t inherit the right environment (e.g., SSH agent, HOME variable).
- **Fix:**
  - Updated the subprocess call to explicitly pass `env=os.environ.copy()` to inherit environment variables.

### 5. **Flask and Frontend Flow Verification**
- Confirmed the Flask endpoint correctly echoed received parameters.
- Integrated dynamic construction of `ansible-playbook` command with `-e key=value` from all received JSON keys.

---
## Final State of the System
- Parameters can be submitted from a frontend form or curl.
- Flask receives and prints them, then triggers Ansible.
- Ansible connects via SSH, creates `~/IDP`, writes `Parameters.txt`, and verifies its presence.
- Result of the Ansible playbook is returned to the frontend for confirmation.

---
## Next Steps (Proposed)
- Add logic in Ansible to handle multiple build tools (Maven, Gradle, NPM).
- Add Docker image build and push steps.
- Add container deployment to target VM.
- Create `/status` endpoint to query execution result or log.
- Integrate with Backstage for portal-based workflows.

---
## Conclusion
This phase successfully validated the infrastructure and communication layers of the IDP MVP — proving secure parameter intake, orchestration, and remote execution are fully functional.