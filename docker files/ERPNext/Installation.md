# 🧭 ERPNext Self-Hosted Docker with Custom Apps Installation Documentation

Comprehensive guide to build, configure, deploy, backup, and restore an **ERPNext** environment using Docker with **custom applications**.

---

## ✅ Prerequisites

Ensure the following software is installed and running:

- **Docker**
- **Docker Compose**
- **Git**
- **Optional Tools**:  
  - `jq` — JSON validation  
  - `nano` or `VS Code` — file editing

> [!NOTE]
> This guide assumes you're working on a Linux system with a Bash-compatible shell.
> Adjust the commands as needed for your operating system and shell.

---

## 🏗️ Building the Custom Docker Imageaccouding to os and your shell
### 1. Clone the Frappe Docker Repository

```bash
git clone https://github.com/frappe/frappe_docker
cd frappe_docker/development
```

---

### 2. Configure Custom Applications via `apps.json`

Create a new file named `apps.json` listing the apps to include.

```bash
rm -f apps-example.json
nano apps.json
# or
code apps.json
```

**Example `apps.json`:**

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
    "url": "https://github.com/frappe/hrms.git",
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

> [!IMPORTANT]
>
> * **Dependencies:** Ensure required apps (e.g., `hrms` requires `erpnext`) are listed.
> * **Private Repos:** Use HTTPS with a **Personal Access Token (PAT)** if needed:
>   `https://{{PAT}}@github.com/your-org/your-private-app.git`

---

### 3. Encode `apps.json` to Base64

From inside `frappe_docker/development`:

```bash
export APPS_JSON_BASE64=$(base64 -w 0 apps.json)
```

Validate encoding:

```bash
echo -n ${APPS_JSON_BASE64} | base64 -d > apps-test-output.json
cat apps-test-output.json
jq empty apps.json  # optional validation
```

---

### 4. Build and Push the Custom Docker Image

Navigate to the root folder:

```bash
cd ..
```

Run the build command:

> [!IMPORTANT]
> **Make sure to change your tag: `--tag=<your-docker-username>` and set the version `<latest-version>` to the current release (for example, `v15.88.1`).**

```bash
docker build \
  --build-arg=FRAPPE_PATH=https://github.com/frappe/frappe \
  --build-arg=FRAPPE_BRANCH=version-15 \
  --build-arg=APPS_JSON_BASE64=$APPS_JSON_BASE64 \
  --tag=riyann00b/frappe-custom:{latest-version} \
  --file=images/layered/Containerfile .
```

> ☕ This process may take a while. Be patient.

Verify build:

```bash
docker images
```

---

## ⚙️ Docker Compose Setup

### 5. Create and Configure `pwd.yml`

Go to the project directory:

```bash
cd /path/to/frappe_docker
```

Edit `pwd.yml`:

```bash
sudo nano pwd.yml
# or
code pwd.yml
```

**Modify:**

* Replace default ERPNext image with your custom image `(riyann00b/frappe-custom:{latest-version})`.
* Add `platform: linux/amd64` to all services.
* Include all your apps in the `create-site` command.

**Example:**

```yaml
create-site:
command:
bench new-site --mariadb-user-host-login-scope='%' --admin-password=admin --db-root-username=root --db-root-password=admin --install-app erpnext 
'--install-app YOUR-CUSTOM-APPs-name' --set-default frontend;
```

> [!TIP]
>  Apps to copy for the --install-app flag:
```shell
--install-app payments --install-app hrms --install-app india_compliance --install-app ecommerce_integrations --install-app print_designer --install-app insights
```
---

Check out my [pwd.yml file]()

### 6. Launch the Stack

```bash
docker compose -f pwd.yml up -d
```

Monitor logs:

```bash
docker logs frappe_docker-create-site-1 -f
```

Access ERPNext at:
👉 [http://localhost:8080](http://localhost:8080)

---

## 🔄 Updating ERPNext

1. Create New Image (follow the instructions from 1 to 4)
2. Update image tags in your pwd.yml file.
3. Restart containers:

```bash
docker compose down
docker compose up -d
```

4. Run migrations:

```bash
docker exec -it frappe_docker-backend-1 bench --site all migrate
```

---

## 💾 Backup Configuration

Create a backup stack file: `backup-job.yml` or include it in your `pwd.yml` file.

```yaml
version: "3.7"
services:
  backup:
    image: frappe/erpnext:${VERSION}
    entrypoint: ["bash", "-c"]
    command: |
      bench --site all backup
      # Optional Restic backup
      # restic snapshots || restic init
      # restic backup sites
      # restic forget --group-by=paths --keep-last=30 --prune
    environment:
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

**Cron-based backup (every 6 hours)**

```bash
0 */6 * * * docker compose -p erpnext exec frappe_docker-backend-1 bench --site all backup --with-files > /dev/null
```

**Manual Backup**
```bash
docker exec -it frappe_docker-backend-1 /bin/bash
```

```bash
bench --site frontend backup --with-files
```

or

```bash
docker exec frappe_docker-backend-1 bench --site frontend backup --with-files
```

copy backup to local path

```bash
docker cp frappe_docker-backend-1:/home/frappe/frappe-bench/sites/frontend/private/backups/ /home/riyan/Documents/backups
```

---

## 🔁 Restoring an ERPNext Backup

### Prerequisites

* Docker environment running.
* Backup files available (`database`, `public`, `private`).
* Site name known (e.g., `frontend`).

---

### Step 1: Identify Backend Container

```bash
docker ps
```

Expected container: `frappe_docker-backend-1`

---

### Step 2: Copy Backup Files From Local to Docker 

```bash
docker cp YYYY-MM-DD_HHMMSS-frontend-database.sql.gz frappe_docker-backend-1:/home/frappe/frappe-bench/sites/database.sql.gz
docker cp YYYY-MM-DD_HHMMSS-frontend-files.tar frappe_docker-backend-1:/home/frappe/frappe-bench/sites/public_files.tar
docker cp YYYY-MM-DD_HHMMSS-frontend-private-files.tar frappe_docker-backend-1:/home/frappe/frappe-bench/sites/private_files.tar
```

---

### Step 3: Access Container Shell

```bash
docker exec -it frappe_docker-backend-1 /bin/bash
```

---

### Step 4: Restore the Site

```bash
cd frappe-bench
bench --site frontend --force restore database.sql.gz \
  --with-public-files public_files.tar \
  --with-private-files private_files.tar
```

or

```bash
docker exec frappe_docker-backend-1 bench --site frontend restore /tmp/database.sql.gz --with-public-files /tmp/public-files.tar --with-private-files /tmp/private-files.tar --force

```

Default SQL password:

```bash
admin
```

---

### Step 5: Finalize and Migrate

```bash
bench --site frontend migrate
exit
```

---

### Step 6: Restart Containers

```bash
docker compose restart
```

Access ERPNext at:
👉 [http://localhost:8080](http://localhost:8080)

Default login:

```bash
Username: Administrator
Password: admin
```

---

## 🧩 References

* [Frappe Docker Documentation](https://github.com/frappe/frappe_docker)
* [ERPNext GitHub Repository](https://github.com/frappe/erpnext)
* [Docker Compose Docs](https://docs.docker.com/compose/)
