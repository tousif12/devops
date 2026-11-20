# Docker Commands Cheat Sheet (Easy Explanations)

A simple guide to the most useful Docker commands. Each command has a short explanation and an example.

Quick vocabulary:
- **Image**: A template or recipe (like `nginx`, `mysql`, `python`). It does not run by itself.
- **Container**: A running copy of an image. You can have many containers from one image.
- **Port mapping**: Connecting a port on your machine to a port inside the container.
- **Volume**: A storage area that keeps data even if the container is deleted.

---

## 1. Running Containers

### Start a container
```bash
docker run --name web01 -d -p 9080:80 nginx
```
| Part | Meaning |
|------|---------|
| `docker run` | Create and start a new container |
| `--name web01` | Give the container a friendly name |
| `-d` | Run in the background (detached) |
| `-p 9080:80` | Your machine's port 9080 goes to the container's port 80 |
| `nginx` | The image to use |

Open `http://localhost:9080` in the browser to see it.

### Run and open a shell inside (interactive)
```bash
docker run -it ubuntu bash
```
- `-i` keeps input open, `-t` gives you a terminal. You land inside the container. Type `exit` to leave.

### Run and auto delete when it stops
```bash
docker run --rm -it alpine sh
```
- `--rm` removes the container automatically after you exit. Good for quick tests.

### Pass environment variables
```bash
docker run -d --name db01 -e MYSQL_ROOT_PASSWORD=secret mysql
```
- `-e` sets a variable inside the container (often used for passwords and settings).

### Mount a folder from your machine
```bash
docker run -d --name web02 -p 9081:80 -v $(pwd)/site:/usr/share/nginx/html nginx
```
- `-v host_folder:container_folder` shares a folder. Edit files on your machine and the container sees them.

### Restart automatically
```bash
docker run -d --name web03 --restart always nginx
```
- The container starts again if it crashes or the machine reboots.

---

## 2. Seeing What Is Running

### List running containers
```bash
docker ps
```

### List all containers (including stopped)
```bash
docker ps -a
```

### Show only container IDs
```bash
docker ps -q
```
- Useful in scripts, for example to stop everything (see section 4).

---

## 3. Inspecting and Debugging

### Full details of a container
```bash
docker inspect web01
```
- Shows everything in JSON: IP address, ports, volumes, environment variables, status, and more.

### Get just one value (for example the IP address)
```bash
docker inspect -f '{{.NetworkSettings.IPAddress}}' web01
```
- `-f` filters the output so you don't have to read the whole JSON.

### View logs
```bash
docker logs web01
```

### Follow logs live
```bash
docker logs -f web01
```
- Like `tail -f`. Press `Ctrl + C` to stop watching.

### Last 50 lines of logs
```bash
docker logs --tail 50 web01
```

### Open a shell inside a running container
```bash
docker exec -it web01 bash
```
- If `bash` is not available (small images like alpine), use `sh` instead.

### Run a single command inside a container
```bash
docker exec web01 ls /usr/share/nginx/html
```

### Open a database prompt inside a container (MySQL)
```bash
docker exec -it db01 mysql -u root -p
```
| Part | Meaning |
|------|---------|
| `docker exec` | Run a command inside a running container |
| `-it` | Interactive terminal, so you can type |
| `db01` | The container name |
| `mysql` | The MySQL client program |
| `-u root` | Log in as the `root` user |
| `-p` | Ask for the password (the one set with `MYSQL_ROOT_PASSWORD`) |

Once inside, you can run SQL:
```sql
SHOW DATABASES;
USE mydb;
SHOW TABLES;
SELECT * FROM users LIMIT 5;
exit
```
- Similar idea for other databases: `docker exec -it pg01 psql -U postgres` (PostgreSQL) or `docker exec -it mongo01 mongosh` (MongoDB).

### Why go inside a container?
Going inside is like opening the box to see what is happening. Common reasons:

1. **Debugging**: Check config files and whether your app files are really there.
   ```bash
   cat /etc/nginx/nginx.conf
   ls /usr/share/nginx/html
   ```
2. **Checking connections**: Can this container reach the database or another service?
   ```bash
   ping db
   curl http://api:8080
   ```
3. **Reading log files**: Some apps write logs to files, which `docker logs` does not show.
   ```bash
   tail -f /var/log/app/error.log
   ```
4. **Working with a database**: Run SQL queries directly (see the MySQL example above).
5. **Quick tests**: Check environment variables (`env`), installed versions, or run a one-time task like a migration.

Notes:
- `bash` not available? Use `sh` instead (common in small images like alpine).
- Type `exit` to leave. The container keeps running.
- Changes made inside are **temporary**. If the container is recreated, they are gone. For permanent changes, update your Dockerfile, config, or use a volume.

Easy way to remember: `docker logs` is listening from outside the door. `docker exec -it web01 bash` is walking inside the house.

