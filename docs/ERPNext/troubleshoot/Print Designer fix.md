wkhtmltopdf and chromium

```shell
docker exec -u 0 -it  frappe_docker-backend-1 bash
```

```shell
cd sites
```

```shell
vim common_site_config.json
```

Add this line:

```json
"chromium_binary_path": "/usr/bin/chromium",
"chromium_path": "/usr/bin/chromium",
"host_name": "http://frappe_docker-frontend-1:8000",
"default_site": "frontend",

```

It should look like this:

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
  "redis_socketio": "redis://redis-queue: 6379",
  "socketio_port": 9000
}
```

1. Add the chromium package (and other print_designer dependencies) into images/custom/Containerfile after the section for weasyprint dependencies
2. Build the image
3. Add this command to the configurator container after its other commands in the compose yaml file: `bench set-config -g chromium_binary_path /usr/bin/chromium`
4. Change the compose yaml file to refer to the new images and start the project back up
5. Uninstall and Reinstall print_designer app into the site(s)
6. Even though the app will now use the configured chromium it still wants to download chromium into the bench directory, but this is unnecessary. 
   
   