***Projet 1: Use Docker for local development***

# Node.js + MongoDB Dockerized Application

## Overview
This is a demo DevOps project featuring a Node.js application with MongoDB and Mongo Express using Docker.

## Technologies
- Node.js
- MongoDB
- Mongo Express
- Docker
- Docker Compose

## Setup Instructions
1. Clone the repository.
2. Run `docker-compose up -d` to start the containers.
3. Access the application at `http://localhost:3000`.
4. Access Mongo Express at `http://localhost:8081`.

## Project Structure
- **Dockerfile**: Configures Docker for the Node.js app.
- **docker-compose.yml**: Manages the multi-container environment.

### INSTRUCTIONS ###

1. Install Docker
sudo dnf update -y
sudo dnf config-manager --add-repo=https://download.docker.com/linux/centos/docker-ce.repo
sudo dnf install docker-ce docker-ce-cli containerd.io -y
sudo systemctl start docker
sudo systemctl enable docker
sudo docker --version
sudo usermod -aG docker $USER

2. Install Docker Compose
sudo curl -L "https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

sudo chmod +x /usr/local/bin/docker-compose
docker-compose --version


Step 1: Create a Basic Node.js Application
1. Install Node.js
sudo dnf module install nodejs:14

2. Create a Project Directory:
mkdir demo-project && cd demo-project

3. Initialize a Node.js Project:
npm init -y

4. Install Dependencies: Install express (web framework) and mongoose (MongoDB ORM):
npm install express mongoose

5. Create a Basic Node.js Server: Create a file named server.js with the following content:
const express = require('express');
const mongoose = require('mongoose');

const app = express();
const PORT = process.env.PORT || 3000;

// Connect to MongoDB
mongoose.connect('mongodb://mongo:27017/mydatabase', {
    useNewUrlParser: true,
    useUnifiedTopology: true
}).then(() => {
    console.log('Connected to MongoDB');
}).catch(err => {
    console.error('MongoDB connection error:', err);
});

app.get('/', (req, res) => {
    res.send('Hello from Node.js with MongoDB!');
});

app.listen(PORT, () => {
    console.log(`Server is running on port ${PORT}`);
});


Step 2: Dockerize the Node.js Application
1. Create a Dockerfile in the project directory:

# Use Node.js version 16 or higher
FROM node:16

# Set working directory
WORKDIR /app

# Copy package.json and install dependencies
COPY package.json .
RUN npm install

# Copy the rest of the application code
COPY . .

# Expose the port the app runs on
EXPOSE 3000

# Start the app
CMD ["node", "server.js"]


2. Build the Docker Image:
  docker build -t node-mongo-app .


Step 3: Set Up Docker Compose
1. Create a docker-compose.yml File in the project directory:

version: '3'
services:
  app:
    build:
      context: .
      dockerfile: Dockerfile
    ports:
      - "3000:3000"
    depends_on:
      - mongo
    environment:
      - MONGO_URL=mongodb://mongo:27017/mydatabase
    networks:
      - mynetwork

  mongo:
    image: mongo
    ports:
      - "27017:27017"
    networks:
      - mynetwork
    volumes:
      - mongo-data:/data/db
    environment:
      - MONGO_INITDB_ROOT_USERNAME=admin
      - MONGO_INITDB_ROOT_PASSWORD=password

  mongo-express:
    image: mongo-express
    ports:
      - "8081:8081"
    depends_on:
      - mongo
    environment:
      - ME_CONFIG_MONGODB_SERVER=mongo
      - ME_CONFIG_MONGODB_PORT=27017
      - ME_CONFIG_MONGODB_ADMINUSERNAME=admin
      - ME_CONFIG_MONGODB_ADMINPASSWORD=password
      - ME_CONFIG_BASICAUTH_USERNAME=admin
      - ME_CONFIG_BASICAUTH_PASSWORD=password

    networks:
      - mynetwork

networks:
  mynetwork:

volumes:
  mongo-data:

This file defines:

app: the Node.js application.
mongo: MongoDB container with a volume for persistent data.
mongo-express: MongoDB admin UI accessible at localhost:8081.


Step 4:  open ports 3000 and 8081

sudo firewall-cmd --zone=public --add-port=3000/tcp --permanent
sudo firewall-cmd --zone=public --add-port=8081/tcp --permanent
sudo firewall-cmd --reload

Step 5: Configure Docker to Use a Stable DNS Server

create /etc/docker/daemon.json and add the DNS settings:

{
  "dns": ["8.8.8.8", "8.8.4.4"]
}

Step 5: Run and Test Locally

1. Start All Containers:
docker-compose up -d

2. Check the Application:
Open a browser and go to http://localhost:3000. You should see the “Hello from Node.js with MongoDB!” message.

To check the Mongo Express UI, go to http://localhost:8081.

3. Verify Container Status:

Run docker ps to see if all containers are running correctly.
