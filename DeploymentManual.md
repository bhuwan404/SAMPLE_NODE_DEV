### Steps Performed ###

1. Run locally
    a. Commands:
        npm install -g yarn
        npm run build
        npm run start
    b. Started on 3000 port
    c. Swagger UI URL: http://localhost:3000/api-docs/

2. Installation of Postgres and docker compose in remte EC2 (Ubuntu) instance
    a. Commands to install Postgresql:
        apt update -y
        apt install wget curl -y
        sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list'
        curl -fsSL https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo gpg --dearmor -o /etc/apt/trusted.gpg.d/postgresql.gpg
        apt install postgresql postgresql-client -y
        systemctl status postgresql
    b. Updated postgresql.conf and pg_hba.conf file for db access.
    c. Created test_db and test user.
    d. Command to install docker and docker compose
        apt install -y ca-certificates curl gnupg lsb-release
        mkdir -p /etc/apt/keyrings
        curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
        sudo echo  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu  $(lsb_release -cs) stable" | tee /etc/apt/sources.list.d/docker.list > /dev/null
        apt update
        apt install -y docker-ce docker-ce-cli containerd.io docker-compose-plugin

3. Updated .env
    a. .env file:
        PORT=3000
        HOST=localhost
        POSTGRES_DB_CONNECTION_URL=postgresql://test:test123@3.86.90.217:5432/test_db
        AUTH_ACCESS_TOKEN_SECRET=your_access_token_secret_for_testing
        AUTH_REFRESH_TOKEN_SECRET=your_refresh_token_secret_for_testing
        AUTH_ACCESS_TOKEN_DURATION=1h
        AUTH_REFRESH_TOKEN_DURATION=7d

4. PostgreSQL migration test locally
    a. Command:
        npm run db:generate

5. Linting and testing locally
    a. Got error on lint "npm run lint"
        Error: buildx failed with: ERROR: failed to solve: process "/bin/sh -c npm run lint" did not complete successfully: exit code: 2
    b. Got error test "npm run test"
        "moduleName":"Authentication","error":{"message":"No authorization header set"}

6. Created repo "bhuwan405/sample_node_dev" in personal dockerhub account

7. Created Dockerfile
    a. Dockerfile
        # Use the official Node.js 16 image as the base image
        FROM node:18-alpine

        # Set the working directory in the container
        WORKDIR /home/app

        # Copy package.json and package-lock.json files to the working directory
        COPY package*.json ./

        # Install dependencies
        RUN yarn install

        # Copy the rest of the application code to the working directory
        COPY . .

        # DB migration
        RUN npm run db:generate

        #Build
        RUN npm run build

        # Expose the port the app runs on
        EXPOSE 3000

        # Define environment variables (optional, adjust as needed)
        ENV PORT=3000
        ENV NODE_ENV=production

        # Start the application
        CMD ["npm", "start"]

8. Created dockerhub public repo "bhuwan404/SAMPLE_NODE_DEV"

9. Pushed to dockerhub
    a. Commands:
        git init
        git branch -M main
        git add .
        git commit -m "Initial Commit"
        git remote add origin https://github.com/bhuwan404/SAMPLE_NODE_DEV.git
        git push -u origin main

10. Created github secret for DockerHub Username and Password

11. Created github action workflows "ci.yaml" for NodeJS
        mkdir -p .github/workflows
        vi .github/workflows/ci.yaml
            name: Node.js CI

            on:
            push:
                branches: [ "main" ]
            #pull_request:
            # branches: [ "main" ]

            jobs:
            build:

                runs-on: ubuntu-latest
                steps:
                - name: check repository
                    uses: actions/checkout@v4

                - name: login to docker registry
                    uses: docker/login-action@v3
                    with:
                    username: ${{secrets.DOCKER_USERNAME}}
                    password: ${{secrets.DOCKER_PASSWORD}}

                - name: build and push docker image to registry
                    uses: docker/build-push-action@v5
                    with:
                    context: .
                    push: true
                    tags: bhuwan405/sample-node-dev:${{ github.run_number }}
    
11. Pushed Dockerfile and workflow file to github
    a. Command:
        git add .
        git commit -m "Updated for workflow - ci.yaml file"
        git push

12. Created docker-compose file in EC2 instance
    a. docker-compose
        version: '3.8'

        services:
        app:
            container_name: sample-node
            image: docker.io/bhuwan405/sample_node_dev:6
            ports:
            - "80:3000"
            environment:
            - NODE_ENV=development
            - HOST=0.0.0.0
            - PORT=3000
            - POSTGRES_DB_CONNECTION_URL=postgresql://test:test123@172.31.89.82:5432/test_db

13. Started docker-compose file
    a. Command:
        docker-compose up

14. Swagger api is accessible in EC2 instance public ip and on 80 port
    a. Curl command output:
        curl -X 'POST'   'http://3.86.90.217:80/auth/login'   -H 'accept: application/json'   -H 'Content-Type: application/json'   -d '{
            "email": "dev.noveltytechnology.co",
            "password": "random"
        }'
        <!DOCTYPE html>
        <html lang="en">
        <head>
        <meta charset="utf-8">
        <title>Error</title>
        </head>
        <body>
        <pre>Cannot POST /auth/login</pre>
        </body>
        </html>



