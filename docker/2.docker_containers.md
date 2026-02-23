# Docker Containers

---

# What is a Container?

A Docker container is a **running instance of a Docker image**.

- Image = Blueprint
- Container = Running application

---

# Container Lifecycle (With Example)

Let’s practice using the `nginx` image.

---

## Step 1: Create a Container (Without Starting)

`docker create --name mynginx nginx`

Check:

`docker ps -a`

`Status → Created`

---

## Step 2: Start the Container

`docker start mynginx`

Check:

`docker ps`

`Status → Up (Running)`

---

## Step 3: Pause the Container

`docker pause mynginx`

`Status → Paused`

---

## Step 4: Unpause the Container

`docker unpause mynginx`

---

## Step 5: Stop the Container

`docker stop mynginx`

`Status → Exited`

---

## Step 6: Restart the Container

`docker restart mynginx`

---

## Step 7: Kill the Container

`docker kill mynginx`

Forcefully stops the container.

---

## Step 8: Remove the Container

`docker rm mynginx`

Container is completely deleted.

---

# Working with Running Containers

---

## Run Nginx in Detached Mode

-d → Run in background  
-p → Map port  

`docker run -d -p 8080:80 --name webserver nginx`

---

## Understanding Port Mapping (-p)

Format:

-p `HOST_PORT:CONTAINER_PORT`

Example:

-p 8080:80

- 8080 → Host Port (your computer)
- 80 → Container Port (inside container)

What happens?

Browser → http://localhost:8080  
⬇  
Host Port 8080  
⬇  
Container Port 80  
⬇  
Nginx running inside container  

So:

Left side = Host  
Right side = Container

---

## Passing Environment Variables

Environment variables are used to pass configuration values into a container.

Use `-e` to pass environment variables.

### Example 1: Passing Single Variable

`docker run -e APP_ENV=production nginx`

This sets:

`APP_ENV=production` inside the container.

---

### Example 2: Multiple Environment Variables

`docker run -e USERNAME=admin -e PASSWORD=1234 nginx`

---

### Example 3: Check Environment Variables Inside Container

Run container:

`docker run -d --name envtest -e MY_NAME=Docker nginx`

Enter container:

`docker exec -it envtest /bin/sh`

Check variable:

`echo $MY_NAME`

Output:
`Docker`

---

Environment variables are commonly used for:
- Database credentials
- API keys
- Application configuration
- Environment settings (dev, test, prod)

---

## View Logs

`docker logs webserver`

Shows output logs of the container.

---

## View Real-Time Logs

`docker logs -f webserver`

Shows live logs (Press Ctrl + C to exit).

---

## Exec into the Container

`docker exec -it webserver /bin/sh`

Now you are inside the container.

Try:

ls /
cd /usr/share/nginx/html
ls

Type `exit` to leave.

---

## Run a Single Command Inside Container

`docker exec webserver ls /`

Runs one command without entering.

---

## Inspect the Container

`docker inspect webserver`

Shows detailed information like:

- Container ID
- IP address
- Port mappings
- Environment variables

---