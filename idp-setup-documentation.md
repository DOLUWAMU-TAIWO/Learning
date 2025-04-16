
# Internal Developer Platform (IDP) Setup Documentation

## Overview

This document outlines the setup and configuration of a lightweight Internal Developer Platform (IDP) hosted on a control node. The platform includes:

- A Flask-based backend API for deployment automation
- An Nginx web server serving a static HTML front end at `/idp/`
- Basic HTTP authentication for front-end access
- Real-time problem-solving steps taken during deployment

---

## Directory Structure

```
/var/www/idp/         # Static files for the front end (index.html)
/etc/nginx/conf.d/    # Nginx config for serving the IDP site
/etc/nginx/.htpasswd_idp # Password file for Basic Auth
```

---

## Steps Completed

### 1. Flask API Server
- Created a Flask app (`idp_api.py`) that accepts deployment details and triggers an Ansible playbook.

### 2. Front-End Setup
- Developed a basic HTML + JavaScript form to collect:
  - Git Repo URL
  - App Name
  - App Port
  - Container Port
  - Build Tool
- Deployed to `/var/www/idp/`

### 3. Nginx Configuration

**Initial Issues Encountered:**
- 404 Errors when accessing `/idp/`
- 403 Forbidden due to incorrect permissions
- Redirection loops caused by improper `try_files` and `alias` configuration

**Fixes Applied:**
- Replaced `alias` config with `root` for cleaner path resolution
- Corrected file ownership and permissions:
  ```bash
  sudo chown -R www-data:www-data /var/www/idp/
  sudo chmod -R 755 /var/www/idp/
  ```
- Added redirect for `/idp` to `/idp/`:
  ```nginx
  location = /idp {
      return 301 $scheme://$host/idp/;
  }
  ```

### 4. Basic Authentication

**Command Used:**
```bash
sudo htpasswd -c /etc/nginx/.htpasswd_idp modo
```

**Nginx Config for Password Protection:**
```nginx
location /idp/ {
    auth_basic "IDP Portal";
    auth_basic_user_file /etc/nginx/.htpasswd_idp;
    alias /var/www/idp/;
    index index.html;
}
```

---

## Final Result

You can now access the IDP front end securely via:

```
http://89.247.214.203/idp/
```

Login with your defined credentials (e.g., user `modo`) to use the deployment form. All submitted form data is forwarded to the internal Flask API for processing.

---

## Next Steps

- Add dynamic form elements based on selected build tool
- Implement a real database for deployment history
- Optionally integrate this form into Backstage via a custom plugin
