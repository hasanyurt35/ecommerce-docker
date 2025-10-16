# Docker Office Hours - E-Commerce Product Catalog

## Purpose
This hands-on training is designed to reinforce Docker concepts learned in the previous sessions by building a real-world multi-container application. Wel will implement a complete e-commerce product catalog system using Docker, Docker Compose, and best practices.

## Learning Outcomes

At the end of this hands-on training, we will be able to:

- Design and implement multi-container applications
- Use Docker Compose to orchestrate multiple services
- Implement container networking and service communication
- Configure persistent data storage with volumes
- Manage environment variables and application configuration
- Build production-ready Docker images
- Implement caching strategies with Redis
- Debug and troubleshoot containerized applications
- Apply Docker best practices and security principles

## Project Overview

### What We're Building

A **Product Catalog System** with the following features:
- View all products
- View single product details
- Add new products
- Update existing products
- Delete products
- Redis caching for improved performance

### Technology Stack

- **Frontend**: React (simple UI for product management)
- **Backend**: Flask (RESTful API)
- **Database**: PostgreSQL (persistent data storage)
- **Cache**: Redis (performance optimization)
- **Orchestration**: Docker Compose

## Architecture

```                       AWS EC2-INSTANCE
┌─────────────────────────────────────────────────────────────┐
│                        Docker Bridge Network                │
│                                                             │
│  ┌──────────────┐      ┌──────────────┐                     │
│  │   Frontend   │      │   Backend    │                     │
│  │   (React)    │◄────►│   (Flask)    │                     │
│  │   Port 3000  │      │   Port 5000  │                     │
│  └──────────────┘      └───────┬──────┘                     │
│                                 │                           │
│                        ┌────────┴────────┐                  │
│                        │                 │                  │
│                 ┌──────▼──────┐   ┌─────▼──────┐            │
│                 │  PostgreSQL │   │   Redis    │            │
│                 │  (Database) │   │  (Cache)   │            │
│                 │  Port 5432  │   │  Port 6379 │            │
│                 └─────────────┘   └────────────┘            │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### Service Details

1. **Frontend Service**
   - Technology: React
   - Purpose: User interface for product management
   - Exposed Port: 3000
   - Dependencies: Backend API

2. **Backend Service**
   - Technology: Flask (Python)
   - Purpose: RESTful API for CRUD operations
   - Exposed Port: 5000
   - Dependencies: PostgreSQL, Redis
   - Features:
     - Product CRUD operations
     - Redis caching for GET requests
     - Database connection pooling
     - Error handling

3. **PostgreSQL Service**
   - Purpose: Persistent data storage
   - Internal Port: 5432 (not exposed to host)
   - Features:
     - Data persistence with volumes
     - Initial data seeding
     - Connection from backend only

4. **Redis Service**
   - Purpose: Caching layer
   - Internal Port: 6379 (not exposed to host)
   - Features:
     - Cache product listings
     - TTL (Time To Live) configuration
     - Cache invalidation on updates

## API Endpoints for Backend Service

| Method | Endpoint | Description | Cache |
|--------|----------|-------------|-------|
| GET | /api/products | Get all products | Yes |
| GET | /api/products/:id | Get single product | Yes |
| POST | /api/products | Create new product | Invalidates cache |
| PUT | /api/products/:id | Update product | Invalidates cache |
| DELETE | /api/products/:id | Delete product | Invalidates cache |
| GET | /api/health | Health check | No |

## Project Structure

```
docker_office_hours_new/
├── README.md                           # This file
├── docker-compose.yml                  # Multi-container orchestration
├── .env.example                        # Environment variables template
├── .gitignore                          # Git ignore file
│
├── backend/
│   ├── Dockerfile                      # Backend image definition
│   ├── requirements.txt                # Python dependencies
│   ├── app/
│   │   ├── __init__.py                # Flask app initialization
│   │   ├── main.py                    # Application entry point
│   │   ├── models.py                  # Database models
│   │   ├── routes.py                  # API routes
│   │   └── cache.py                   # Redis cache utilities
│
├── frontend/
│   ├── Dockerfile                      # Frontend image definition
│   ├── package.json                    # Node.js dependencies
│   ├── public/
│   │   └── index.html                 # HTML template
│   └── src/
│       ├── App.js                     # Main React component
│       ├── index.js                   # React entry point
│       ├── ProductList.js             # Product listing component
│       └── ProductForm.js             # Add/Edit product form
│
└── database/
    └── init.sql                        # Database initialization script
```

## Hands-On Implementation Guide

### Session Timeline (3 Hours)

**Hour 1: Setup & Architecture (60 minutes)**
- Part 1: Project Introduction (10 min)
- Part 2: EC2 Instance Setup (10 min)
- Part 3: Project Structure Creation (15 min)
- Part 4: Backend Service Implementation (25 min)

**Hour 2: Integration & Deployment (60 minutes)**
- Part 5: Database Setup (15 min)
- Part 6: Redis Cache Implementation (15 min)
- Part 7: Frontend Development (20 min)
- Part 8: Docker Compose Configuration (10 min)

**Hour 3: Production & Best Practices (60 minutes)**
- Part 9: Running the Application (15 min)
- Part 10: Testing & Debugging (15 min)
- Part 11: Production Best Practices (15 min)
- Part 12: Q&A and Troubleshooting (15 min)

---

## Part 1 - Project Introduction (10 minutes)

### Overview
Explain the architecture and what we're building.

**Key Points to Cover:**
1. Multi-container application concept
2. Microservices architecture basics
3. Why we use each technology
4. Real-world use cases

**Discussion Topics:**
- How this relates to real production applications
- Scalability considerations
- Why separate frontend and backend

---

## Part 2 - Launch EC2 Instance and Setup (10 minutes)

### Launch EC2 Instance

- Launch a Docker-enabled EC2 instance on Amazon Linux 2023 AMI with security group allowing SSH (port 22), HTTP (port 3000), and API (port 5000) connections.

**Security Group Rules:**
```
Type        Protocol    Port Range    Source         Description
SSH         TCP         22           0.0.0.0/0      SSH access
Custom TCP  TCP         3000         0.0.0.0/0      Frontend
Custom TCP  TCP         5000         0.0.0.0/0      Backend API
```

### Connect to Instance

```bash
ssh -i your-key.pem ec2-user@<your-ec2-public-ip>
```

### Install Docker and Docker Compose

**Step 1: Install Docker**

```bash
# Update package manager
sudo dnf update -y

# Install Docker
sudo dnf install docker -y

# Start Docker service
sudo systemctl start docker

# Enable Docker to start on boot
sudo systemctl enable docker

# Add ec2-user to docker group (to run docker without sudo)
sudo usermod -a -G docker ec2-user

# Apply group changes (logout and login, or use this command)
newgrp docker

# Verify Docker installation
docker --version
```

**Step 2: Install Docker Compose**

```bash
# Download Docker Compose
sudo curl -SL https://github.com/docker/compose/releases/download/v2.24.0/docker-compose-linux-x86_64 -o /usr/local/bin/docker-compose

# Make it executable
sudo chmod +x /usr/local/bin/docker-compose

