# 📘 Sub-Flow Node

## **Description**
This node allows you to execute another flow within your current flow, enabling modular and reusable workflow design.

- **subgraph_id** is required and specifies which flow to execute as a sub-flow.
  - You can reference any existing flow **except** the current flow (recursion is forbidden and validated).
- **input_map** is optional and allows you to pass data from the parent flow to the sub-flow.
  - Any variables passed will **override** existing variables in the sub-flow with the same name.
- **output_variable_path** is optional, but required if you want to access the sub-flow's results in subsequent nodes.
  - The entire state of the sub-flow after execution will be stored at this path.

---

## Important Notes

- **Recursion is forbidden**: You cannot reference the same flow you're currently in. The system validates this and will prevent circular references.
- **Variable Override**: If your sub-flow has a variable named `A` and you pass a variable `A` from the parent flow via `input_map`, the sub-flow's original `A` will be overridden with the parent's value.
- **State Isolation**: Each sub-flow executes in its own context. Only variables explicitly passed through `input_map` are shared from parent to sub-flow.

---

## Use Cases

- **Modular Workflows**: Break complex flows into smaller, manageable sub-flows.
- **Reusable Logic**: Create common operations (e.g., sending emails, validating data) that can be reused across multiple flows.
- **Conditional Branching**: Execute different sub-flows based on conditions in your parent flow.
- **Parallel Processing**: Execute multiple sub-flows to handle different aspects of your workflow simultaneously.

---

## API Endpoints

The Sub-Flow Node can be managed via the following REST API endpoints:

### **Base URL**
```
/subgraph-nodes/
```

### **Available Endpoints**

- GET /subgraph-nodes/
- POST /subgraph-nodes/
- GET /subgraph-nodes/{id}/
- PUT /subgraph-nodes/{id}/
- PATCH /subgraph-nodes/{id}/
- DELETE /subgraph-nodes/{id}/

---

## Validation Rules

- **Recursion Prevention**: The `subgraph` ID cannot be the same as the `graph` ID. The system will return a validation error if you attempt to create a recursive sub-flow.
- **Graph/Subgraph Existence**: Both `graph` and `subgraph` must reference existing flows in the system.
- **Input Map Format**: `input_map` must be a valid JSON object with string keys.'
