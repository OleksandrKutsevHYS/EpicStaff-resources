# 📘 Organization Variables Documentation

## **Overview**

Organization Variables allow you to manage persistent state across flow executions. You can configure which variables persist at the organization level (shared across all users) and which persist per user (individual state). Variables are automatically managed based on your configuration.

---

## **API Endpoints**

### Organization Endpoints
```bash
GET    /api/organizations/
POST   /api/organizations/
GET    /api/organizations/{id}/
PUT    /api/organizations/{id}/
PATCH  /api/organizations/{id}/
DELETE /api/organizations/{id}/
```

### Organization User Endpoints
```bash
GET    /api/organization-users/
POST   /api/organization-users/
GET    /api/organization-users/{id}/
PUT    /api/organization-users/{id}/
PATCH  /api/organization-users/{id}/
DELETE /api/organization-users/{id}/
```

### Graph Organization Objects Endpoints
```bash
GET    /api/graph-organizations/
POST   /api/graph-organizations/
GET    /api/graph-organizations/{id}/
PUT    /api/graph-organizations/{id}/
PATCH  /api/graph-organizations/{id}/
DELETE /api/graph-organizations/{id}/
```

### Graph Organization User Endpoints
```bash
GET    /api/organization-users/
GET    /api/organization-users/{id}/
```

---

## **Setup Process**

### Step 1: Create an Organization

**Request**
```bash
POST /api/organizations/
```

**Body Parameters**
| Field  | Type   | Required | Description |
|--------|--------|----------|-------------|
| `name` | string | ✅       | Unique organization identifier |

**Example**
```bash
curl -X POST http://localhost:8000/api/organizations/ \
  -H "Content-Type: application/json" \
  -d '{"name": "acme_corp"}'
```

---

### Step 2: Create Organization Users

**Request**
```bash
POST /api/organization-users/
```

**Body Parameters**
| Field  | Type   | Required | Description |
|--------|--------|----------|-------------|
| `name` | string | ✅       | Unique username within organization |

**Example**
```bash
curl -X POST http://localhost:8000/api/organization-users/ \
  -H "Content-Type: application/json" \
  -d '{"name": "john_doe"}'
```

---

### Step 3: Configure Persistent Variables for a Flow

**Request**
```bash
POST /api/graph-organizations/
```

**Body Parameters**
| Field                  | Type    | Required | Description |
|------------------------|---------|----------|-------------|
| `graph`                | integer | ✅       | Flow ID |
| `organization`         | integer | ✅       | Organization ID |
| `persistent_variables` | object  | ❌       | Variables that persist at organization level |
| `user_variables`       | object  | ❌       | Variables that persist per user |

**Validation Rules**
- All specified variables must exist in the flow's domain
- A variable cannot be in both `persistent_variables` and `user_variables`
- If a variable is removed from the flow domain, it will be automatically removed from persistent storage

**Example**
```bash
curl -X POST http://localhost:8000/api/graph-organizations/ \
  -H "Content-Type: application/json" \
  -d '{
    "graph": 123,
    "organization": 1,
    "persistent_variables": {
      "api_calls": 0,
      "total_credits": 100
    },
    "user_variables": {
      "user_credits": 50,
      "last_login": null
    }
  }'
```

---

## **Running Flows with Persistent Variables**

### Run Session Endpoint
```bash
POST /api/run-session/
```

### Headers
| Name            | Value                 | Required | Notes                   |
|-----------------|-----------------------|----------|-------------------------|
| `Content-Type`  | `multipart/form-data` | ✅       | Required for file uploads |

### Body Parameters
| Field       | Type        | Required | Description |
|-------------|-------------|----------|-------------|
| `graph_id`  | integer     | ✅       | ID of the flow to run |
| `username`  | string      | ❌       | Username to load user-specific persistent variables |

---

## **Example Requests**

### Using `curl`

```bash
curl -X POST http://localhost:8000/api/run-session/ \
  -F "graph_id=123" \
  -F "username=john_doe"
```

### Using Python

**With User Context**
```python
import requests

url = "http://localhost:8000/api/run-session/"
data = {
    "graph_id": 123,
    "username": "john_doe"
}

response = requests.post(url, data=data)
print(response.json())
```

---

## **How Persistent Variables Work**

### Variable Lifecycle

1. **Initialization**: When a flow starts, persistent variables are loaded from the database
   - Organization variables: loaded from the organization's stored state
   - User variables: loaded from the specific user's stored state (if `username` provided)

2. **Execution**: During flow execution, you can read and modify these variables
   - Modify variables in your flow nodes
   - Use `output_variable_path` to update the variable values

3. **Persistence**: After successful flow completion, modified variables are automatically saved back
   - Organization variables: saved to organization storage
   - User variables: saved to user storage

---

## **Variable Cleanup**

When you remove a variable from your flow's domain, the system automatically:
- Removes it from organization `persistent_variables`
- Removes it from all users' `user_variables`

This ensures your persistent storage stays synchronized with your flow definition.

---

## **Important Notes**

📌 **Content Type**
- `/api/run-session/` endpoint uses `multipart/form-data`
- All other endpoints use `application/json`

📌 **Variable Configuration**
- Variables must exist in the flow domain before being marked as persistent
- A variable can only be persistent at organization OR user level, not both
- Configuration is per flow and per organization

📌 **Variable Persistence**
- Variables are saved after successful flow completion
- Failed flows do not persist variable changes
- Only variables with `output_variable_path` set will be updated

📌 **User Context**
- Pass `username` in run-session to load user-specific variables
- Without `username`, only organization variables are available
- Each user maintains independent state for their variables

---