# Verify installation
docker-compose --version
```

**Verify both are working:**

```bash
docker ps
docker-compose version
```

---

## Part 3 - Create Project Structure (15 minutes)

### Create Base Directory Structure

```bash
# Create project root
mkdir -p ~/ecommerce-docker
cd ~/ecommerce-docker

# Create service directories
mkdir -p backend/app
mkdir -p frontend/src/components
mkdir -p frontend/public
mkdir -p database
```

### Verify Structure

```bash
sudo dnf install tree -y
tree -L 2
```

**Expected Output:**
```
ecommerce-docker/
├── backend
│   └── app
├── database
└── frontend
    ├── public
    └── src
```

---

## Part 4 - Backend Service Implementation (25 minutes)

### Step 1: Create Backend Dependencies

Create `backend/requirements.txt`:

> **What this file does:** Lists all Python packages needed by the Flask backend (web framework, database driver, Redis client, CORS support, and environment variable manager).

```bash
cat > backend/requirements.txt << 'EOF'
Flask==3.0.0
Flask-CORS==4.0.0
psycopg2-binary==2.9.9
redis==5.0.1
python-dotenv==1.0.0
EOF
```

**Explain each dependency:**
- `Flask`: Web framework
- `Flask-CORS`: Flask-CORS enables cross-origin requests between frontend and backend on different ports - browsers block these by default for security.
- `psycopg2-binary`: PostgreSQL database driver that allows Python code to connect to PostgreSQL, execute SQL queries, and retrieve results.
- `redis`: Redis client library for Python - enables connecting to Redis cache server, storing/retrieving data, and setting TTL (Time To Live) for cached values.
- `python-dotenv`: Environment variable management loads environment variables from .env file into Python - keeps sensitive data (passwords, API keys) out of code and enables easy configuration per
  environment.

### Step 2: Create Database Models

Create `backend/app/models.py`:

> **What this file does:** Defines the Product data model (class) with methods to convert between Python objects and JSON format for API responses.

```bash
cat > backend/app/models.py << 'EOF'
"""
Database models for the e-commerce application
"""

class Product:
    """Product model representing items in the catalog"""

    def __init__(self, id=None, name=None, description=None, price=None, stock=None):
        self.id = id
        self.name = name
        self.description = description
        self.price = price
        self.stock = stock

    def to_dict(self):
        """Convert product to dictionary format"""
        return {
            'id': self.id,
            'name': self.name,
            'description': self.description,
            'price': float(self.price) if self.price else None,
            'stock': self.stock
        }

    @staticmethod
    def from_dict(data):
        """Create product from dictionary"""
        return Product(
            id=data.get('id'),
            name=data.get('name'),
            description=data.get('description'),
            price=data.get('price'),
            stock=data.get('stock')
        )
EOF
```

### Step 3: Create Cache Utilities

Create `backend/app/cache.py`:

> **What this file does:** Contains helper functions to store, retrieve, and delete product data in Redis cache with automatic expiration (TTL).

```bash
cat > backend/app/cache.py << 'EOF'
"""
Redis cache utilities
"""
import json
import redis
import os

# Redis connection
redis_client = redis.Redis(
    host=os.getenv('REDIS_HOST', 'redis'),
    port=int(os.getenv('REDIS_PORT', 6379)),
    db=0,
    decode_responses=True
)

def get_cached_products():
    """Get products from cache"""
    try:
        cached = redis_client.get('products:all')
        if cached:
            return json.loads(cached)
        return None
    except Exception as e:
        print(f"Cache error: {e}")
        return None

def set_cached_products(products, ttl=300):
    """Cache products with TTL (default 5 minutes)"""
    try:
        redis_client.setex(
            'products:all',
            ttl,
            json.dumps(products)
        )
    except Exception as e:
        print(f"Cache error: {e}")

def invalidate_products_cache():
    """Invalidate products cache"""
    try:
        redis_client.delete('products:all')
    except Exception as e:
        print(f"Cache error: {e}")

def get_cached_product(product_id):
    """Get single product from cache"""
    try:
        cached = redis_client.get(f'product:{product_id}')
        if cached:
            return json.loads(cached)
        return None
    except Exception as e:
        print(f"Cache error: {e}")
        return None

def set_cached_product(product_id, product, ttl=300):
    """Cache single product"""
    try:
        redis_client.setex(
            f'product:{product_id}',
            ttl,
            json.dumps(product)
        )
    except Exception as e:
        print(f"Cache error: {e}")
EOF
```

**Explain caching strategy:**

Redis caching improves performance by storing frequently accessed data in memory:

1. **TTL (Time To Live): 5 minutes**
   - Cached data automatically expires after 5 minutes (300 seconds)
   - First request fetches from PostgreSQL and stores in Redis
   - Subsequent requests within 5 minutes are served from Redis (much faster!)
   - After 5 minutes, cache expires and next request fetches fresh data from database
   - Example: If you fetch products at 2:00 PM, Redis serves it instantly until 2:05 PM

2. **Cache Invalidation on Write Operations**
   - When you CREATE, UPDATE, or DELETE a product, the cache is immediately cleared
   - This ensures users always see the most up-to-date data after changes
   - Without invalidation, users might see old data for up to 5 minutes
   - Example: Add new product → cache cleared → next GET request fetches fresh data from database

3. **Separate Cache Keys**
   - `products:all` - stores the complete list of all products
   - `product:1`, `product:2`, etc. - stores individual product details
   - Separate keys allow independent caching and invalidation
   - Example: If product #5 is updated, only `product:5` needs to be invalidated, not the entire list

**Test cache behavior:**
```bash
# First request - from database (slower)
curl http://localhost:5000/api/products
# Response includes: "source": "database"

# Second request - from cache (faster!)
curl http://localhost:5000/api/products
# Response includes: "source": "cache"

# Create a new product - invalidates cache
curl -X POST http://localhost:5000/api/products \
  -H "Content-Type: application/json" \
  -d '{"name":"New Item","description":"Test","price":25.00,"stock":5}'

# Next request - from database again (cache was cleared)
curl http://localhost:5000/api/products
# Response includes: "source": "database"
```

### Step 4: Create API Routes

Create `backend/app/routes.py`:

> **What this file does:** Implements all REST API endpoints (GET, POST, PUT, DELETE) for product management with PostgreSQL database operations and Redis caching logic.

```bash
cat > backend/app/routes.py << 'EOF'
"""
API routes for product management
"""
from flask import Blueprint, request, jsonify
import psycopg2
import os
from .models import Product
from . import cache

api = Blueprint('api', __name__)

def get_db_connection():
    """Create database connection"""
    conn = psycopg2.connect(
        host=os.getenv('DB_HOST', 'postgres'),
        database=os.getenv('DB_NAME', 'ecommerce'),
        user=os.getenv('DB_USER', 'postgres'),
        password=os.getenv('DB_PASSWORD', 'postgres')
    )
    return conn

@api.route('/health', methods=['GET'])
def health_check():
    """Health check endpoint"""
    return jsonify({'status': 'healthy', 'service': 'backend'}), 200