### See CPU and memory usage
```bash
docker stats
```

### See processes inside a container
```bash
docker top web01
```

### Check which ports are mapped
```bash
docker port web01
```
- Example output: `80/tcp -> 0.0.0.0:9080`

---

## 4. Starting, Stopping, Removing

### Stop a container
```bash
docker stop web01
```

### Start a stopped container
```bash
docker start web01
```

### Restart a container
```bash
docker restart web01
```

### Remove a container (must be stopped first)
```bash
docker rm web01
```

### Force remove a running container
```bash
docker rm -f web01
```

### Stop all running containers
```bash
docker stop $(docker ps -q)
```

### Remove all stopped containers
```bash
docker container prune
```

---

## 5. Working With Images

### List images on your machine
```bash
docker images
```

### Download an image
```bash
docker pull nginx
```

### Download a specific version (tag)
```bash
docker pull nginx:1.25
```
- Without a tag, Docker uses `latest`.

### Remove an image
```bash
docker rmi nginx
```

### Remove unused images
```bash
docker image prune -a
```

### Build your own image from a Dockerfile
```bash
docker build -t myapp:1.0 .
```
- `-t` gives the image a name and tag. The `.` means "use the Dockerfile in this folder".

### Tag an image with a new name
```bash
docker tag myapp:1.0 myusername/myapp:1.0
```

### Push an image to Docker Hub
```bash
docker login
docker push myusername/myapp:1.0
```

### See how an image was built (its layers)
```bash
docker history nginx
```

---

## 6. Copying Files

### Copy from your machine into a container
```bash
docker cp index.html web01:/usr/share/nginx/html/
```

### Copy from a container to your machine
```bash
docker cp web01:/etc/nginx/nginx.conf ./nginx.conf
```

---

## 7. Volumes (Saving Data)

### Create a volume
```bash
docker volume create mydata
```

### Use the volume in a container
```bash
docker run -d --name db02 -v mydata:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=secret mysql
```
- Database files are stored in `mydata`. Delete the container and the data is still safe.

### List volumes
```bash
docker volume ls
```

### Inspect a volume
```bash
docker volume inspect mydata
```

### Remove a volume
```bash
docker volume rm mydata
```

---

## 8. Networks (Letting Containers Talk)

### Create a network
```bash
docker network create mynet
```

### Run containers on the same network
```bash
docker run -d --name api --network mynet myapp:1.0
docker run -d --name db --network mynet -e MYSQL_ROOT_PASSWORD=secret mysql
```
- Now `api` can reach the database simply using the hostname `db`.

### List networks
```bash
docker network ls
```

### Inspect a network (see which containers are connected)
```bash
docker network inspect mynet
```

### Connect an existing container to a network
```bash
docker network connect mynet web01
```

---

## 9. Docker Compose (Many Containers at Once)

### Where Compose fits
Think of Docker in 3 levels:
1. **Image**: The recipe (like `nginx`, `mysql`).
2. **Container**: One running copy of a recipe (`docker run`).
3. **Compose**: Run many containers together, described in one file.

Easy way to remember: `docker run` is cooking one dish by hand. `docker compose` is a full menu written on paper. You say "start the menu" and everything is cooked together, in the right order.

### Why use it
Without Compose, a project with a database, an API and a website needs many long commands:
```bash
docker network create mynet
docker run -d --name db --network mynet -e MYSQL_ROOT_PASSWORD=secret -v dbdata:/var/lib/mysql mysql
docker run -d --name api --network mynet -p 8080:8080 myapi
docker run -d --name web --network mynet -p 9080:80 nginx
```
With Compose, you write it once in a file and start everything with one command.

### Example `docker-compose.yml`
```yaml
services:
  db:
    image: mysql
    environment:
      MYSQL_ROOT_PASSWORD: secret
    volumes:
      - dbdata:/var/lib/mysql

  api:
    build: ./api            # build image from ./api/Dockerfile
    ports:
      - "8080:8080"
    depends_on:
      - db                  # start db before api

  web:
    image: nginx
    ports:
      - "9080:80"
    restart: always

volumes:
  dbdata:
```
| Key | Meaning |
|-----|---------|
| `services` | The list of containers in your project |
| `image` | Use a ready image from Docker Hub |
| `build` | Build your own image from a folder with a Dockerfile |
| `ports` | Same as `-p HOST:CONTAINER` |
| `environment` | Same as `-e` |
| `volumes` | Same as `-v`, keeps data safe |
| `depends_on` | Start order (db first, then api) |
| `restart` | Same as `--restart` |

Bonus: Compose creates a network automatically, so `api` can reach the database simply using the hostname `db`.

> Run all commands below from the folder that has `docker-compose.yml`. Use the **service name** (`web`, `db`, `api`), not the container name.

