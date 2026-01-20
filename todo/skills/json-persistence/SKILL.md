---
name: json-persistence
description: Implement data persistence using JSON file storage. Use this skill when adding data storage capabilities, implementing CRUD operations at the data layer, or modifying how data is read and written to disk.
compatibility: Node.js 18+, fs/promises module
---

# JSON Persistence Skill

## When to use this skill

Use this skill when:
- Adding new data storage functionality
- Implementing CRUD operations at the persistence layer
- Modifying data file structure or location
- Adding data validation at the storage level

## Architecture Overview

```
src/persistence/
└── fileStorage.js     # Generic JSON file storage class

data/
└── todos.json         # Data file (auto-created)
```

The persistence layer is the lowest layer in the architecture:

```
Routes → Services → Persistence (this layer)
                         ↓
                    data/*.json
```

## Core Storage Implementation

### FileStorage Class

```javascript
// src/persistence/fileStorage.js

const fs = require('fs').promises;
const path = require('path');

class FileStorage {
  constructor(filename) {
    this.filepath = path.join(__dirname, '../../data', filename);
  }

  async load() {
    try {
      const data = await fs.readFile(this.filepath, 'utf8');
      return JSON.parse(data);
    } catch (error) {
      if (error.code === 'ENOENT') {
        return []; // File doesn't exist yet
      }
      throw error;
    }
  }

  async save(data) {
    const dir = path.dirname(this.filepath);
    await fs.mkdir(dir, { recursive: true });
    await fs.writeFile(this.filepath, JSON.stringify(data, null, 2), 'utf8');
  }

  async findById(id) {
    const data = await this.load();
    return data.find(item => item.id === id) || null;
  }

  async create(item) {
    const data = await this.load();
    data.push(item);
    await this.save(data);
    return item;
  }

  async update(id, updates) {
    const data = await this.load();
    const index = data.findIndex(item => item.id === id);
    if (index === -1) {
      return null;
    }
    data[index] = { ...data[index], ...updates };
    await this.save(data);
    return data[index];
  }

  async delete(id) {
    const data = await this.load();
    const index = data.findIndex(item => item.id === id);
    if (index === -1) {
      return false;
    }
    data.splice(index, 1);
    await this.save(data);
    return true;
  }
}

module.exports = FileStorage;
```

### Usage in Services

```javascript
// src/services/todoService.js

const FileStorage = require('../persistence/fileStorage');
const storage = new FileStorage('todos.json');

const todoService = {
  async getAll() {
    return await storage.load();
  },

  async getById(id) {
    return await storage.findById(id);
  },

  async create(text) {
    const todo = {
      id: Date.now().toString(),
      text: text.trim(),
      completed: false,
      createdAt: new Date().toISOString()
    };
    return await storage.create(todo);
  },

  async update(id, updates) {
    return await storage.update(id, updates);
  },

  async delete(id) {
    return await storage.delete(id);
  }
};

module.exports = todoService;
```

## Data File Format

### todos.json Structure

```json
[
  {
    "id": "1697234567890",
    "text": "Buy groceries",
    "completed": false,
    "createdAt": "2024-01-15T10:30:00.000Z"
  },
  {
    "id": "1697234567891",
    "text": "Walk the dog",
    "completed": true,
    "createdAt": "2024-01-15T11:00:00.000Z"
  }
]
```

### Field Conventions

| Field | Type | Description |
|-------|------|-------------|
| `id` | string | Unique identifier (timestamp-based) |
| `createdAt` | string | ISO 8601 timestamp |
| `updatedAt` | string | ISO 8601 timestamp (optional) |

## Conventions

### File Location
- All data files in `data/` directory
- One JSON file per resource type (e.g., `todos.json`)
- Directory auto-created if missing

### ID Generation
- Use timestamp-based IDs: `Date.now().toString()`
- Simple and unique enough for single-user apps

### Error Handling
- Return empty array `[]` if file doesn't exist
- Throw errors for actual read/write failures
- Let service layer handle business logic errors

### Data Integrity
- Always use `JSON.stringify(data, null, 2)` for readable files
- Load entire file, modify in memory, save entire file
- No partial updates (simple but safe)

## Adding a New Resource

1. Create storage instance in service:
```javascript
const storage = new FileStorage('newresource.json');
```

2. Define data structure with required fields:
```javascript
const item = {
  id: Date.now().toString(),
  // ... resource-specific fields
  createdAt: new Date().toISOString()
};
```

3. Data file auto-created on first save

## Testing

```javascript
// tests/persistence/fileStorage.test.js

const FileStorage = require('../../src/persistence/fileStorage');
const fs = require('fs').promises;

describe('FileStorage', () => {
  const testStorage = new FileStorage('test-todos.json');

  afterEach(async () => {
    try {
      await fs.unlink('./data/test-todos.json');
    } catch {}
  });

  test('load() returns empty array for missing file', async () => {
    const data = await testStorage.load();
    expect(data).toEqual([]);
  });

  test('save() and load() round-trip data', async () => {
    const todos = [{ id: '1', text: 'Test' }];
    await testStorage.save(todos);
    const loaded = await testStorage.load();
    expect(loaded).toEqual(todos);
  });
});
```