@api.route('/products', methods=['GET'])
def get_products():
    """Get all products with caching"""
    # Check cache first
    cached_products = cache.get_cached_products()
    if cached_products:
        return jsonify({
            'products': cached_products,
            'source': 'cache'
        }), 200

    # If not in cache, get from database
    conn = get_db_connection()
    cursor = conn.cursor()
    cursor.execute('SELECT id, name, description, price, stock FROM products ORDER BY id')
    rows = cursor.fetchall()
    cursor.close()
    conn.close()

    products = []
    for row in rows:
        product = Product(
            id=row[0],
            name=row[1],
            description=row[2],
            price=row[3],
            stock=row[4]
        )
        products.append(product.to_dict())

    # Cache the results
    cache.set_cached_products(products)

    return jsonify({
        'products': products,
        'source': 'database'
    }), 200

@api.route('/products/<int:product_id>', methods=['GET'])
def get_product(product_id):
    """Get single product by ID"""
    # Check cache
    cached_product = cache.get_cached_product(product_id)
    if cached_product:
        return jsonify({
            'product': cached_product,
            'source': 'cache'
        }), 200

    conn = get_db_connection()
    cursor = conn.cursor()
    cursor.execute(
        'SELECT id, name, description, price, stock FROM products WHERE id = %s',
        (product_id,)
    )
    row = cursor.fetchone()
    cursor.close()
    conn.close()

    if row is None:
        return jsonify({'error': 'Product not found'}), 404

    product = Product(
        id=row[0],
        name=row[1],
        description=row[2],
        price=row[3],
        stock=row[4]
    )

    product_dict = product.to_dict()
    cache.set_cached_product(product_id, product_dict)

    return jsonify({
        'product': product_dict,
        'source': 'database'
    }), 200

@api.route('/products', methods=['POST'])
def create_product():
    """Create new product"""
    data = request.get_json()

    # Validation
    required_fields = ['name', 'description', 'price', 'stock']
    for field in required_fields:
        if field not in data:
            return jsonify({'error': f'{field} is required'}), 400

    conn = get_db_connection()
    cursor = conn.cursor()
    cursor.execute(
        'INSERT INTO products (name, description, price, stock) VALUES (%s, %s, %s, %s) RETURNING id',
        (data['name'], data['description'], data['price'], data['stock'])
    )
    product_id = cursor.fetchone()[0]
    conn.commit()
    cursor.close()
    conn.close()

    # Invalidate cache
    cache.invalidate_products_cache()

    return jsonify({
        'message': 'Product created successfully',
        'product_id': product_id
    }), 201

@api.route('/products/<int:product_id>', methods=['PUT'])
def update_product(product_id):
    """Update existing product"""
    data = request.get_json()

    conn = get_db_connection()
    cursor = conn.cursor()

    # Check if product exists
    cursor.execute('SELECT id FROM products WHERE id = %s', (product_id,))
    if cursor.fetchone() is None:
        cursor.close()
        conn.close()
        return jsonify({'error': 'Product not found'}), 404

    # Update product
    cursor.execute(
        'UPDATE products SET name = %s, description = %s, price = %s, stock = %s WHERE id = %s',
        (data['name'], data['description'], data['price'], data['stock'], product_id)
    )
    conn.commit()
    cursor.close()
    conn.close()

    # Invalidate cache
    cache.invalidate_products_cache()

    return jsonify({'message': 'Product updated successfully'}), 200

@api.route('/products/<int:product_id>', methods=['DELETE'])
def delete_product(product_id):
    """Delete product"""
    conn = get_db_connection()
    cursor = conn.cursor()

    cursor.execute('DELETE FROM products WHERE id = %s', (product_id,))
    rows_deleted = cursor.rowcount
    conn.commit()
    cursor.close()
    conn.close()

    if rows_deleted == 0:
        return jsonify({'error': 'Product not found'}), 404

    # Invalidate cache
    cache.invalidate_products_cache()

    return jsonify({'message': 'Product deleted successfully'}), 200
EOF
```

**Explain each endpoint:**
- GET /products: List with cache
- GET /products/:id: Single item with cache
- POST /products: Create and invalidate cache
- PUT /products/:id: Update and invalidate cache
- DELETE /products/:id: Delete and invalidate cache

### Step 5: Create Flask Application

Create `backend/app/__init__.py`:

> **What this file does:** Initializes the Flask application with CORS enabled and registers the API routes blueprint.

```bash
cat > backend/app/__init__.py << 'EOF'
"""
Flask application factory
"""
from flask import Flask
from flask_cors import CORS

def create_app():
    app = Flask(__name__)
    CORS(app)  # Enable CORS for frontend communication

    # Register blueprints
    from .routes import api
    app.register_blueprint(api, url_prefix='/api')

    return app
EOF
```

Create `backend/app/main.py`:

> **What this file does:** The entry point that creates and runs the Flask application on port 5000 for all network interfaces.

```bash
cat > backend/app/main.py << 'EOF'
"""
Application entry point
"""
from . import create_app

app = create_app()

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000, debug=True)
EOF
```

### Step 6: Create Backend Dockerfile

Create `backend/Dockerfile`:

> **What this file does:** Defines how to build the backend Docker image - installs Python dependencies, copies application code, and sets up the container to run Flask on port 5000.

```bash
cat > backend/Dockerfile << 'EOF'
# Use Python 3.11 slim image for smaller size (50MB)
FROM python:3.11-slim

# Set working directory (/app folder will be created automatically and changed to /app folder)
WORKDIR /app

# Set environment variables
ENV PYTHONDONTWRITEBYTECODE=1 \
    PYTHONUNBUFFERED=1

