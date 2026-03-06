# Dockerized Flask Application with Redis


Hello, my name is **Amine**.

I am a **future Network and Programmable Services Engineer** with a strong interest in **Linux system administration, networking, and DevOps technologies**. I am passionate about building, managing, and automating IT infrastructures using open-source tools and modern practices.

My technical focus is mainly on **Linux environments**, particularly **Red Hat Enterprise Linux**, where I develop and test most of my projects. Thanks to my solid foundation in **Linux system administration**, I prefer working in Red Hat–based environments to design and deploy reliable systems.


This project demonstrates how to **containerize a Python Flask application using Docker and Docker Compose** while integrating a **Redis service** for state persistence.

The goal of this project is to illustrate core **DevOps concepts**, including:

* Containerization with Docker
* Multi-service orchestration using Docker Compose
* Service-to-service communication inside containers
* Image rebuilding vs bind mount volumes
* Development workflow optimization with containers

---

# Project Architecture

The application consists of two services:

1. **Web Application**

   * Python Flask API
   * Tracks the number of visits to the application
2. **Redis Database**

   * Stores the number of visits

The architecture is managed using **Docker Compose**.


![Screenshot](Screenshot%20(7859).png)



---

# Project Structure

Run the `tree` command to view the project structure.

```
project/
│
├── app.py
├── requirements.txt
├── Dockerfile
└── docker-compose.yml
```

---

# Application Code

`app.py`

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
    return 'Hello From Amine, I have been seen {} times.\n'.format(count)
```


This application is a simple **Python web service** built using the Flask framework and integrated with a Redis database. The main objective of this project is to demonstrate how a lightweight web application can interact with a key-value data store to persist data across requests. The application counts how many times the webpage has been visited and displays the result to the user.

The program begins by importing the required libraries. The **time** module is used to introduce delays when retrying connections, **redis** is used to interact with the Redis database, and **Flask** is used to build the web application. A Flask application instance is then created, which serves as the foundation of the web service.

Next, the application establishes a connection to the Redis service using `redis.Redis(host='redis', port=6379)`. The hostname **redis** refers to the Redis container or service running in the environment, which is commonly defined in a containerized setup such as Docker Compose. Redis acts as a fast in-memory database that stores the number of visits to the application.

The `get_hit_count()` function is responsible for retrieving and updating the number of visits. It attempts to increment a value stored in Redis under the key **hits** using the `incr()` command. This command automatically increases the stored number each time the function is called. To make the system more reliable, the function includes a retry mechanism. If the Redis service is temporarily unavailable, the application will retry the connection several times with a short delay before failing.

The route definition `@app.route('/')` creates the main endpoint of the web application. When a user accesses the root URL of the application, the `hello()` function is executed. This function calls `get_hit_count()` to retrieve the current number of visits and then returns a formatted message displaying how many times the page has been viewed.




---

# Requirements

`requirements.txt`

```
flask
redis
```

The `requirements.txt` file defines the Python dependencies required for this project to run correctly. It allows anyone who clones the repository to easily install the necessary libraries using a single command with **pip**.

In this project, the application relies on two main Python packages:

**Flask**
Flask is a lightweight Python web framework used to build the web application. It handles HTTP requests, routing, and responses, allowing the application to serve a web page when users access the service.

**Redis**
The Redis Python client library enables the application to communicate with the Redis database. It allows the program to store and update the visit counter using Redis commands, such as incrementing the number of page visits.


```
pip install -r requirements.txt
```

This approach follows best practices in Python development and helps ensure consistency across different development and deployment environments.


---

# Dockerfile

The Dockerfile builds a lightweight container image based on **Python Alpine**.

```dockerfile
FROM python:3.7-alpine

WORKDIR /app

ENV FLASK_APP=app.py
ENV FLASK_RUN_HOST=0.0.0.0

RUN apk add --no-cache gcc musl-dev linux-headers

COPY requirements.txt requirements.txt
RUN pip install -r requirements.txt

EXPOSE 5000

COPY . .

CMD ["flask", "run"]
```


This Dockerfile defines the environment required to run the Flask application inside a container. It starts from the **Python 3.7 Alpine** base image, which is lightweight and suitable for building small and efficient containers.

The `WORKDIR /app` instruction sets the working directory inside the container where the application files will be stored and executed. Environment variables such as `FLASK_APP` and `FLASK_RUN_HOST` are configured to specify the main Flask application file and allow the service to be accessible from outside the container.

Next, the Dockerfile installs the required system packages needed to build some Python dependencies. The `requirements.txt` file is copied into the container, and the necessary Python libraries are installed using `pip`.

The container exposes **port 5000**, which is the default port used by the Flask web server. Finally, all project files are copied into the container, and the Flask application starts automatically using the `flask run` command when the container is launched.

---

# Docker Compose Configuration

`docker-compose.yml`

```yaml
version: '3'

services:
  web:
    build: .
    ports:
      - "9000:5000"

  redis:
    image: redis:alpine
```

The `docker-compose.yml` file defines and manages the multi-container environment used in this project. It allows the application and its dependencies to run together using a single command.

In this configuration, two services are defined: **web** and **redis**. The **web** service builds the container image from the Dockerfile in the current directory and maps port **9000** on the host machine to port **5000** inside the container, where the Flask application runs.

The **redis** service uses the official Redis Alpine image, which provides a lightweight Redis database used to store the visit counter for the application.

Docker Compose automatically creates a default network that allows both containers to communicate with each other. Thanks to this network, the Flask application can connect to Redis using the hostname **redis**, which corresponds to the Redis service name defined in the configuration.

---

# Running the Project

Start the containers:

```bash
docker compose up -d
```
![Screenshot](Screenshot%20(7857).png)

Access the application:

```
http://localhost:9000
```

Each refresh increments the counter stored in Redis.

![Screenshot](Screenshot%20(7860).png)

![Screenshot](Screenshot%20(7861).png)
---

# Updating the Application Code

When changing the application code (for example modifying the text in `app.py`), the running container **will not reflect the change automatically**.

Example change:

```
Hello From Amine
```

to

```
Hello from a Future Network Engineer
```

Refreshing the page will still show the old message.

---

# Option 1: Rebuild the Docker Image

Stop containers:

```bash
docker-compose down
```

Rebuild and start again:

```bash
docker-compose up -d --build
```

This rebuilds the Docker image with the updated code.

However, rebuilding images for every change **slows down development**.

---

# Option 2: Using Bind Mount Volumes (Recommended for Development)

Update `docker-compose.yml`:

```yaml
version: '3'

services:
  web:
    build: .
    ports:
      - "9000:5000"
    volumes:
      - .:/app
    environment:
      FLASK_DEBUG: "true"

  redis:
    image: redis:alpine
```

Now restart the services:

```bash
docker-compose up -d
```



# Why Changes Reflect Without Rebuilding

The line:

```
volumes:
  - .:/app
```

creates a **bind mount** between the host machine and the container.
This means: The **current project directory on the host** is mounted into `/app` inside the container. When a file is modified on the host (e.g., `app.py`), the container immediately sees the change.

Because Flask is running in **debug mode**, it automatically reloads the application when code changes.


This approach is commonly used in **development environments**, while production environments rely on **immutable container images**.




# Technologies Used

* Python
* Flask
* Redis
* Docker
* Docker Compose