### Start and stop
```bash
docker compose up -d            # start everything in the background
docker compose up -d web        # start only the web service (and what it depends on)
docker compose down             # stop and remove containers and network
docker compose down -v          # same, and also delete volumes (careful: deletes data)
docker compose stop             # stop containers but keep them
docker compose start            # start stopped containers again
```
- `down` removes containers. `stop` just pauses them, so `start` can bring them back.

### Restart
```bash
docker compose restart          # restart all services
docker compose restart api      # restart only the api service
```
- Useful after changing a config file that is mounted with a volume.
- Note: `restart` does **not** pick up changes in `docker-compose.yml` or code baked into the image. For that, use `up -d` or `up -d --build`.

### Go inside a service (exec)
```bash
docker compose exec web bash                    # open a shell in the web service
docker compose exec web sh                      # if bash is not available
docker compose exec db mysql -u root -p         # open MySQL prompt in the db service
docker compose exec api env                     # run one command and see the output
```
- Same as `docker exec -it`, but you use the service name and `-it` is on by default.

### Run a one-time command (run)
```bash
docker compose run --rm api python manage.py migrate
```
- Starts a **new temporary** container for the service, runs the command, then removes it (`--rm`).
- `exec` = use the container that is already running. `run` = start a fresh one.

### See what is going on
```bash
docker compose ps               # list services and their status
docker compose ps -a            # include stopped ones
docker compose logs             # logs from all services
docker compose logs -f api      # follow logs of one service live
docker compose logs --tail 50   # last 50 lines
docker compose top              # processes running in each service
```

### Build and update
```bash
docker compose build            # build images that use "build:"
docker compose build --no-cache # build from scratch, ignore cache
docker compose up -d --build    # rebuild and restart in one step
docker compose pull             # download newer versions of images
docker compose up -d            # apply changes after editing docker-compose.yml
```
- Changed your code? Use `up -d --build`.
- Changed only `docker-compose.yml`? `up -d` is enough. Compose recreates only what changed.

### Scale a service
```bash
docker compose up -d --scale api=3
```
- Runs 3 copies of `api`. (Remove fixed `ports` on that service first, or they will clash.)

### Check and debug the file
```bash
docker compose config           # show the final config, catches typos
docker compose images           # images used by each service
docker compose port web 80      # which host port maps to web's port 80
```

### Use a different file or project name
```bash
docker compose -f docker-compose.prod.yml up -d
docker compose -p myproject up -d
```
- `-f` picks a different compose file. `-p` sets the project name (prefix for container names).

### Compose quick table
| Task | Command |
|------|---------|
| Start all | `docker compose up -d` |
| Stop and remove | `docker compose down` |
| Stop only | `docker compose stop` |
| Restart one service | `docker compose restart api` |
| Go inside | `docker compose exec web bash` |
| MySQL prompt | `docker compose exec db mysql -u root -p` |
| One-time command | `docker compose run --rm api <command>` |
| Status | `docker compose ps` |
| Live logs | `docker compose logs -f api` |
| Rebuild and start | `docker compose up -d --build` |
| Validate file | `docker compose config` |

### When to use which
- `docker run`: Quick tests or a single container.
- `docker compose`: Real projects with more than one service, or when you want the setup saved in a file to share with your team.

> Older setups use `docker-compose` (with a dash). New Docker uses `docker compose` (with a space). They work the same way.

---

## 10. Cleanup and System Info

### See disk space used by Docker
```bash
docker system df
```

### Clean up everything unused (stopped containers, unused networks, dangling images)
```bash
docker system prune
```

### Even deeper cleanup (also unused images and volumes)
```bash
docker system prune -a --volumes
```
- Careful: this deletes data in unused volumes.

### Docker version and info
```bash
docker version
docker info
```

---

## Quick Reference Table

| Task | Command |
|------|---------|
| Run container | `docker run --name web01 -d -p 9080:80 nginx` |
| List running | `docker ps` |
| List all | `docker ps -a` |
| Details | `docker inspect web01` |
| Logs | `docker logs -f web01` |
| Shell inside | `docker exec -it web01 bash` |
| MySQL prompt inside | `docker exec -it db01 mysql -u root -p` |
| Stop / Start | `docker stop web01` / `docker start web01` |
| Remove | `docker rm -f web01` |
| List images | `docker images` |
| Pull image | `docker pull nginx` |
| Build image | `docker build -t myapp:1.0 .` |
| Copy file in | `docker cp file.txt web01:/path/` |
| Resource usage | `docker stats` |
| Start Compose project | `docker compose up -d` |
| Shell in Compose service | `docker compose exec web bash` |
| Cleanup | `docker system prune` |

---

## Tips

- Port format is always **host:container**. `-p 9080:80` means "my 9080 goes to container 80".
- You can use the container **name** or the first few characters of its **ID** in any command.
- Use `docker <command> --help` to see all options, for example `docker run --help`.
