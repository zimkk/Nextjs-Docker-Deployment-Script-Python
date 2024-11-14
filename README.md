# README.md

## Overview

This script automates the deployment of a Next.js application using Docker. It clones a GitHub repository, sets up the environment, builds a Docker image, and runs a Docker container.

## Prerequisites

- Python 3.x
- Docker
- Git

## Usage

1. Clone the repository containing this script.
2. Navigate to the directory containing the script.
3. Run the script:

```sh
python script.py
```

## Script Details

### Functions

#### 

is_port_available(port)



Checks if a given port is available.

```python
def is_port_available(port):
    with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as s:
        return s.connect_ex(('localhost', port)) != 0
```

#### 

get_free_port()



Finds an available port in the range 3001-3099.

```python
def get_free_port():
    for port in range(3001, 3100):
        if is_port_available(port):
            return port
    raise Exception("No available ports found.")
```

#### 

deploy_nextjs()



Clones the GitHub repository, sets up the environment, builds the Docker image, and runs the Docker container.

```python
def deploy_nextjs():
    repo_url = input("GitHub repo URL: ")
    env_path = input("Path to .env file: ")

    repo_name = repo_url.split('/')[-1].replace('.git', '')
    subprocess.run(f"git clone {repo_url}", shell=True)

    if not os.path.exists(repo_name):
        print(f"Failed to clone repository {repo_url}")
        sys.exit(1)

    os.chdir(repo_name)

    if not os.path.isfile(env_path):
        print(f".env file not found at {env_path}")
        sys.exit(1)
    
    subprocess.run(f"cp {env_path} .env", shell=True)

    dockerfile_content = """
    FROM node:16-alpine
    WORKDIR /app
    COPY package*.json ./
    RUN npm install
    COPY . .
    RUN npm run build
    EXPOSE 3000
    CMD ["npm", "start"]
    """

    with open("Dockerfile", "w") as dockerfile:
        dockerfile.write(dockerfile_content)

    image_name = f"{repo_name.lower()}_image"
    subprocess.run(f"docker build -t {image_name} .", shell=True)

    port = 3000 if is_port_available(3000) else get_free_port()
    container_name = f"{repo_name.lower()}_container"
    subprocess.run(f"docker run -d -p {port}:3000 --name {container_name} {image_name}", shell=True)
    print(f"App is running at http://localhost:{port}")
```

### Main Execution

The script starts by calling the 

deploy_nextjs

 function.

```python
if __name__ == "__main__":
    deploy_nextjs()
```

## License

This project is licensed under the MIT License. See the LICENSE file for details.