# Install system dependencies
RUN apt-get update && apt-get install -y --no-install-recommends \
    gcc \
    postgresql-client \
    && rm -rf /var/lib/apt/lists/*

# Copy requirements first for better caching and fast build.
COPY requirements.txt .

# Install Python dependencies
RUN pip install --no-cache-dir -r requirements.txt

# Copy application code
COPY app/ ./app/

# Expose port
EXPOSE 5000

# Run the application
CMD ["python", "-m", "app.main"]
EOF
```

---

## Part 5 - Database Setup (15 minutes)

### Step 1: Create Database Initialization Script

Create `database/init.sql`:

> **What this file does:** SQL script that runs automatically when PostgreSQL container starts - creates products table, adds indexes for performance, inserts sample data, and sets up auto-update triggers.

```bash
cat > database/init.sql << 'EOF'
-- Create products table
CREATE TABLE IF NOT EXISTS products (
    id SERIAL PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    description TEXT,
    price DECIMAL(10, 2) NOT NULL,
    stock INTEGER NOT NULL DEFAULT 0,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Create index on name for faster searches
CREATE INDEX IF NOT EXISTS idx_products_name ON products(name);

-- Insert sample data
INSERT INTO products (name, description, price, stock) VALUES
    ('Laptop', 'High-performance laptop with 16GB RAM', 1299.99, 15),
    ('Wireless Mouse', 'Ergonomic wireless mouse with USB receiver', 29.99, 50),
    ('USB-C Hub', '7-in-1 USB-C hub with HDMI and card reader', 49.99, 30),
    ('Mechanical Keyboard', 'RGB mechanical keyboard with blue switches', 89.99, 20),
    ('Monitor 27"', '4K UHD monitor with HDR support', 399.99, 10),
    ('Webcam HD', '1080p webcam with built-in microphone', 79.99, 25),
    ('Laptop Stand', 'Adjustable aluminum laptop stand', 39.99, 40),
    ('Headphones', 'Noise-cancelling wireless headphones', 199.99, 18),
    ('External SSD', '1TB portable external SSD', 129.99, 35),
    ('Phone Stand', 'Adjustable phone stand for desk', 19.99, 60);

-- Create function to update updated_at timestamp
CREATE OR REPLACE FUNCTION update_updated_at_column()
RETURNS TRIGGER AS $$
BEGIN
    NEW.updated_at = CURRENT_TIMESTAMP;
    RETURN NEW;
END;
$$ language 'plpgsql';

-- Create trigger to automatically update updated_at
CREATE TRIGGER update_products_updated_at BEFORE UPDATE
    ON products FOR EACH ROW
    EXECUTE FUNCTION update_updated_at_column();
EOF
```

**Explain database setup:**
- Table schema with proper data types
- Indexes for performance
- Sample data for testing
- Timestamps for audit trail
- Triggers for automatic updates

### Step 2: Understand Database Service Configuration

**Discussion points:**
- PostgreSQL official image
- Volume for data persistence
- Environment variables for configuration
- Internal networking (not exposed to host)
- Health checks

---

## Part 6 - Redis Cache Implementation (15 minutes)

### Understanding Redis Configuration

**Discuss:**
- Redis official image
- Why we use Alpine (smaller size)
- No persistent storage needed (cache is temporary)
- Internal networking
- Connection from backend only

### Testing Cache Strategy

**Explain the caching flow:**

1. **First Request (Cache Miss):**
   ```
   Browser → Backend → Check Redis → Not found → Query PostgreSQL → Save to Redis → Return data
   ```

2. **Subsequent Requests (Cache Hit):**
   ```
   Browser → Backend → Check Redis → Found → Return cached data (faster!)
   ```

3. **Write Operations (Cache Invalidation):**
   ```
   Browser → Backend → Create/Update/Delete in PostgreSQL → Clear Redis cache → Return success
   ```

**Benefits:**
- Reduced database load
- Faster response times
- Scalability improvement

---

## Part 7 - Frontend Development (20 minutes)

### Step 1: Create Package.json

Create `frontend/package.json`:

> **What this file does:** Defines the React frontend project metadata, lists required npm packages (React, ReactDOM, build scripts), and configures browser compatibility settings.

```bash
cat > frontend/package.json << 'EOF'
{
  "name": "ecommerce-frontend",
  "version": "1.0.0",
  "description": "E-commerce product catalog frontend",
  "private": true,
  "dependencies": {
    "react": "^18.2.0",
    "react-dom": "^18.2.0",
    "react-scripts": "5.0.1"
  },
  "scripts": {
    "start": "react-scripts start",
    "build": "react-scripts build",
    "test": "react-scripts test",
    "eject": "react-scripts eject"
  },
  "eslintConfig": {
    "extends": [
      "react-app"
    ]
  },
  "browserslist": {
    "production": [
      ">0.2%",
      "not dead",
      "not op_mini all"
    ],
    "development": [
      "last 1 chrome version",
      "last 1 firefox version",
      "last 1 safari version"
    ]
  }
}
EOF
```

### Step 2: Create HTML Template

Create `frontend/public/index.html`:

> **What this file does:** The main HTML template that serves as the container for the React application - includes basic styling and a root div where React components will be rendered. 

```bash
cat > frontend/public/index.html << 'EOF'
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1" />
    <meta name="description" content="E-commerce Product Catalog" />
    <title>Product Catalog</title>
    <style>
      * {
        margin: 0;
        padding: 0;
        box-sizing: border-box;
      }
      body {
        font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', 'Roboto', 'Oxygen',
          'Ubuntu', 'Cantarell', 'Fira Sans', 'Droid Sans', 'Helvetica Neue',
          sans-serif;
        -webkit-font-smoothing: antialiased;
        -moz-osx-font-smoothing: grayscale;
        background-color: #f5f5f5;
      }
    </style>
  </head>
  <body>
    <noscript>You need to enable JavaScript to run this app.</noscript>
    <div id="root"></div>
  </body>
</html>
EOF
```

### Step 3: Create React Components

Create `frontend/src/index.js`:

> **What this file does:** React entry point that initializes the app and renders the main App component into the HTML root element.

```bash
cat > frontend/src/index.js << 'EOF'
import React from 'react';
import ReactDOM from 'react-dom/client';
import App from './App';

const root = ReactDOM.createRoot(document.getElementById('root'));
root.render(
  <React.StrictMode>
    <App />
  </React.StrictMode>
);
EOF
```

Create `frontend/src/App.js`:

> **What this file does:** Main React component that manages product state, handles API calls to backend (fetch, create, update, delete), and renders the product form and list components.

```bash
cat > frontend/src/App.js << 'EOF'
import React, { useState, useEffect } from 'react';
import ProductList from './components/ProductList';
import ProductForm from './components/ProductForm';
import './App.css';

const API_URL = process.env.REACT_APP_API_URL || 'http://localhost:5000/api';

function App() {
  const [products, setProducts] = useState([]);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);
  const [editingProduct, setEditingProduct] = useState(null);
  const [cacheSource, setCacheSource] = useState('');

  const fetchProducts = async () => {
    try {
      setLoading(true);
      const response = await fetch(`${API_URL}/products`);
      if (!response.ok) throw new Error('Failed to fetch products');
      const data = await response.json();
      setProducts(data.products);
      setCacheSource(data.source);
      setError(null);
    } catch (err) {
      setError(err.message);
    } finally {
      setLoading(false);
    }
  };

  useEffect(() => {
    fetchProducts();
  }, []);

  const handleDelete = async (id) => {
    if (!window.confirm('Are you sure you want to delete this product?')) return;

    try {
      const response = await fetch(`${API_URL}/products/${id}`, {
        method: 'DELETE',
      });
      if (!response.ok) throw new Error('Failed to delete product');
      await fetchProducts();
    } catch (err) {
      setError(err.message);
    }
  };

  const handleEdit = (product) => {
    setEditingProduct(product);
  };

  const handleSave = async (product) => {
    try {
      const url = product.id
        ? `${API_URL}/products/${product.id}`
        : `${API_URL}/products`;

      const method = product.id ? 'PUT' : 'POST';

      const response = await fetch(url, {
        method,
        headers: {
          'Content-Type': 'application/json',
        },
        body: JSON.stringify(product),
      });

      if (!response.ok) throw new Error('Failed to save product');

      setEditingProduct(null);
      await fetchProducts();
    } catch (err) {
      setError(err.message);
    }
  };

  const handleCancel = () => {
    setEditingProduct(null);
  };

  return (
    <div className="App">
      <header className="App-header">
        <h1>Product Catalog</h1>
        <p className="subtitle">Docker Multi-Container Application</p>
      </header>

      <main className="container">
        {error && (
          <div className="error-message">
            Error: {error}
          </div>
        )}

        {cacheSource && (
          <div className={`cache-indicator ${cacheSource}`}>
            Data source: {cacheSource.toUpperCase()}
            {cacheSource === 'cache' && ' ⚡'}
          </div>
        )}

        <ProductForm
          product={editingProduct}
          onSave={handleSave}
          onCancel={handleCancel}
        />

        {loading ? (
          <div className="loading">Loading products...</div>
        ) : (
          <ProductList
            products={products}
            onDelete={handleDelete}
            onEdit={handleEdit}
          />
        )}
      </main>
    </div>
  );
}

export default App;
EOF
```

Create `frontend/src/components/ProductList.js`:

> **What this file does:** React component that displays all products in a responsive grid layout with edit and delete buttons for each product.

```bash
mkdir -p frontend/src/components
cat > frontend/src/components/ProductList.js << 'EOF'
import React from 'react';

function ProductList({ products, onDelete, onEdit }) {
  return (
    <div className="product-list">
      <h2>Products ({products.length})</h2>
      {products.length === 0 ? (
        <p className="no-products">No products found. Add your first product above!</p>
      ) : (
        <div className="products-grid">
          {products.map((product) => (
            <div key={product.id} className="product-card">
              <div className="product-header">
                <h3>{product.name}</h3>
                <span className="product-price">${product.price}</span>
              </div>
              <p className="product-description">{product.description}</p>
              <div className="product-footer">
                <span className="product-stock">
                  Stock: {product.stock}
                </span>
                <div className="product-actions">
                  <button
                    onClick={() => onEdit(product)}
                    className="btn btn-edit"
                  >
                    Edit
                  </button>
                  <button
                    onClick={() => onDelete(product.id)}
                    className="btn btn-delete"
                  >
                    Delete
                  </button>
                </div>
              </div>
            </div>
          ))}
        </div>
      )}
    </div>
  );
}

export default ProductList;
EOF
```

Create `frontend/src/components/ProductForm.js`:

> **What this file does:** React form component that handles adding new products and editing existing ones - includes input validation and submit/cancel functionality.

```bash
cat > frontend/src/components/ProductForm.js << 'EOF'
import React, { useState, useEffect } from 'react';

function ProductForm({ product, onSave, onCancel }) {
  const [formData, setFormData] = useState({
    name: '',
    description: '',
    price: '',
    stock: ''
  });

  useEffect(() => {
    if (product) {
      setFormData(product);
    } else {
      setFormData({
        name: '',
        description: '',
        price: '',
        stock: ''
      });
    }
  }, [product]);

  const handleChange = (e) => {
    const { name, value } = e.target;
    setFormData(prev => ({
      ...prev,
      [name]: value
    }));
  };

  const handleSubmit = (e) => {
    e.preventDefault();
    onSave(formData);
  };

  const handleReset = () => {
    setFormData({
      name: '',
      description: '',
      price: '',
      stock: ''
    });
    if (product) {
      onCancel();
    }
  };

  return (
    <div className="product-form">
      <h2>{product ? 'Edit Product' : 'Add New Product'}</h2>
      <form onSubmit={handleSubmit}>
        <div className="form-group">
          <label htmlFor="name">Product Name *</label>
          <input
            type="text"
            id="name"
            name="name"
            value={formData.name}
            onChange={handleChange}
            required
            placeholder="Enter product name"
          />
        </div>

        <div className="form-group">
          <label htmlFor="description">Description *</label>
          <textarea
            id="description"
            name="description"
            value={formData.description}
            onChange={handleChange}
            required
            placeholder="Enter product description"
            rows="3"
          />
        </div>

        <div className="form-row">
          <div className="form-group">
            <label htmlFor="price">Price ($) *</label>
            <input
              type="number"
              id="price"
              name="price"
              value={formData.price}
              onChange={handleChange}
              required
              min="0"
              step="0.01"
              placeholder="0.00"
            />
          </div>

          <div className="form-group">
            <label htmlFor="stock">Stock *</label>
            <input
              type="number"
              id="stock"
              name="stock"
              value={formData.stock}
              onChange={handleChange}
              required
              min="0"
              placeholder="0"
            />
          </div>
        </div>

        <div className="form-actions">
          <button type="submit" className="btn btn-primary">
            {product ? 'Update Product' : 'Add Product'}
          </button>
          <button type="button" onClick={handleReset} className="btn btn-secondary">
            {product ? 'Cancel' : 'Reset'}
          </button>
        </div>
      </form>
    </div>
  );
}

export default ProductForm;
EOF
```

Create `frontend/src/App.css`:

> **What this file does:** Complete CSS stylesheet that styles the entire frontend - includes responsive design, gradient headers, card layouts, buttons, forms, and mobile-friendly media queries.

```bash
cat > frontend/src/App.css << 'EOF'
.App {
  min-height: 100vh;
}

.App-header {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: white;
  padding: 2rem;
  text-align: center;
  box-shadow: 0 2px 4px rgba(0,0,0,0.1);
}

.App-header h1 {
  margin: 0;
  font-size: 2.5rem;
}

.subtitle {
  margin: 0.5rem 0 0 0;
  opacity: 0.9;
  font-size: 1rem;
}

.container {
  max-width: 1200px;
  margin: 0 auto;
  padding: 2rem;
}

.error-message {
  background-color: #fee;
  border: 1px solid #fcc;
  color: #c33;
  padding: 1rem;
  border-radius: 4px;
  margin-bottom: 1rem;
}

.cache-indicator {
  background-color: #e3f2fd;
  border-left: 4px solid #2196f3;
  padding: 0.75rem;
  margin-bottom: 1rem;
  border-radius: 4px;
  font-weight: 500;
}

.cache-indicator.cache {
  background-color: #e8f5e9;
  border-left-color: #4caf50;
}

.loading {
  text-align: center;
  padding: 3rem;
  color: #666;
  font-size: 1.2rem;
}

/* Product Form */
.product-form {
  background: white;
  padding: 2rem;
  border-radius: 8px;
  box-shadow: 0 2px 8px rgba(0,0,0,0.1);
  margin-bottom: 2rem;
}

