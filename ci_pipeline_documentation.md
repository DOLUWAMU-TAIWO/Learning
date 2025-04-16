
# CI Pipeline Implementation Documentation

## Overview

This document describes the CI pipeline implementation we developed, including the errors encountered and the improvements made. The pipeline performs the following operations:

1. **Clone or update a Git repository.**
2. **Extract the commit hash** (used for tagging the Docker image).
3. **Run a Maven build** locally using Java 17.
4. **Log build output** and verify the generated artifact.
5. **Build a Docker image** using the repository as the build context.
6. **Push the Docker image** to Docker Hub.
7. **Update build metadata** and clean up the repository.

Our Python Flask application (`p_test.py`) triggers this playbook via a `/deploy` endpoint, passing along parameters such as the repository URL, application name, and build tool. Initially, there was dynamic behavior to handle multiple build tools, but the final implementation was simplified to focus on Maven.

---

## Errors Encountered

### 1. Recursive Template Error on `commit_hash`

- **Error Message:**  
  `recursive loop detected in template string: {{ commit_hash | default('latest') }}. maximum recursion depth exceeded`

- **Cause:**  
  The playbook initially attempted to default the variable by referring to itself.

- **Resolution:**  
  We removed the self-referencing default and properly set the default values.

---

### 2. Undefined Variable for Docker Hub Credentials

- **Error Message:**  
  `The task includes an option with an undefined variable. The error was: 'dockerhub_username' is undefined.`

- **Cause:**  
  The secrets file was not correctly passed or set.

- **Resolution:**  
  We ensured that the playbook asserts the presence of these variables via `secrets.yml`.

---

### 3. Unexpected Token `<` Error

- **Error Message:**  
  `Error: Unexpected token '<'`

- **Cause:**  
  This typically happens when a process expecting JSON or command output instead receives an HTML error page.

- **Resolution:**  
  We verified network and proxy configurations and adjusted verbosity for detailed debugging.

---

### 4. Permission Denied Error (Errno 13)

- **Error Message:**  
  `[Errno 13] Permission denied: 'pom.properties'`

- **Cause:**  
  This indicated file permission issues during the Maven build.

- **Resolution:**  
  We adjusted file ownership and permissions (using become in Ansible) so that the build process could read necessary files.

---

### 5. Docker COPY Failed Error

- **Error Message:**  
  `COPY failed: file not found in build context or excluded by .dockerignore: stat target/MessagingService-0.0.1-SNAPSHOT.jar: file does not exist`

- **Cause:**  
  The Dockerfile expected a built artifact that was missing or not found in the expected path.

- **Resolution:**  
  We verified the Maven build output and ensured the artifact was generated in the correct location.

---

## Improvements Made

- Full transition from Dockerized Maven builds to native Java 17 Maven builds for reliability.
- Introduced metadata tracking with commit hashes and timestamps.
- Implemented `build_metadata.json` for auditability and tagging.
- Cleanup logic to remove local clones after build.
- Frontend auto console setup and reset UI.
- Separation of build vs. Docker packaging logic for better debugging.

---

## What Can Be Improved

### 1. Dynamic Build Tool Handling

Currently, the playbook assumes Maven only. In a future version, it should:

- Parse `build_tool` input (`maven`, `gradle`, `npm`, etc.).
- Set appropriate build commands:
  - Maven: `mvn clean package`
  - Gradle: `./gradlew build`
  - NPM: `npm install && npm run build`
- Use conditionals in Ansible to switch build strategy.

### 2. Multi-language Support

- Templates and tasks can be extended to support Python, CMake, Rust, and others.

### 3. Better Log Aggregation

- Separate `build.log`, `docker.log`, and `metadata.log`.
- JSON format for structured parsing.

### 4. Webhook or GitHub Actions Support

- Trigger pipeline via GitHub webhooks or integrate into GitHub Actions CI/CD.

---

## Conclusion

The project successfully automates Maven-based Java project builds and Docker deployments. The system can be expanded to support broader build tools and full-stack pipelines, and it lays a strong foundation for an Internal Developer Platform (IDP).
