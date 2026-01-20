---
name: vanilla-ui
description: Build frontend UI components using vanilla JavaScript, HTML5, and Tailwind CSS. Use this skill when creating user interface elements, handling DOM interactions, or implementing client-side rendering logic.
compatibility: Modern browsers (ES6+), Tailwind CSS
---

# Vanilla UI Skill

## When to use this skill

Use this skill when:
- Creating new UI components
- Adding user interactions (click handlers, form submissions)
- Rendering data to the DOM
- Styling elements with Tailwind CSS

## Architecture Overview

```
public/
├── index.html           # Single page, semantic HTML5
├── css/
│   └── styles.css       # Tailwind imports + custom CSS
└── js/
    ├── app.js           # Main entry, initializes app
    ├── components/      # UI components (DOM manipulation)
    │   └── todoList.js
    └── api/             # Backend API calls
        └── todoApi.js
```

## Creating a Component

### Step 1: Create component file in `public/js/components/`

```javascript
// public/js/components/todoList.js

const TodoList = {
  container: null,

  init(containerSelector) {
    this.container = document.querySelector(containerSelector);
    this.bindEvents();
  },

  bindEvents() {
    this.container.addEventListener('click', (e) => {
      if (e.target.matches('[data-action="delete"]')) {
        const id = e.target.closest('[data-id]').dataset.id;
        this.onDelete(id);
      }
    });
  },

  render(todos) {
    if (todos.length === 0) {
      this.container.innerHTML = this.renderEmpty();
      return;
    }
    this.container.innerHTML = todos.map(todo => this.renderItem(todo)).join('');
  },

  renderItem(todo) {
    const completedClass = todo.completed ? 'line-through text-gray-400' : '';
    return `
      <li data-id="${todo.id}" class="flex items-center gap-3 p-3 bg-white rounded-lg shadow">
        <input
          type="checkbox"
          ${todo.completed ? 'checked' : ''}
          data-action="toggle"
          class="w-5 h-5 rounded border-gray-300"
        >
        <span class="flex-1 ${completedClass}">${this.escapeHtml(todo.text)}</span>
        <button data-action="delete" class="text-red-500 hover:text-red-700">
          Delete
        </button>
      </li>
    `;
  },

  renderEmpty() {
    return `
      <li class="text-center text-gray-500 py-8">
        No todos yet. Add one above!
      </li>
    `;
  },

  escapeHtml(text) {
    const div = document.createElement('div');
    div.textContent = text;
    return div.innerHTML;
  },

  onDelete(id) {},
  onToggle(id) {}
};
```

### Step 2: Use component in `app.js`

```javascript
// public/js/app.js

import TodoList from './components/todoList.js';
import TodoApi from './api/todoApi.js';

const App = {
  async init() {
    TodoList.init('#todo-list');
    TodoList.onDelete = (id) => this.deleteTodo(id);
    TodoList.onToggle = (id) => this.toggleTodo(id);

    await this.loadTodos();
    this.bindFormEvents();
  },

  async loadTodos() {
    const todos = await TodoApi.getAll();
    TodoList.render(todos);
  },

  bindFormEvents() {
    const form = document.querySelector('#todo-form');
    form.addEventListener('submit', async (e) => {
      e.preventDefault();
      const input = form.querySelector('input');
      if (input.value.trim()) {
        await TodoApi.create(input.value);
        input.value = '';
        await this.loadTodos();
      }
    });
  }
};

document.addEventListener('DOMContentLoaded', () => App.init());
```

## HTML Structure

Use semantic HTML5 with Tailwind classes:

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Todo App</title>
  <script src="https://cdn.tailwindcss.com"></script>
  <link rel="stylesheet" href="css/styles.css">
</head>
<body class="bg-gray-100 min-h-screen">
  <main class="max-w-md mx-auto py-8 px-4">
    <h1 class="text-3xl font-bold text-center mb-8">Todo App</h1>

    <form id="todo-form" class="flex gap-2 mb-6">
      <input
        type="text"
        placeholder="Add a todo..."
        class="flex-1 px-4 py-2 rounded-lg border border-gray-300 focus:outline-none focus:ring-2 focus:ring-blue-500"
      >
      <button
        type="submit"
        class="px-6 py-2 bg-blue-500 text-white rounded-lg hover:bg-blue-600"
      >
        Add
      </button>
    </form>

    <ul id="todo-list" class="space-y-2">
      <!-- Todos rendered here -->
    </ul>
  </main>

  <script type="module" src="js/app.js"></script>
</body>
</html>
```

## Conventions

### Component Pattern
- One component per file in `public/js/components/`
- Components are objects with `init()`, `render()`, and event methods
- Use event delegation on container elements
- Always escape user content with `escapeHtml()`

### Tailwind CSS Classes
- Use utility classes directly in HTML/templates
- Common patterns:
  - Layout: `flex`, `grid`, `items-center`, `justify-between`
  - Spacing: `p-4`, `m-2`, `gap-3`, `space-y-2`
  - Colors: `bg-gray-100`, `text-blue-500`, `border-gray-300`
  - States: `hover:bg-blue-600`, `focus:ring-2`

### Data Attributes
- Use `data-action` for event handlers: `data-action="delete"`
- Use `data-id` for element identification: `data-id="${todo.id}"`

### File Naming
- Components: `{name}.js` in camelCase (e.g., `todoList.js`)
- Keep files under 200 lines

## Security

Always escape HTML to prevent XSS:

```javascript
escapeHtml(text) {
  const div = document.createElement('div');
  div.textContent = text;
  return div.innerHTML;
}
```

Never use `innerHTML` with unescaped user input.
