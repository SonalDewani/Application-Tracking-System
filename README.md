# Application-Tracking-System

1️⃣ Folder Structure
microservices-demo/
│
├── customer-service/
│   ├── src/app.js
│   ├── package.json
│   └── Dockerfile
│
├── item-service/
│   ├── src/app.js
│   ├── package.json
│   └── Dockerfile
│
├── nginx/
│   └── nginx.conf
│
└── docker-compose.yml

-----------------------------------

2️⃣ customer-service
customer-service/src/app.js
const express = require("express");
const mongoose = require("mongoose");

const app = express();
app.use(express.json());

mongoose.connect("mongodb://mongo:27017/customersdb", {
  useNewUrlParser: true,
  useUnifiedTopology: true
});

const Customer = mongoose.model("Customer", {
  name: String,
  age: Number
});

// Get all customers
app.get("/customers", async (req, res) => {
  const customers = await Customer.find();
  res.json(customers);
});

// Create a customer
app.post("/customers", async (req, res) => {
  const newCustomer = new Customer(req.body);
  await newCustomer.save();
  res.json(newCustomer);
});

app.listen(3002, () => console.log("Customer Service running on port 3002"));

-----------------------------------

customer-service/Dockerfile
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
EXPOSE 3002
CMD ["node", "src/app.js"]

-----------------------------------

3️⃣ item-service
item-service/src/app.js
const express = require("express");
const mongoose = require("mongoose");

const app = express();
app.use(express.json());

mongoose.connect("mongodb://mongo:27017/itemsdb", {
  useNewUrlParser: true,
  useUnifiedTopology: true
});

const Item = mongoose.model("Item", {
  name: String,
  qty: Number,
  rate: Number
});

// Get all items
app.get("/items", async (req, res) => {
  const items = await Item.find();
  res.json(items);
});

// Create item
app.post("/items", async (req, res) => {
  const newItem = new Item(req.body);
  await newItem.save();
  res.json(newItem);
});

app.listen(3001, () => console.log("Item Service running on port 3001"));

-----------------------------------

item-service/Dockerfile
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
EXPOSE 3001
CMD ["node", "src/app.js"]

-----------------------------------

4️⃣ NGINX Configuration
nginx/nginx.conf
# Upstreams
upstream item_service {
    server item-service:3001;
}

upstream customer_service {
    server customer-service:3002;
}

server {
    listen 80;

    location /items {
        proxy_pass http://item_service;
    }

    location /customers {
        proxy_pass http://customer_service;
    }
}

-----------------------------------

5️⃣ Docker Compose
docker-compose.yml
version: "3.9"
services:
  mongo:
    image: mongo:6
    container_name: mongo
    ports:
      - "27017:27017"
    volumes:
      - mongo-data:/data/db

  item-service:
    build: ./item-service
    container_name: item-service
    depends_on:
      - mongo
    ports:
      - "3001:3001"

  customer-service:
    build: ./customer-service
    container_name: customer-service
    depends_on:
      - mongo
    ports:
      - "3002:3002"

  nginx:
    image: nginx:alpine
    container_name: nginx
    ports:
      - "8080:80"
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf:ro
    depends_on:
      - item-service
      - customer-service

volumes:
  mongo-data:

-----------------------------------

6️⃣ Running the Setup
docker-compose up --build
•	NGINX listens on localhost:8080
•	Routes:
•	http://localhost:8080/items → forwards to Item-Service (3001)
•	http://localhost:8080/customers → forwards to Customer-Service (3002)

-----------------------------------
