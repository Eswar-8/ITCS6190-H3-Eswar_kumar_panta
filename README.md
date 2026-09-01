# Hands-On L3: Containers with Docker

## Verification of Docker Installation

```bash
docker --version
```

## Standalone PostgreSQL Setup

1. Pull the official PostgreSQL 16 image from Docker Hub:
   ```bash
   docker pull postgres:16
   ```

2. Run the PostgreSQL container in detached mode with port mapping and environment variables set:
   ```bash
   docker run -d -p 5432:5432 --name postgres1 -e POSTGRES_PASSWORD=pass12345 postgres:16
   ```

3. Access the container's interactive shell:
   ```bash
   docker exec -it postgres1 bash
   ```

4. Connect to the database using the `psql` command-line utility:
   ```bash
   psql -d postgres -U postgres
   ```

5. Exit `psql` by entering `\q`, exit the container shell, and remove the container to free up port 5432:
   ```bash
   docker stop postgres1 && docker rm postgres1
   ```

## Building and Running the Multi-Container Web App

### Application Configuration

* **requirements.txt**
  ```text
  flask==3.0.3
  redis==5.0.7
  ```

* **app.py**
  ```python
  import time
  import redis
  from flask import Flask

  app = Flask(__name__)
  cache = redis.Redis(host='redis', port=6379)

  def get_hit_count():
      retries = 5
      while True:
          try:
              return cache.incr('hits')
          except redis.exceptions.ConnectionError as exc:
              if retries == 0:
                  raise exc
              retries -= 1
              time.sleep(0.5)

  @app.route('/')
  def hello():
      count = get_hit_count()
      return 'Hello World! I have been seen { } times.\n'.format(count)
  ```

* **Dockerfile**
  ```dockerfile
  FROM python:3.12-alpine
  WORKDIR /code
  ENV FLASK_APP=app.py
  ENV FLASK_RUN_HOST=0.0.0.0
  RUN apk add --no-cache gcc musl-dev linux-headers
  COPY requirements.txt requirements.txt
  RUN pip install -r requirements.txt
  EXPOSE 5000
  COPY . .
  CMD ["flask", "run"]
  ```

* **compose.yaml**
  ```yaml
  services:
  web:
    build: .
    ports:
      - 8000:5000
    depends_on:
      - redis
  redis:
    image: "redis:alpine"
    ```
  Build and run the application
  
  - docker compose up
  - open the application ``http://localhost:8000``
  - docker compose down
