---
name: express-service
description: Create and modify Express.js routes and service layer business logic. Use this skill when adding API endpoints, implementing request handlers, or writing business logic that orchestrates data operations.
compatibility: Node.js 18+, Express.js
---

# Express Service Skill

## When to use this skill

Use this skill when:
- Adding new API endpoints to the application
- Modifying existing route handlers
- Implementing business logic in the service layer
- Adding request validation or response formatting

## Architecture Overview

This project follows a 3-layer architecture:

```
Routes (src/routes/)      → HTTP handling, validation
    ↓
Services (src/services/)  → Business logic, orchestration
    ↓
Persistence (src/persistence/) → Data storage
```

## Creating a New Route

### Step 1: Define the route in `src/routes/`

```javascript
const express = require('express');
const router = express.Router();
const todoService = require('../services/todoService');

// GET /api/todos - List all todos
router.get('/', async (req, res) => {
  try {
    const todos = await todoService.getAll();
    res.json(todos);
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

module.exports = router;
```

### Step 2: Implement business logic in `src/services/`

```javascript
const storage = require('../persistence/fileStorage');

const todoService = {
  async getAll() {
    return await storage.load();
  },

  async create(text) {
    if (!text || text.trim() === '') {
      throw new Error('Todo text is required');
    }
    const todos = await storage.load();
    const newTodo = {
      id: Date.now().toString(),
      text: text.trim(),
      completed: false,
      createdAt: new Date().toISOString()
    };
    todos.push(newTodo);
    await storage.save(todos);
    return newTodo;
  }
};

module.exports = todoService;
```

### Step 3: Register route in `server.js`

```javascript
const todoRoutes = require('./src/routes/todoRoutes');
app.use('/api/todos', todoRoutes);
```

## Conventions

### Route Files
- One route file per resource (e.g., `todoRoutes.js`)
- Use Express Router for modular routes
- Handle errors with try/catch and appropriate status codes
- Keep route handlers thin - delegate to services

### Service Files
- One service file per domain (e.g., `todoService.js`)
- Services contain business logic and validation
- Services call persistence layer for data operations
- Return data objects, not HTTP responses

### HTTP Status Codes
- `200` - Success (GET, PATCH)
- `201` - Created (POST)
- `204` - No Content (DELETE)
- `400` - Bad Request (validation errors)
- `404` - Not Found
- `500` - Server Error

### Error Handling Pattern

```javascript
router.post('/', async (req, res) => {
  try {
    const { text } = req.body;
    if (!text) {
      return res.status(400).json({ error: 'Text is required' });
    }
    const todo = await todoService.create(text);
    res.status(201).json(todo);
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});
```

## File Naming

- Routes: `{resource}Routes.js` (e.g., `todoRoutes.js`)
- Services: `{resource}Service.js` (e.g., `todoService.js`)
- Use camelCase for all JavaScript files

## Testing

After creating routes and services, add tests in `tests/services/`:

```javascript
const todoService = require('../../src/services/todoService');

describe('todoService', () => {
  test('create() should add a new todo', async () => {
    const todo = await todoService.create('Test todo');
    expect(todo.text).toBe('Test todo');
    expect(todo.completed).toBe(false);
  });
});
```