.product-form h2 {
  margin-top: 0;
  color: #333;
}

.form-group {
  margin-bottom: 1.5rem;
}

.form-group label {
  display: block;
  margin-bottom: 0.5rem;
  color: #555;
  font-weight: 500;
}

.form-group input,
.form-group textarea {
  width: 100%;
  padding: 0.75rem;
  border: 1px solid #ddd;
  border-radius: 4px;
  font-size: 1rem;
  font-family: inherit;
}

.form-group input:focus,
.form-group textarea:focus {
  outline: none;
  border-color: #667eea;
  box-shadow: 0 0 0 3px rgba(102, 126, 234, 0.1);
}

.form-row {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 1rem;
}

.form-actions {
  display: flex;
  gap: 1rem;
  margin-top: 1.5rem;
}

/* Buttons */
.btn {
  padding: 0.75rem 1.5rem;
  border: none;
  border-radius: 4px;
  font-size: 1rem;
  cursor: pointer;
  transition: all 0.3s ease;
  font-weight: 500;
}

.btn-primary {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: white;
}

.btn-primary:hover {
  transform: translateY(-2px);
  box-shadow: 0 4px 12px rgba(102, 126, 234, 0.4);
}

.btn-secondary {
  background: #e0e0e0;
  color: #333;
}

