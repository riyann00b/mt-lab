# Print Designer Troubleshooting Guide

## Overview

This guide documents the steps required to restore `print_designer` functionality in an ERPNext deployment that relies on Chromium for PDF generation. The procedure covers the configuration updates inside the running container, container image changes, and follow-up actions to ensure that `print_designer` targets the system Chromium binary instead of downloading its own copy.

## Symptoms

- Printing from `print_designer` fails or produces blank outputs.
- Logs show errors referencing missing Chromium binaries.
- The app attempts to download Chromium into the bench directory even though Chromium exists in the base image.

## Root Cause

The container image is missing the Chromium dependency and related configuration. ERPNext’s `print_designer` app falls back to downloading Chromium when it cannot locate a configured binary path, leading to repeated failures or redundant downloads.

## Prerequisites

- Administrative access to the Docker host running the ERPNext stack.
- Ability to rebuild and redeploy custom container images.
- Access to the project’s `docker-compose.yml` and `images/custom/Containerfile`.
- Familiarity with editing JSON files (specifically `common_site_config.json`).

## Resolution Overview

1. Configure the active bench to reference the system Chromium binary.
2. Bake Chromium and related dependencies into the custom container image.
3. Update the configurator container to set the Chromium path automatically.
4. Rebuild and redeploy the stack with the updated image.
5. Reinstall the `print_designer` app so it consumes the new configuration.

## Step-by-Step Resolution

### 1. Configure `common_site_config.json`

1. Exec into the backend container as root:

   ```bash
   docker exec -u 0 -it frappe_docker-backend-1 bash
   ```
   
2. Navigate to the sites directory:

   ```bash
   cd sites
   ```
   
3. Edit `common_site_config.json`:

   ```bash
   vim common_site_config.json
   ```
   
4. Ensure the file includes the following keys (updating host name and default site to match your environment):
   
   ```json
   {
     "chromium_binary_path": "/usr/bin/chromium",
     "chromium_path": "/usr/bin/chromium",
     "db_host": "db",
     "db_port": 3306,
     "default_site": "frontend",
     "host_name": "http://frappe_docker-frontend-1:8080",
     "redis_cache": "redis://redis-cache:6379",
     "redis_queue": "redis://redis-queue:6379",
     "redis_socketio": "redis://redis-queue:6379",
     "socketio_port": 9000
   }
   ```

### 2. Extend the custom container image

1. Open `images/custom/Containerfile`.
2. After the section that installs WeasyPrint dependencies, add Chromium and any other required packages for `print_designer`. For example:
   ```
   RUN apt-get update && apt-get install -y chromium && apt-get clean
   ```
   (Adapt the package list as needed for your base image.)
3. Save the file.

### 3. Rebuild the custom image

From the project root, rebuild the image that powers the backend (or the relevant service defined in your compose file).

### 4. Update the configurator commands

1. In `docker-compose.yml`, locate the configurator service.
2. Append the following command after existing ones to ensure the Chromium path is set during configuration:
   ```bash
   bench set-config -g chromium_binary_path /usr/bin/chromium
   ```

### 5. Deploy the updated stack

1. Point the relevant services in `docker-compose.yml` to the newly built custom image tag.
2. Restart the stack to apply changes.

### 6. Reinstall `print_designer`

1. Uninstall `print_designer` from each affected site.
2. Reinstall `print_designer` so it reads the updated bench configuration.

### 7. Acknowledge redundant Chromium downloads

Even with the binary path configured, the app may still attempt to download Chromium into the bench directory. This download is unnecessary and can be safely ignored as long as the configured `/usr/bin/chromium` executable remains available.

## Additional Notes

- If the host name or default site differ from the example values, update them accordingly in `common_site_config.json`.
- Rebuilding containers may require downtime; schedule maintenance windows to minimize disruption.
- Monitor disk usage to ensure redundant Chromium downloads do not accumulate unexpectedly.
