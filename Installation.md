## ✅ Prerequisites

Ensure the following software is installed and running:
- **Docker**
- **Docker Compose**
- **Git**
- **Optional**: `jq` (for JSON validation), `nano` or `VS Code` (for editing files)

## ERPNext Self-Hosted Docker Container with Custom Apps

### Building the Custom Docker Image

#### Step 1: Clone the Frappe Docker Repository

Clone the official `frappe_docker` repository and navigate into the `development` directory.

```shell
git clone https://github.com/frappe/frappe_docker
cd frappe_docker/development
```

#### Step 2: Configure Custom Applications via `apps.json`

Create a file named `apps.json` that lists all the apps to be installed.

First, remove the example file:

```shell
rm -f apps-example.json
```

Then, create and edit the new file using your preferred editor:

```shell
# Using nano
nano apps.json

# Or using VS Code
code apps.json
```

**`apps.json` Example:**

```json
[
  {
    "url": "https://github.com/frappe/erpnext.git",
    "branch": "version-15"
  },
  {
    "url": "https://github.com/frappe/payments.git",
    "branch": "version-15"
  },
  {
    "url": "https://github.com/resilient-tech/india-compliance",
    "branch": "version-15"
  },
  {
    "url": "https://github.com/frappe/ecommerce_integrations.git",
    "branch": "main"
  },
  {
    "url": "https://github.com/frappe/print_designer.git",
    "branch": "main"
  },
  {
    "url": "https://github.com/frappe/insights.git",
    "branch": "version-3"
  }
]
```

> [!Important]
> **Dependencies**: You must manually add app dependencies. For example, `hrms` requires `erpnext` to be in the list. **Private Repos**: For private repositories, use an HTTPS git URL with a Personal Access Token (PAT): `https://{{PAT}}@github.com/your-org/your-private-app.git`.

#### Step 3: Encode `apps.json` to Base64

The Docker build process requires the `apps.json` content to be passed as a Base64 encoded string.

Note

Ensure you are in the `frappe_docker/development` directory.

```shell
export APPS_JSON_BASE64=$(base64 -w 0 apps.json)
```

To validate the string, you can decode it and check the contents:

```shell
# Decode the variable into a test file and display it
echo -n ${APPS_JSON_BASE64} | base64 -d > apps-test-output.json
cat apps-test-output.json

# Optional: Validate the original JSON syntax (requires jq)
jq empty apps.json
```

#### Step 4: Build and Push the Custom Docker Image

This is a multi-stage process to build, tag, and push your image.

##### 🏗️ A. Build the Image

Note

Navigate back to the root `frappe_docker` directory.

```shell
cd ..
```

Run the build command using the **Quick Build** method for faster builds.

**Quick Build Command Example:**

Refer to [configure-build](https://github.com/frappe/frappe_docker/blob/main/docs/custom-apps.md#configure-build) for full details.

> [!TIP]
> Change the `tag` to your Docker Hub `username` and tag the image as `version`. 

```shell
docker build \
  --build-arg=FRAPPE_PATH=https://github.com/frappe/frappe \
  --build-arg=FRAPPE_BRANCH=version-15 \
  --build-arg=APPS_JSON_BASE64=$APPS_JSON_BASE64 \
  --tag=riyann00b/frappe-custom:v15.87.0 \
  --file=images/layered/Containerfile .
```

Important

☕ Sit back, relax, and have a coffee. This will take a while.

Verify the new image:

```shell
docker images
```

#### Step 5: Docker Compose Configuration and Launch

##### Create the Final `pwd.yml` Docker Compose File

> [!TIP]
> Navigate to the `frappe_docker` directory if not already there:

```shell
cd /path/to/your/frappe_docker
# or
cd ..
```

Edit the `pwd.yml` file with your preferred text editor:

```shell
# Using nano
sudo nano pwd.yml

# Or using VS Code
code pwd.yml
```

> [!Tip]
> - Replace all instances of the default image (e.g., `frappe/erpnext:v15.xx.x`) with your custom image (e.g., `riyann00b/frappe-custom:version`).
> - Add `platform: linux/amd64` to all services in the `pwd.yml` file to ensure compatibility (e.g., for Linux, Apple M1/M2 systems).
> - Update the `create-site` service's command to include your custom apps for installation.
 
**Example modification in `pwd.yml`:**

```yaml
create-site:
command:
bench new-site --mariadb-user-host-login-scope='%' --admin-password=admin --db-root-username=root --db-root-password=admin --install-app erpnext 
'--install-app YOUR-CUSTOM-APPs-name' --set-default frontend;
```

See example `pwd.yml` file [here]().


> [!TIP] 
> Apps to copy for the `--install-app` flag:

```shell
--install-app payments --install-app india_compliance --install-app ecommerce_integrations --install-app print_designer --install-app insights
```

#### Step 6: Launch the Stack and Finalize Setup

1. **Launch all services:** From the root `frappe_docker` directory, run:
    
    ```shell
    docker compose -f pwd.yml up -d
    ```
    
2. **Monitor site creation**:
    
    ```shell
    docker logs frappe_docker-create-site-1 -f
    ```
    
1. **Access your site:** Open your browser and navigate to [http://localhost:8080](http://localhost:8080/).


### How to update?

Change the image tag for all the frappe framework services, take down all the containers and start them again with new images. Once the containers are running you'll need to migrate sites with `bench --site all migrate` command. The image and container replacement is done as per the container orchestrator.

### How to Backup?

Create backup service or stack.

```yaml
# backup-job.yml
version: "3.7"
services:
  backup:
    image: frappe/erpnext:${VERSION}
    entrypoint: ["bash", "-c"]
    command:
      - |
        bench --site all backup
        ## Uncomment for restic snapshots.
        # restic snapshots || restic init
        # restic backup sites
        ## Uncomment to keep only last n=30 snapshots.
        # restic forget --group-by=paths --keep-last=30 --prune
    environment:
      # Set correct environment variables for restic
      - RESTIC_REPOSITORY=s3:https://s3.endpoint.com/restic
      - AWS_ACCESS_KEY_ID=access_key
      - AWS_SECRET_ACCESS_KEY=secret_access_key
      - RESTIC_PASSWORD=restic_password
    volumes:
      - "sites:/home/frappe/frappe-bench/sites"
    networks:
      - erpnext-network

networks:
  erpnext-network:
    external: true
    name: ${PROJECT_NAME:-erpnext}_default

volumes:
  sites:
    external: true
    name: ${PROJECT_NAME:-erpnext}_sites
```

In case of single docker host setup, add crontab entry for backup every 6 hours.

```
0 */6 * * * docker compose -p erpnext exec frappe_docker-backend-1 bench --site all backup --with-files > /dev/null
```

Notes:

- Make sure `docker-compose` or `docker compose` is available in path during execution.
- Change the cron string as per need.
- Set the correct project name in place of `erpnext`.
- For Docker Swarm add it as a [swarm-cronjob](https://github.com/crazy-max/swarm-cronjob)
- Add it as a `CronJob` in case of Kubernetes cluster.