.btn-secondary:hover {
  background: #d0d0d0;
}

.btn-edit {
  background: #2196f3;
  color: white;
  padding: 0.5rem 1rem;
  font-size: 0.875rem;
}

.btn-edit:hover {
  background: #1976d2;
}

.btn-delete {
  background: #f44336;
  color: white;
  padding: 0.5rem 1rem;
  font-size: 0.875rem;
}

.btn-delete:hover {
  background: #d32f2f;
}

/* Product List */
.product-list {
  background: white;
  padding: 2rem;
  border-radius: 8px;
  box-shadow: 0 2px 8px rgba(0,0,0,0.1);
}

.product-list h2 {
  margin-top: 0;
  color: #333;
}

.no-products {
  text-align: center;
  padding: 3rem;
  color: #999;
}

.products-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(300px, 1fr));
  gap: 1.5rem;
  margin-top: 1.5rem;
}

.product-card {
  border: 1px solid #e0e0e0;
  border-radius: 8px;
  padding: 1.5rem;
  transition: all 0.3s ease;
}

.product-card:hover {
  box-shadow: 0 4px 12px rgba(0,0,0,0.15);
  transform: translateY(-4px);
}

.product-header {
  display: flex;
  justify-content: space-between;
  align-items: flex-start;
  margin-bottom: 1rem;
}

.product-header h3 {
  margin: 0;
  color: #333;
  flex: 1;
}

.product-price {
  color: #667eea;
  font-size: 1.25rem;
  font-weight: 700;
  margin-left: 1rem;
}

.product-description {
  color: #666;
  margin-bottom: 1rem;
  line-height: 1.5;
}

.product-footer {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding-top: 1rem;
  border-top: 1px solid #f0f0f0;
}

.product-stock {
  color: #666;
  font-size: 0.875rem;
}

.product-actions {
  display: flex;
  gap: 0.5rem;
}

/* Responsive */
@media (max-width: 768px) {
  .container {
    padding: 1rem;
  }

  .App-header h1 {
    font-size: 2rem;
  }

  .products-grid {
    grid-template-columns: 1fr;
  }

  .form-row {
    grid-template-columns: 1fr;
  }

  .form-actions {
    flex-direction: column;
  }

  .btn {
    width: 100%;
  }
}
EOF
```

### Step 4: Create Frontend Dockerfile

Create `frontend/Dockerfile`:

> **What this file does:** Dockerfile that builds the React app and serves the static files on port 3000.

```bash
cat > frontend/Dockerfile << 'EOF'
FROM node:18-alpine

WORKDIR /app

# Build-time argument (IMPORTANT: React embeds env vars during build!)
ARG REACT_APP_API_URL
ENV REACT_APP_API_URL=${REACT_APP_API_URL}

# Copy package files
COPY package*.json ./

# Install dependencies
RUN npm install --silent

# Copy source code
COPY public/ ./public/

COPY src/ ./src/

# Build the application
RUN npm run build

# Install serve to run the built app
RUN npm install -g serve

# Expose port
EXPOSE 3000

# Run the application
CMD ["serve", "-s", "build", "-l", "3000"]
EOF
```

**Explain Dockerfile:**
- Uses Node.js 18 Alpine (lightweight ~150MB base)
- Accepts build-time argument for API URL
- Installs dependencies and builds React app
- Uses 'serve' package to serve static files
- Simple single-stage build for learning purposes

**Note on Image Size:**
After building, check the image size:
```bash
docker images | grep frontend
```

You'll notice the image is relatively large (~450-500MB). This is because it contains:
- node_modules (~300MB)
- Source code (src/)
- Build artifacts (build/)
- Development dependencies

**Research Challenge:**
Can you research and implement a "multi-stage Dockerfile" to reduce this image size?
Multi-stage builds can reduce the image to ~150-200MB by keeping only the build artifacts in the final image.
This is a common production optimization technique!

---

## Part 8 - Docker Compose Configuration (10 minutes)

### Create Docker Compose File

Create `docker-compose.yml`:

> **What this file does:** Orchestrates all four containers (frontend, backend, PostgreSQL, Redis) - defines how they connect, their dependencies, health checks, environment variables, ports, and persistent storage.

```bash
cat > docker-compose.yml << 'EOF'
services:
  # PostgreSQL Database
  postgres:
    image: postgres:15-alpine
    container_name: ecommerce-db
    environment:
      POSTGRES_DB: ${DB_NAME:-ecommerce}
      POSTGRES_USER: ${DB_USER:-postgres}
      POSTGRES_PASSWORD: ${DB_PASSWORD:-postgres}
    volumes:
      - postgres_data:/var/lib/postgresql/data
      - ./database/init.sql:/docker-entrypoint-initdb.d/init.sql
    networks:
      - ecommerce-network
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 10s
      timeout: 5s
      retries: 5

  # Redis Cache
  redis:
    image: redis:7-alpine
    container_name: ecommerce-cache
    networks:
      - ecommerce-network
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 10s
      timeout: 5s
      retries: 5

  # Backend API
  backend:
    build:
      context: ./backend
      dockerfile: Dockerfile
    container_name: ecommerce-backend
    ports:
      - "5000:5000"
    environment:
      DB_HOST: postgres
      DB_NAME: ${DB_NAME:-ecommerce}
      DB_USER: ${DB_USER:-postgres}
      DB_PASSWORD: ${DB_PASSWORD:-postgres}
      REDIS_HOST: redis
      REDIS_PORT: 6379
    depends_on:
      postgres:
        condition: service_healthy
      redis:
        condition: service_healthy
    networks:
      - ecommerce-network
    restart: unless-stopped

  # Frontend
  frontend:
    build:
      context: ./frontend
      dockerfile: Dockerfile
      args:
        REACT_APP_API_URL: http://${HOST_IP}:5000/api
    container_name: ecommerce-frontend
    ports:
      - "3000:3000"
    depends_on:
      - backend
    networks:
      - ecommerce-network
    restart: unless-stopped

networks:
  ecommerce-network:
    driver: bridge

volumes:
  postgres_data:
    driver: local
EOF
```

**Explain each section:**

1. **Services:**
   - postgres: Database with volume and init script
   - redis: Cache with health check
   - backend: API with dependencies
   - frontend: UI with environment variable

2. **Networks:**
   - Custom bridge network for service communication
   - Isolation from other containers

3. **Volumes:**
   - postgres_data: Persistent storage
   - Named volume vs bind mount

4. **Health Checks:**
   - Why they're important
   - How depends_on works with health checks

5. **Environment Variables:**
   - Default values with ${VAR:-default}
   - Security considerations

### Create Environment File Template

Create `.env.example`:

> **What this file does:** Template file showing all required environment variables (database credentials, host IP) - users copy this to `.env` and customize for their environment.

```bash
cat > .env.example << 'EOF'
# Database Configuration
DB_NAME=ecommerce
DB_USER=postgres
DB_PASSWORD=postgres

# Host Configuration
# For EC2, set this to your EC2 public IP
# For localhost, use localhost or 127.0.0.1
HOST_IP=localhost

