---
name: openapi-docs
description: Document REST API endpoints using OpenAPI 3.0 specification. Use this skill when adding new endpoints, modifying API contracts, or updating request/response schemas.
compatibility: OpenAPI 3.0+, YAML format
---

# OpenAPI Documentation Skill

## When to use this skill

Use this skill when:
- Adding new API endpoints
- Modifying existing endpoint contracts
- Updating request/response schemas
- Documenting error responses

## Architecture Overview

```
docs/
└── openapi.yaml    # Complete API specification
```

All API endpoints must be documented in `docs/openapi.yaml` before or during implementation.

## OpenAPI File Structure

```yaml
openapi: 3.0.3
info:
  title: Todo API
  description: Simple todo application API
  version: 1.0.0

servers:
  - url: http://localhost:3000
    description: Development server

paths:
  /api/todos:
    get:
      # ... endpoint definition
    post:
      # ... endpoint definition

components:
  schemas:
    Todo:
      # ... schema definition
```

## Documenting Endpoints

### GET Endpoint (List)

```yaml
paths:
  /api/todos:
    get:
      summary: List all todos
      description: Retrieve all todos from the database
      operationId: getTodos
      tags:
        - Todos
      responses:
        '200':
          description: List of todos
          content:
            application/json:
              schema:
                type: array
                items:
                  $ref: '#/components/schemas/Todo'
              example:
                - id: "1697234567890"
                  text: "Buy groceries"
                  completed: false
                  createdAt: "2024-01-15T10:30:00.000Z"
```

### POST Endpoint (Create)

```yaml
paths:
  /api/todos:
    post:
      summary: Create a new todo
      description: Add a new todo item to the list
      operationId: createTodo
      tags:
        - Todos
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/CreateTodoRequest'
            example:
              text: "Buy groceries"
      responses:
        '201':
          description: Todo created successfully
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Todo'
        '400':
          description: Invalid request
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Error'
```

### PATCH Endpoint (Update)

```yaml
paths:
  /api/todos/{id}:
    patch:
      summary: Update a todo
      description: Update an existing todo's properties
      operationId: updateTodo
      tags:
        - Todos
      parameters:
        - name: id
          in: path
          required: true
          description: Todo ID
          schema:
            type: string
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/UpdateTodoRequest'
      responses:
        '200':
          description: Todo updated successfully
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Todo'
        '404':
          description: Todo not found
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Error'
```

### DELETE Endpoint

```yaml
paths:
  /api/todos/{id}:
    delete:
      summary: Delete a todo
      description: Remove a todo from the list
      operationId: deleteTodo
      tags:
        - Todos
      parameters:
        - name: id
          in: path
          required: true
          description: Todo ID
          schema:
            type: string
      responses:
        '204':
          description: Todo deleted successfully
        '404':
          description: Todo not found
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Error'
```

## Defining Schemas

### Resource Schema

```yaml
components:
  schemas:
    Todo:
      type: object
      required:
        - id
        - text
        - completed
        - createdAt
      properties:
        id:
          type: string
          description: Unique identifier
          example: "1697234567890"
        text:
          type: string
          description: Todo item text
          minLength: 1
          maxLength: 500
          example: "Buy groceries"
        completed:
          type: boolean
          description: Completion status
          example: false
        createdAt:
          type: string
          format: date-time
          description: Creation timestamp
          example: "2024-01-15T10:30:00.000Z"
```

### Request Schemas

```yaml
components:
  schemas:
    CreateTodoRequest:
      type: object
      required:
        - text
      properties:
        text:
          type: string
          minLength: 1
          maxLength: 500
          example: "Buy groceries"

    UpdateTodoRequest:
      type: object
      properties:
        text:
          type: string
          minLength: 1
          maxLength: 500
        completed:
          type: boolean
```

### Error Schema

```yaml
components:
  schemas:
    Error:
      type: object
      required:
        - error
      properties:
        error:
          type: string
          description: Error message
          example: "Todo not found"
```

## Conventions

### Naming
- Use `operationId` for each endpoint (camelCase): `getTodos`, `createTodo`
- Use singular resource names in schemas: `Todo`, not `Todos`
- Use descriptive request schema names: `CreateTodoRequest`, `UpdateTodoRequest`

### Tags
- Group endpoints by resource: `tags: [Todos]`
- One tag per resource type

### Examples
- Include realistic examples for all schemas
- Show both request and response examples

### HTTP Methods
- `GET` - Retrieve resources
- `POST` - Create new resource
- `PATCH` - Partial update (preferred over PUT)
- `DELETE` - Remove resource

### Response Codes
- `200` - Success (GET, PATCH)
- `201` - Created (POST)
- `204` - No Content (DELETE)
- `400` - Bad Request
- `404` - Not Found
- `500` - Server Error

## Adding a New Endpoint

1. Add path under `paths:` section
2. Define all possible responses
3. Reference or create schemas in `components/schemas`
4. Include examples for documentation
5. Implement the endpoint in `src/routes/`

## Validation

Use tools to validate your OpenAPI spec:
- Online: [Swagger Editor](https://editor.swagger.io/)
- CLI: `npx @redocly/cli lint docs/openapi.yaml`
