# Dynamic Dependency Installation and Validation - Internal Developer Platform (IDP)

## Objective
Implement a dynamic and modular system for installing essential system dependencies and developer-specific build tools during provisioning using Ansible, with support for validation and real-time feedback. This step forms a foundational part of building an Internal Developer Platform (IDP) that is configurable via the frontend and scalable.

---

## Key Features Implemented

- **Essential Dependencies Installation**
  - Packages: `git`, `curl`, `unzip`, `docker.io`, `docker-compose`
  - Always installed on all target nodes.

- **Dynamic Build Tool Selection**
  - Build tools such as `maven`, `nodejs`, and `npm` are passed as an extra variable (`build_tools`) via CLI or Vagrantfile.
  - Only the tools passed are installed and validated.

- **Validation of Installed Tools**
  - Version checks are done using proper commands (e.g., `mvn -v`, `docker --version`, etc.).
  - Errors are ignored to allow the playbook to proceed while still reporting missing tools.

- **User Docker Permissions**
  - The playbook fetches the current SSH user and adds them to the `docker` group.
  - SSH connection is reset using `meta: reset_connection` to apply group membership changes.

- **Docker Test Run**
  - Runs `docker run hello-world` to confirm Docker is working correctly.

---

## File Structure

- `deploy_pipeline.yml`: Main playbook for installing and validating dependencies.
- `validate_dependency.yml`: External task file included to handle looped validation logic (avoids Ansible's block+loop limitation).

---

## Notable Errors & Fixes

- **Loop with Block Error**:
  - Error: `'loop' is not a valid attribute for a Block`
  - Fix: Moved the version validation block into an external task file and used `include_tasks` with loop.

- **Wrong Executable Names**:
  - `nodejs` is the actual executable name on Ubuntu, not `node`.
  - Fix: Corrected version commands in `build_tool_commands` map.

- **Invalid Tool Validation**:
  - If a tool like `nodejs` or `npm` wasn't selected for install but included in validation, it caused a "command not found" error.
  - Fix: Filtered validation map using `selectattr('key','in', build_tools)` to validate only selected tools.

---

## How to Pass Build Tools

### Via CLI
```bash
ansible-playbook -i inventory deploy_pipeline.yml -e "build_tools=['maven']"
```

### Via Vagrantfile
```ruby
ansible.extra_vars = {
  build_tools: ['maven']
}
```

---

## Next Step

Now that the backend logic for dynamic dependency installation is confirmed working, the next step is to:
- Update the Flask Python API to accept build tool selection from the frontend.
- Allow users to dynamically trigger provisioning based on their selection.

---