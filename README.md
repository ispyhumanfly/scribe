[![CircleCI](https://circleci.com/gh/ispyhumanfly/scribe/tree/master.svg?style=svg)](https://circleci.com/gh/ispyhumanfly/scribe/tree/master)
[![Known Vulnerabilities](https://snyk.io/test/github/ispyhumanfly/scribe/badge.svg)](https://snyk.io/test/github/ispyhumanfly/scribe)
[![npm (scoped)](https://img.shields.io/npm/v/@spytech/scribe.svg)](https://www.npmjs.com/package/@spytech/scribe)
[![NpmLicense](https://img.shields.io/npm/l/@spytech/scribe.svg)](https://github.com/ispyhumanfly/scribe/blob/master/LICENSE)

# Scribe

Scribe is a RESTful API server that brings component-based architecture to data modeling. Just as modern frontend frameworks break down UIs into reusable components, Scribe organizes your data into logical, self-contained components and subcomponents that can be composed and related to each other.

## How Components Work

In Scribe, a component represents a distinct data model with its own schema, validation rules, and history tracking. Components can be:

-   **Base Components**: Like `users` or `products`
-   **Subcomponents**: Extensions of base components like `users/profile` or `products/inventory`
-   **Related**: Through parent-child relationships or references

For example, an e-commerce system might be modeled as:

```typescript
// Base user component
POST /users
{
  "name": "John Doe",
  "email": "john@example.com"
}

// User profile as a subcomponent
POST /users/profile
{
  "avatar": "https://...",
  "bio": "Software engineer",
  "location": "San Francisco"
}

// Product component with inventory subcomponent
POST /products
{
  "name": "Gaming Laptop",
  "price": 1299.99
}

POST /products/inventory
{
  "sku": "GL-2023",
  "stockLevel": 50,
  "warehouse": "SF-1"
}
```

Each component and subcomponent automatically gets:

-   Schema validation
-   Version history tracking
-   Relationship querying
-   Time machine capabilities

This component-based approach makes it natural to:

-   Organize complex data models
-   Maintain data integrity
-   Track changes over time
-   Scale your data architecture

## Features

-   **Schema Validation**: Automatic validation of data against JSON schemas
-   **History Tracking**: Built-in version history for all records
-   **Redis Caching**: Schema caching for improved performance
-   **PostgreSQL Storage**: Reliable and scalable data storage
-   **Complex Queries**: Support for filtering, grouping, and relationships
-   **Time Machine**: Ability to view data as it existed at any point in time
-   **Multi-language Support**: Easy to use from any programming language
-   **Flexible SQL Queries**: Support for complex SQL operations including joins, aggregations, and subqueries
-   **Query Parameter Support**: Easy filtering, sorting, and pagination through URL parameters
-   **Transaction Support**: Atomic operations for data integrity
-   **Dynamic Query Building**: API for constructing complex queries programmatically
-   **Raw SQL Access**: Direct SQL execution for advanced use cases

## Installation

### Prerequisites

-   Node.js >= 12
-   PostgreSQL >= 9.6.10
-   Redis (optional, for schema caching)

```bash
npm install scribe
```

## Configuration

Scribe can be configured through environment variables or command line arguments:

```bash
# Required
SCRIBE_APP_DB_HOST=localhost
SCRIBE_APP_DB_USER=your_user
SCRIBE_APP_DB_PASS=your_password
SCRIBE_APP_DB_NAME=your_database

# Optional
SCRIBE_APP_PORT=1337
SCRIBE_APP_MODE=development
SCRIBE_APP_SCHEMA_BASE_URL=http://your-schema-server
```

## Usage Examples

### TypeScript with Axios

```typescript
import axios from "axios"

const scribeClient = axios.create({
    baseURL: "http://localhost:1337"
})

// Create a new record
const createUser = async () => {
    const response = await scribeClient.post("/users", {
        name: "John Doe",
        email: "john@example.com",
        age: 30
    })
    return response.data
}

// Get a record by ID
const getUser = async (id: string) => {
    const response = await scribeClient.get(`/users/${id}`)
    return response.data
}

// Get all records with filtering
const getUsers = async () => {
    const response = await scribeClient.get("/users/all", {
        params: {
            filter: JSON.stringify({
                age: [25, 30, 35]
            })
        }
    })
    return response.data
}

// Update a record
const updateUser = async (id: string, data: any) => {
    const response = await scribeClient.put(`/users/${id}`, data)
    return response.data
}

// Get history of a record
const getUserHistory = async (id: string) => {
    const response = await scribeClient.get(`/users/${id}/history`)
    return response.data
}
```

### Go with net/http

```go
package main

import (
    "bytes"
    "encoding/json"
    "fmt"
    "io"
    "net/http"
)

type User struct {
    Name  string `json:"name"`
    Email string `json:"email"`
    Age   int    `json:"age"`
}

func createUser(user User) (*User, error) {
    data, err := json.Marshal(user)
    if err != nil {
        return nil, err
    }

    resp, err := http.Post("http://localhost:1337/users", "application/json", bytes.NewBuffer(data))
    if err != nil {
        return nil, err
    }
    defer resp.Body.Close()

    var result User
    if err := json.NewDecoder(resp.Body).Decode(&result); err != nil {
        return nil, err
    }

    return &result, nil
}

func getUser(id string) (*User, error) {
    resp, err := http.Get(fmt.Sprintf("http://localhost:1337/users/%s", id))
    if err != nil {
        return nil, err
    }
    defer resp.Body.Close()

    var result []User
    if err := json.NewDecoder(resp.Body).Decode(&result); err != nil {
        return nil, err
    }

    if len(result) == 0 {
        return nil, fmt.Errorf("user not found")
    }

    return &result[0], nil
}

func getUsers() ([]User, error) {
    filter := map[string][]int{"age": {25, 30, 35}}
    filterJSON, err := json.Marshal(filter)
    if err != nil {
        return nil, err
    }

    resp, err := http.Get(fmt.Sprintf("http://localhost:1337/users/all?filter=%s", string(filterJSON)))
    if err != nil {
        return nil, err
    }
    defer resp.Body.Close()

    var result []User
    if err := json.NewDecoder(resp.Body).Decode(&result); err != nil {
        return nil, err
    }

    return result, nil
}
```

### Python with requests

```python
import requests
import json

class ScribeClient:
    def __init__(self, base_url="http://localhost:1337"):
        self.base_url = base_url

    def create_user(self, user_data):
        response = requests.post(
            f"{self.base_url}/users",
            json=user_data
        )
        return response.json()

    def get_user(self, user_id):
        response = requests.get(f"{self.base_url}/users/{user_id}")
        return response.json()

    def get_users(self, filter_data=None):
        params = {}
        if filter_data:
            params['filter'] = json.dumps(filter_data)

        response = requests.get(
            f"{self.base_url}/users/all",
            params=params
        )
        return response.json()

    def update_user(self, user_id, user_data):
        response = requests.put(
            f"{self.base_url}/users/{user_id}",
            json=user_data
        )
        return response.json()

    def get_user_history(self, user_id):
        response = requests.get(f"{self.base_url}/users/{user_id}/history")
        return response.json()

# Usage example
client = ScribeClient()

# Create a user
user = client.create_user({
    "name": "John Doe",
    "email": "john@example.com",
    "age": 30
})

# Get user by ID
user_data = client.get_user(user[0]["id"])

# Get users with filter
users = client.get_users({"age": [25, 30, 35]})

# Update user
updated_user = client.update_user(user[0]["id"], {
    "name": "John Doe",
    "email": "john@example.com",
    "age": 31
})

# Get user history
history = client.get_user_history(user[0]["id"])
```

## Advanced Features

### Time Machine

Scribe supports viewing data as it existed at any point in time:

```typescript
// Get data as it existed at a specific timestamp
const getHistoricalData = async (id: string, timestamp: string) => {
    const response = await scribeClient.get(`/users/${id}`, {
        params: {
            timeMachine: JSON.stringify({
                key: "updatedAt",
                timestamp: timestamp
            })
        }
    })
    return response.data
}
```

### Relationships

Scribe supports querying related data:

```typescript
// Get parent records
const getParentRecords = async (id: string) => {
    const response = await scribeClient.get(`/users/${id}`, {
        params: {
            parents: "parentId"
        }
    })
    return response.data
}

// Get child records
const getChildRecords = async (id: string) => {
    const response = await scribeClient.get(`/users/${id}`, {
        params: {
            children: "parentId"
        }
    })
    return response.data
}
```

### SQL Queries

Scribe provides a direct SQL endpoint for advanced querying capabilities:

```typescript
// Execute a custom SQL query
const executeSqlQuery = async (query: string) => {
    const response = await scribeClient.post("/sql", {
        query: query
    })
    return response.data
}

// Example: Complex join query
const getUsersWithOrders = async () => {
    const query = `
        SELECT u.*, o.*
        FROM users u
        LEFT JOIN orders o ON u.id = o.user_id
        WHERE o.status = 'completed'
        ORDER BY u.created_at DESC
    `
    return executeSqlQuery(query)
}

// Example: Aggregation query
const getUserStats = async () => {
    const query = `
        SELECT 
            COUNT(*) as total_users,
            AVG(age) as average_age,
            MAX(created_at) as newest_user
        FROM users
    `
    return executeSqlQuery(query)
}

// Example: Subquery with filtering
const getActiveUsersWithRecentOrders = async () => {
    const query = `
        SELECT *
        FROM users
        WHERE id IN (
            SELECT DISTINCT user_id
            FROM orders
            WHERE created_at > NOW() - INTERVAL '30 days'
        )
        AND status = 'active'
    `
    return executeSqlQuery(query)
}
```

> **Note**: The SQL endpoint should only be used in trusted environments as it provides direct database access. Make sure to properly validate and sanitize any user input before using it in queries.

## API Endpoints

-   `POST /:component` - Create a new record
-   `GET /:component/:id` - Get a record by ID
-   `GET /:component/all` - Get all records
-   `PUT /:component/:id` - Update a record
-   `DELETE /:component/:id` - Delete a record
-   `GET /:component/:id/history` - Get record history
-   `DELETE /:component/all` - Delete all records
-   `DELETE /:component` - Drop the component table

## License

MIT
