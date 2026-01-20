# ARCHITECTURE.md

This file provides comprehensive guidance for web development projects, emphasizing simplicity, maintainability, and web fundamentals.

## Core Development Philosophy

This will be an express application. State of objects will be saved on the server side as json.

### KISS (Keep It Simple, Stupid)

Simplicity should be a key goal in design. Choose straightforward solutions over complex ones whenever possible. Simple solutions are easier to understand, maintain, and debug.

## 🧱 Code Structure & Modularity

### File and Function Limits

- **Multi-File Projects**: Never create a file longer than 500 lines
- **Functions should be under 50 lines** with a single, clear responsibility
- **Line length should be max 100 characters** for readability
- **Prefer vanilla web technologies** over frameworks when simplicity is the goal

### Project Architecture

#### Simple Server Architecture
```
/
├── server.js               # Express/Node.js server
├── public/
│   ├── index.html
│   ├── styles.css
│   └── script.js
├── data/
│   └── storage.json        # File-based data storage
├── docs/
│   ├── openapi.yaml        # OpenAPI specification
│   └── api/
│       └── endpoints/      # Individual endpoint documentation
├── package.json
└── README.md
```

#### Technology Standards

##### Core Technology Stack
- **Backend**: Node.js with Express.js framework
- **Frontend**: Vanilla HTML5, CSS3, JavaScript (ES6+)
- **Data Storage**: JSON files for simplicity, with option to migrate to database later
- **Documentation**: OpenAPI 3.0+ for API specifications

##### Approved Technologies
- **HTTP Client**: Native `fetch()` API or Axios for complex scenarios
- **CSS Framework**: None (vanilla CSS preferred) or Tailwind CSS for utility-first approach
- **Testing**: Jest for unit/integration testing
- **Linting**: ESLint with standard configuration
- **Formatting**: Prettier with 2-space indentation


##### Version Requirements
- **Node.js**: LTS version (currently 18.x or 20.x)
- **npm**: Latest stable version bundled with Node.js

## 🛠️ Development Environment

### Simple Server Setup

#### Required Tools
- **Runtime**: Node.js 
- **Package Manager**: npm (comes with Node.js) 

## 📋 Style & Conventions

### HTML/CSS/JavaScript Style Guide

#### HTML Standards
- **Use semantic HTML5 elements** (`<main>`, `<section>`, `<article>`, etc.)
- **Proper nesting and indentation** (2 spaces)
- **Accessible attributes** (`alt`, `aria-label`, proper form labels)
- **Clean, minimal markup** - avoid unnecessary divs

#### CSS Standards  
- **Mobile-first responsive design** using CSS Grid and Flexbox
- **CSS custom properties** for theming and consistency
- **BEM methodology** for class naming (when using separate CSS files)
- **Avoid !important** - use proper specificity
- **Line length: 100 characters max**

#### JavaScript Standards
- **ES6+ modern JavaScript** - avoid legacy patterns
- **const/let instead of var** - proper scoping
- **Arrow functions** for brevity and lexical this
- **Template literals** for string interpolation
- **Async/await** instead of callback pyramids
- **Semicolons optional but consistent**
- **camelCase** for variables and functions
- **PascalCase** for constructors and classes

## 🧪 Testing Strategy

### Test-Driven Development (TDD)

1. **Write the test first** - Define expected behavior before implementation, create a openAPI documentation before creating the rest service, write a test first.
2. **Write minimal code** - Just enough to make the test pass




## 🔧 Configuration Management

### Simple Configuration Approaches

#### Single HTML File Configuration
```javascript
// Configuration object in the HTML file
const CONFIG = {
  appName: 'My Todo App',
  theme: 'light',
  maxTodos: 100,
  storageKey: 'todos-data',
  backupEnabled: true
};
```

#### Node.js Environment Configuration
```javascript
// Load from environment variables or defaults
const config = {
  port: process.env.PORT || 3000,
  nodeEnv: process.env.NODE_ENV || 'development',
  apiKey: process.env.API_KEY || 'dev-key',
  logLevel: process.env.LOG_LEVEL || 'info'
};

module.exports = config;
```

## 🏗️ Data Storage and Validation

#### JSON File Storage (Node.js)
```javascript
const fs = require('fs').promises;
const path = require('path');

class FileStorage {
  constructor(filename) {
    this.filepath = path.join(__dirname, 'data', filename);
  }
  
  async load() {
    try {
      const data = await fs.readFile(this.filepath, 'utf8');
      return JSON.parse(data);
    } catch (error) {
      return []; // Return empty array if file doesn't exist
    }
  }
  
  async save(data) {
    await fs.mkdir(path.dirname(this.filepath), { recursive: true });
    await fs.writeFile(this.filepath, JSON.stringify(data, null, 2));
  }
}

// Usage
const storage = new FileStorage('todos.json');
const todos = await storage.load();
```

### API Documentation

#### OpenAPI Requirements

- **All REST endpoints MUST have OpenAPI documentation**
- **Main specification**: Store in `docs/openapi.yaml`
- **Individual endpoints**: Store detailed docs in `docs/api/endpoints/`
- **Documentation structure**: Follow OpenAPI 3.0+ specification
- **Validation**: Ensure all endpoints have complete request/response schemas
- **Examples**: Include realistic request/response examples for each endpoint

```typescript
import { Router, Request, Response } from 'express';

const router = Router();

/**
 * @openapi
 * /api/v1/products:
 *   get:
 *     summary: List all products
 *     description: Retrieve a paginated list of all active products
 *     parameters:
 *       - in: query
 *         name: skip
 *         schema:
 *           type: integer
 *           minimum: 0
 *         description: Number of products to skip (for pagination)
 *       - in: query
 *         name: limit
 *         schema:
 *           type: integer
 *           minimum: 1
 *           maximum: 100
 *         description: Maximum number of products to return
 *       - in: query
 *         name: category
 *         schema:
 *           type: string
 *         description: Filter by product category
 *     responses:
 *       200:
 *         description: List of products
 *         content:
 *           application/json:
 *             schema:
 *               type: array
 *               items:
 *                 $ref: '#/components/schemas/Product'
 */
router.get('/products', async (req: Request, res: Response) => {
  const { skip = 0, limit = 100, category } = req.query;
  
  // Implementation here
});
```

## ⚠️ Important Notes

- **NEVER ASSUME OR GUESS** - When in doubt, ask for clarification
- **Always verify file paths and module names** before use
- **Keep CLAUDE.md updated** when adding new patterns or dependencies
- **Test your code** - No feature is complete without tests
- **Document your decisions** - Future developers (including yourself) will thank you

---

_This document is a living guide. Update it as the project evolves and new patterns emerge._