# Note: Never commit .env file to git!
# Copy this file to .env and update values
EOF
```

Create `.gitignore`:

> **What this file does:** Tells Git which files to ignore (environment secrets, build artifacts, IDE files, dependencies) - prevents sensitive data and temporary files from being committed to repository.

```bash
cat > .gitignore << 'EOF'
# Environment variables
.env

# Node modules
frontend/node_modules
frontend/build

# Python
backend/__pycache__
backend/*.pyc
backend/.pytest_cache

# IDE
.vscode
.idea
*.swp
*.swo

# OS
.DS_Store
Thumbs.db
EOF
```

---

## Part 9 - Running the Application (15 minutes)

### Step 1: Create Environment File

```bash
cp .env.example .env
```

If on EC2, update HOST_IP:
```bash
# Get EC2 public IP, and change HOST_IP in .env file.
```

### Step 2: Build and Start Services

```bash
# Build all images
docker-compose build

# Start all services
docker-compose up -d

# View logs
docker-compose logs -f
```

**Explain the process:**
- Build stage: Creating images
- Start stage: Creating and starting containers
- Log following: Real-time monitoring

### Step 3: Verify Services are Running

```bash
# Check all containers
docker-compose ps

# Check specific service logs
docker-compose logs backend
docker-compose logs frontend
docker-compose logs postgres
docker-compose logs redis
```

**Expected output:**
```
NAME                    STATUS              PORTS
ecommerce-backend       Up X minutes        0.0.0.0:5000->5000/tcp
ecommerce-cache         Up X minutes        6379/tcp
ecommerce-db            Up X minutes        5432/tcp
ecommerce-frontend      Up X minutes        0.0.0.0:3000->3000/tcp
```

### Step 4: Test the Application

**Test Backend API:**
```bash
# Health check
curl http://localhost:5000/api/health

# Get all products (first time - from database)
curl http://localhost:5000/api/products

# Get all products (second time - from cache)
curl http://localhost:5000/api/products
```

**Observe cache behavior:**
- First request: `"source": "database"`
- Second request: `"source": "cache"` ⚡

**Test Frontend:**
- Open browser: `http://localhost:3000` (or EC2 public IP)
- You should see the product catalog UI

### Step 5: Test CRUD Operations

**Create a product:**
```bash
curl -X POST http://localhost:5000/api/products \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Test Product",
    "description": "This is a test product",
    "price": 99.99,
    "stock": 10
  }'
```

**Update a product:**
```bash
curl -X PUT http://localhost:5000/api/products/1 \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Updated Laptop",
    "description": "Updated description",
    "price": 1399.99,
    "stock": 12
  }'
```

**Delete a product:**
```bash
curl -X DELETE http://localhost:5000/api/products/11
```

### Step 6: Verify Cache Invalidation

```bash
# Get products (should be from cache)
curl http://localhost:5000/api/products | grep source

# Create a new product
curl -X POST http://localhost:5000/api/products \
  -H "Content-Type: application/json" \
  -d '{"name": "New Product", "description": "Test", "price": 50, "stock": 5}'

# Get products again (should be from database, cache was invalidated)
curl http://localhost:5000/api/products | grep source
```

---

## Part 10 - Testing & Debugging (15 minutes)

### Docker Compose Commands

```bash
# View all containers
docker-compose ps

# View logs for all services
docker-compose logs

# View logs for specific service
docker-compose logs backend
docker-compose logs -f backend  # Follow mode

# Execute commands in containers
docker-compose exec backend python -c "print('Hello from backend')"
docker-compose exec postgres psql -U postgres -d ecommerce -c "SELECT COUNT(*) FROM products;"
docker-compose exec redis redis-cli KEYS "*"

# Restart a service
docker-compose restart backend

# Stop all services
docker-compose stop

# Start all services
docker-compose start

# Remove all containers (keeps images and volumes)
docker-compose down

# Remove all containers and volumes
docker-compose down -v

# Rebuild a specific service
docker-compose build backend
docker-compose up -d backend
```

### Debugging Common Issues

#### Issue 1: Backend Can't Connect to Database

**Symptoms:**
```
Error: Connection refused
```

**Debug steps:**
```bash
# Check if postgres is healthy
docker-compose ps postgres

# Check postgres logs
docker-compose logs postgres

# Verify database is ready
docker-compose exec postgres pg_isready -U postgres

# Check network connectivity
docker-compose exec backend ping postgres
```

#### Issue 2: Frontend Can't Reach Backend

**Symptoms:**
```
Failed to fetch
```

**Debug steps:**
```bash
# Check HOST_IP in .env file
cat .env

# Test backend from host
curl http://localhost:5000/api/health

# Check frontend environment
docker-compose exec frontend env | grep REACT_APP
```

#### Issue 3: Cache Not Working

**Debug steps:**
```bash
# Check Redis is running
docker-compose exec redis redis-cli ping

# Check cached keys
docker-compose exec redis redis-cli KEYS "*"

# Check TTL of cached data
docker-compose exec redis redis-cli TTL "products:all"

# Clear cache manually
docker-compose exec redis redis-cli FLUSHALL
```

#### Issue 4: Port Already in Use

**Symptoms:**
```
Error: Bind for 0.0.0.0:5000 failed: port is already allocated
```

**Solution:**
```bash
# Find process using the port
sudo lsof -i :5000

# Kill the process (replace PID)
kill -9 <PID>

# Or change port in docker-compose.yml
ports:
  - "5001:5000"  # Use different host port
```

### Database Operations

```bash
# Access PostgreSQL CLI
docker-compose exec postgres psql -U postgres -d ecommerce

# Run SQL queries
docker-compose exec postgres psql -U postgres -d ecommerce -c "SELECT * FROM products LIMIT 5;"

# Check table structure
docker-compose exec postgres psql -U postgres -d ecommerce -c "\d products"

# Count products
docker-compose exec postgres psql -U postgres -d ecommerce -c "SELECT COUNT(*) FROM products;"
```

### Performance Testing

```bash
# Install Apache Bench (if not available)
sudo yum install httpd-tools -y

# Test API performance
ab -n 1000 -c 10 http://localhost:5000/api/products

# Compare cache vs database performance
# First request (clears cache): measure time
# Subsequent requests (from cache): should be faster
```

---

## Part 11 - Production Best Practices (15 minutes)

### 1. Image Optimization

#### Multi-Stage Builds (Already Implemented in Frontend)

**Benefits:**
- Smaller image size
- No development dependencies in production
- Better security

**Example improvement for backend:**

```dockerfile
# Development stage
FROM python:3.11 AS development
WORKDIR /app
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY app/ ./app/

# Production stage
FROM python:3.11-slim AS production
WORKDIR /app
COPY --from=development /usr/local/lib/python3.11/site-packages /usr/local/lib/python3.11/site-packages
COPY --from=development /app /app
CMD ["python", "-m", "app.main"]
```

### 2. Security Best Practices

#### A. Use Non-Root User

Add to Dockerfile:
```dockerfile
RUN adduser -D appuser
USER appuser
```

#### B. Scan Images for Vulnerabilities

```bash
# Using Docker Scout (built-in)
docker scout cves ecommerce-backend

# Using Trivy
docker run aquasec/trivy image ecommerce-backend:latest
```

#### C. Use Secrets Management

**Don't do this:**
```yaml
environment:
  DB_PASSWORD: mypassword123  # Bad!
```

**Do this:**
```yaml
environment:
  DB_PASSWORD: ${DB_PASSWORD}  # Good! From .env file

# Or use Docker secrets (Swarm mode)
secrets:
  - db_password
```

### 3. Resource Limits

Add to docker-compose.yml:
```yaml
services:
  backend:
    # ... other config ...
    deploy:
      resources:
        limits:
          cpus: '0.5'
          memory: 512M
        reservations:
          cpus: '0.25'
          memory: 256M
```

### 4. Logging Configuration

```yaml
services:
  backend:
    logging:
      driver: "json-file"
      options:
        max-size: "10m"
        max-file: "3"
```

### 5. Health Checks in Dockerfile

Add to backend Dockerfile:
```dockerfile
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD python -c "import requests; requests.get('http://localhost:5000/api/health')" || exit 1
```

### 6. Environment-Specific Configurations

Create different docker-compose files:
```bash
# docker-compose.yml - Base configuration
# docker-compose.dev.yml - Development overrides
# docker-compose.prod.yml - Production settings
```

Usage:
```bash
# Development
docker-compose -f docker-compose.yml -f docker-compose.dev.yml up

# Production
docker-compose -f docker-compose.yml -f docker-compose.prod.yml up
```

### 7. Backup Strategy

```bash
# Backup database
docker-compose exec postgres pg_dump -U postgres ecommerce > backup.sql

# Restore database
cat backup.sql | docker-compose exec -T postgres psql -U postgres ecommerce

# Backup volume
docker run --rm -v ecommerce_postgres_data:/data -v $(pwd):/backup alpine tar czf /backup/postgres-backup.tar.gz /data
```

### 8. Monitoring and Observability

**Add monitoring stack (optional):**
```yaml
# Add to docker-compose.yml
  prometheus:
    image: prom/prometheus
    ports:
      - "9090:9090"
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml

  grafana:
    image: grafana/grafana
    ports:
      - "3001:3000"
    depends_on:
      - prometheus
```

### 9. CI/CD Integration

**GitHub Actions example (.github/workflows/docker.yml):**
```yaml
name: Docker Build and Push

on:
  push:
    branches: [ main ]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2

      - name: Build images
        run: docker-compose build

      - name: Run tests
        run: docker-compose up -d && sleep 10 && curl http://localhost:5000/api/health

      - name: Push to registry
        run: |
          echo "${{ secrets.DOCKER_PASSWORD }}" | docker login -u "${{ secrets.DOCKER_USERNAME }}" --password-stdin
          docker-compose push
```

### 10. Deployment Checklist

**Before deploying to production:**

- [ ] All secrets are in environment variables
- [ ] Images are scanned for vulnerabilities
- [ ] Resource limits are set
- [ ] Health checks are configured
- [ ] Logging is properly configured
- [ ] Backup strategy is in place
- [ ] Monitoring is set up
- [ ] SSL/TLS is configured (if applicable)
- [ ] Database migrations are tested
- [ ] Rollback plan is ready

---

## Part 12 - Q&A and Next Steps (15 minutes)

### Common Questions

**Q: How is this different from Kubernetes?**
A: Docker Compose is for single-host deployments. Kubernetes is for multi-host, production-scale orchestration. This is a stepping stone to Kubernetes.

**Q: Can I use this in production?**
A: For small applications on a single server, yes. For larger scale, consider Kubernetes or Docker Swarm.

**Q: How do I update my application without downtime?**
A:
```bash
# Build new image
docker-compose build backend

# Recreate only backend
docker-compose up -d --no-deps backend
```

**Q: How do I scale services?**
A:
```bash
# Scale backend to 3 instances
docker-compose up -d --scale backend=3
```
Note: You'll need a load balancer in front.

### Next Steps for Students

1. **Add features to this project:**
   - User authentication
   - Image upload for products
   - Shopping cart functionality
   - Order management

2. **Explore advanced topics:**
   - Docker Swarm mode
   - Kubernetes basics
   - Service mesh (Istio, Linkerd)
   - Container security

3. **Practice scenarios:**
   - Deploy to AWS ECS
   - Use AWS ECR for images
   - Set up CI/CD pipeline
   - Implement blue-green deployment

4. **Portfolio project:**
   - Push this to GitHub
   - Add comprehensive README
   - Include architecture diagrams
   - Document your learning

### Useful Resources

- **Docker Documentation**: https://docs.docker.com
- **Docker Compose Reference**: https://docs.docker.com/compose/compose-file/
- **Best Practices**: https://docs.docker.com/develop/dev-best-practices/
- **Security**: https://docs.docker.com/engine/security/
- **Kubernetes (next step)**: https://kubernetes.io/docs/tutorials/

### Cleanup

When you're done:
```bash
# Stop and remove containers, networks
docker-compose down

# Remove containers, networks, and volumes
docker-compose down -v

# Remove images
docker-compose down --rmi all

# Complete cleanup
docker system prune -a --volumes
```

---

## Summary

### What We Learned Today

1. **Multi-Container Architecture**
   - Designing microservices
   - Service communication
   - Network isolation

2. **Docker Compose**
   - Service definition
   - Dependencies management
   - Volume and network configuration

3. **Real-World Patterns**
   - API development
   - Database integration
   - Caching strategies
   - Frontend-backend separation

4. **Operational Skills**
   - Building and running containers
   - Debugging issues
   - Log management
   - Health monitoring

5. **Best Practices**
   - Image optimization
   - Security considerations
   - Production readiness
   - CI/CD integration

### Skills Gained

- Full-stack application containerization
- Docker Compose orchestration
- Multi-container debugging
- Production deployment preparation
- Cloud deployment readiness (AWS/Azure/GCP)

---

## Troubleshooting Guide

### Quick Reference

| Issue | Command | Solution |
|-------|---------|----------|
| Container won't start | `docker-compose logs <service>` | Check logs for errors |
| Port already in use | `sudo lsof -i :<port>` | Kill process or change port |
| Database connection error | `docker-compose exec postgres pg_isready` | Ensure DB is healthy |
| Cache not working | `docker-compose exec redis redis-cli ping` | Verify Redis is running |
| Changes not reflected | `docker-compose build <service> && docker-compose up -d <service>` | Rebuild and recreate |
| Out of disk space | `docker system df` | Check space, prune if needed |
| Network issues | `docker-compose exec <service> ping <other-service>` | Test connectivity |

---

## Congratulations!

You've successfully built a production-ready multi-container application using Docker! This project demonstrates:

- Professional software architecture
- Real-world development practices
- DevOps engineering skills
- Cloud-native application design

**You're now ready to:**
- Add this project to your portfolio
- Interview confidently about Docker
- Proceed to Kubernetes learning
- Deploy applications to cloud platforms

**Remember:** The best way to learn is by doing. Modify this project, break it, fix it, and make it your own!

---

**Happy Dockerizing! 🐳**