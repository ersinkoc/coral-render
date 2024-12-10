# CoralRender - Browser-Based Template Engine

## Overview
CoralRender is a lightweight, browser-focused JavaScript template engine that can be included directly in web pages. It provides powerful templating features with a focus on ease of use and performance. The library is designed to work without any dependencies and can be used by simply including a single JavaScript file.

## Technical Specifications
- File Size: < 15KB minified and gzipped
- Browser Support: Modern browsers + IE11 (with polyfills)
- Dependencies: Zero
- Module Formats: UMD, ESM, IIFE
- Performance Target: < 3ms render time for simple templates
- Memory Usage: < 5MB for typical applications
- Test Coverage: > 95%
- Bundle Size Budget: 
  - Core: < 15KB
  - Virtual DOM: < 5KB (optional)
  - Each Plugin: < 3KB

## Development Standards

### 1. Code Style & Quality
- **Formatting:**
  - Indentation: 2 spaces
  - Max line length: 80 characters
  - UTF-8 encoding
  - LF line endings
  - No trailing whitespace

- **Naming Conventions:**
  ```javascript
  // Classes: PascalCase
  class TemplateParser {}
  
  // Methods & Variables: camelCase
  const templateString = '';
  function parseTemplate() {}
  
  // Constants: UPPER_SNAKE_CASE
  const MAX_CACHE_SIZE = 1000;
  
  // Private members: _prefixed
  class Parser {
    _privateMethod() {}
  }
  
  // File names: kebab-case
  // coral-render.js
  // virtual-dom.js
  ```

- **Documentation:**
  ```javascript
  /**
   * Parses a template string into an AST.
   * 
   * @param {string} template - The template string to parse
   * @param {Object} options - Parser options
   * @param {boolean} options.strict - Enable strict mode
   * @returns {AST} The parsed Abstract Syntax Tree
   * @throws {ParseError} If template syntax is invalid
   * 
   * @example
   * const ast = parser.parse('{{variable}}', { strict: true });
   */
  parse(template, options = {}) {
    // Implementation
  }
  ```

### 2. Design Patterns

#### Creational Patterns
1. **Factory Method:**
   ```javascript
   class NodeFactory {
     createNode(type, props) {
       switch (type) {
         case 'if': return new IfNode(props);
         case 'each': return new EachNode(props);
         case 'var': return new VariableNode(props);
         default: throw new Error(`Unknown node type: ${type}`);
       }
     }
   }
   ```

2. **Builder:**
   ```javascript
   class TemplateBuilder {
     constructor() {
       this.reset();
     }
     
     reset() {
       this.template = new Template();
     }
     
     addVariables(vars) {
       this.template.variables = vars;
       return this;
     }
     
     addHelpers(helpers) {
       this.template.helpers = helpers;
       return this;
     }
     
     build() {
       const result = this.template;
       this.reset();
       return result;
     }
   }
   ```

3. **Singleton:**
   ```javascript
   class Cache {
     static instance = null;
     
     static getInstance() {
       if (!Cache.instance) {
         Cache.instance = new Cache();
       }
       return Cache.instance;
     }
   }
   ```

#### Structural Patterns
1. **Decorator:**
   ```javascript
   class CacheDecorator {
     constructor(renderer) {
       this.renderer = renderer;
       this.cache = new Map();
     }
     
     render(template, data) {
       const key = this.getCacheKey(template, data);
       if (this.cache.has(key)) {
         return this.cache.get(key);
       }
       
       const result = this.renderer.render(template, data);
       this.cache.set(key, result);
       return result;
     }
   }
   ```

2. **Composite:**
   ```javascript
   class CompositeNode {
     constructor() {
       this.children = [];
     }
     
     add(node) {
       this.children.push(node);
     }
     
     render(data) {
       return this.children
         .map(child => child.render(data))
         .join('');
     }
   }
   ```

3. **Proxy:**
   ```javascript
   class TemplateProxy {
     constructor(realTemplate) {
       this.realTemplate = realTemplate;
       this.cache = new Map();
     }
     
     render(data) {
       const key = JSON.stringify(data);
       if (!this.cache.has(key)) {
         this.cache.set(key, this.realTemplate.render(data));
       }
       return this.cache.get(key);
     }
   }
   ```

#### Behavioral Patterns
1. **Observer:**
   ```javascript
   class EventEmitter {
     constructor() {
       this.listeners = new Map();
     }
     
     on(event, callback) {
       if (!this.listeners.has(event)) {
         this.listeners.set(event, new Set());
       }
       this.listeners.get(event).add(callback);
     }
     
     emit(event, data) {
       const callbacks = this.listeners.get(event) || new Set();
       callbacks.forEach(callback => callback(data));
     }
   }
   ```

2. **Strategy:**
   ```javascript
   class RenderStrategy {
     static strategies = {
       simple: new SimpleRenderer(),
       virtual: new VirtualDOMRenderer(),
       stream: new StreamRenderer()
     };
     
     static getStrategy(type) {
       return this.strategies[type] || this.strategies.simple;
     }
   }
   ```

3. **Chain of Responsibility:**
   ```javascript
   class MiddlewareChain {
     constructor() {
       this.middlewares = [];
     }
     
     use(middleware) {
       this.middlewares.push(middleware);
       return this;
     }
     
     async execute(context, next) {
       let index = -1;
       
       const dispatch = async (i) => {
         if (i <= index) {
           throw new Error('Next called multiple times');
         }
         
         index = i;
         const middleware = this.middlewares[i];
         
         if (i === this.middlewares.length) {
           return next();
         }
         
         return middleware(context, () => dispatch(i + 1));
       };
       
       return dispatch(0);
     }
   }
   ```

### 3. Error Handling
```javascript
// Custom error classes
class CoralError extends Error {
  constructor(message) {
    super(message);
    this.name = this.constructor.name;
    Error.captureStackTrace(this, this.constructor);
  }
}

class ParseError extends CoralError {
  constructor(message, line, column) {
    super(`${message} at line ${line}, column ${column}`);
    this.line = line;
    this.column = column;
  }
}

class RenderError extends CoralError {
  constructor(message, template) {
    super(message);
    this.template = template;
  }
}

// Error handling middleware
const errorHandler = async (ctx, next) => {
  try {
    await next();
  } catch (err) {
    if (err instanceof ParseError) {
      console.error(`Template parse error: ${err.message}`);
      // Handle parse errors
    } else if (err instanceof RenderError) {
      console.error(`Render error: ${err.message}`);
      // Handle render errors
    } else {
      console.error(`Unexpected error: ${err.message}`);
      // Handle unknown errors
    }
  }
};
```

### 4. Testing Strategy

#### Unit Tests
```javascript
// Using Jest
describe('TemplateParser', () => {
  let parser;
  
  beforeEach(() => {
    parser = new TemplateParser();
  });
  
  describe('parse', () => {
    test('should parse variable interpolation', () => {
      const template = '{{variable}}';
      const ast = parser.parse(template);
      
      expect(ast).toMatchSnapshot();
      expect(ast.type).toBe('variable');
      expect(ast.name).toBe('variable');
    });
    
    test('should throw on invalid syntax', () => {
      const template = '{{invalid}}}}';
      expect(() => parser.parse(template)).toThrow(ParseError);
    });
  });
});
```

#### Integration Tests
```javascript
describe('CoralRender Integration', () => {
  let coral;
  
  beforeEach(() => {
    coral = new CoralRender();
    document.body.innerHTML = '<div id="app"></div>';
  });
  
  test('should render template with data', async () => {
    const template = '<div>{{message}}</div>';
    const data = { message: 'Hello' };
    
    await coral.render(template, data, '#app');
    
    expect(document.querySelector('#app').innerHTML)
      .toBe('<div>Hello</div>');
  });
});
```

#### Performance Tests
```javascript
describe('Performance', () => {
  test('should render large lists within 16ms', () => {
    const items = Array.from({ length: 1000 }, (_, i) => ({
      id: i,
      name: `Item ${i}`
    }));
    
    const start = performance.now();
    coral.render(template, { items });
    const duration = performance.now() - start;
    
    expect(duration).toBeLessThan(16);
  });
});
```

### 5. Performance Optimization Guidelines

#### 1. DOM Operations
```javascript
// Bad
items.forEach(item => {
  const el = document.createElement('div');
  el.textContent = item.name;
  container.appendChild(el);
});

// Good
const fragment = document.createDocumentFragment();
items.forEach(item => {
  const el = document.createElement('div');
  el.textContent = item.name;
  fragment.appendChild(el);
});
container.appendChild(fragment);
```

#### 2. String Concatenation
```javascript
// Bad
let html = '';
items.forEach(item => {
  html += `<div>${item.name}</div>`;
});

// Good
const html = items
  .map(item => `<div>${item.name}</div>`)
  .join('');
```

#### 3. Memory Management
```javascript
class TemplateCache {
  constructor(maxSize = 100) {
    this.maxSize = maxSize;
    this.cache = new Map();
    this.usage = new Map();
  }
  
  get(key) {
    if (this.cache.has(key)) {
      this.usage.set(key, Date.now());
      return this.cache.get(key);
    }
    return null;
  }
  
  set(key, value) {
    if (this.cache.size >= this.maxSize) {
      this.evictLRU();
    }
    
    this.cache.set(key, value);
    this.usage.set(key, Date.now());
  }
  
  evictLRU() {
    let oldest = Infinity;
    let oldestKey = null;
    
    this.usage.forEach((time, key) => {
      if (time < oldest) {
        oldest = time;
        oldestKey = key;
      }
    });
    
    if (oldestKey) {
      this.cache.delete(oldestKey);
      this.usage.delete(oldestKey);
    }
  }
}
```

### 6. Security Guidelines

#### 1. XSS Prevention
```javascript
class SecurityManager {
  constructor() {
    this.escapeHTML = str => {
      const div = document.createElement('div');
      div.textContent = str;
      return div.innerHTML;
    };
    
    this.allowedTags = new Set([
      'div', 'span', 'p', 'a', 'b', 'i', 'strong', 'em'
    ]);
    
    this.allowedAttrs = new Set([
      'id', 'class', 'href', 'title', 'alt'
    ]);
  }
  
  sanitize(html) {
    const doc = new DOMParser().parseFromString(html, 'text/html');
    this.sanitizeNode(doc.body);
    return doc.body.innerHTML;
  }
  
  sanitizeNode(node) {
    Array.from(node.children).forEach(child => {
      if (!this.allowedTags.has(child.tagName.toLowerCase())) {
        node.removeChild(child);
        return;
      }
      
      Array.from(child.attributes).forEach(attr => {
        if (!this.allowedAttrs.has(attr.name)) {
          child.removeAttribute(attr.name);
        }
      });
      
      this.sanitizeNode(child);
    });
  }
}
```

#### 2. Safe Template Compilation
```javascript
class TemplateCompiler {
  compile(template) {
    const ast = this.parse(template);
    this.validate(ast);
    return this.generate(ast);
  }
  
  validate(ast) {
    // Check for unsafe patterns
    this.checkForScriptTags(ast);
    this.checkForDangerousAttrs(ast);
    this.checkForUnsafeUrls(ast);
  }
  
  checkForScriptTags(ast) {
    ast.traverse(node => {
      if (node.type === 'element' && node.tag === 'script') {
        throw new SecurityError('Script tags are not allowed');
      }
    });
  }
  
  checkForDangerousAttrs(ast) {
    const dangerous = ['onerror', 'onclick', 'onload'];
    ast.traverse(node => {
      if (node.type === 'attribute' && 
          dangerous.includes(node.name.toLowerCase())) {
        throw new SecurityError(`Dangerous attribute: ${node.name}`);
      }
    });
  }
  
  checkForUnsafeUrls(ast) {
    ast.traverse(node => {
      if (node.type === 'attribute' && 
          (node.name === 'href' || node.name === 'src')) {
        const value = node.value.toLowerCase();
        if (value.startsWith('javascript:') || 
            value.startsWith('data:') ||
            value.startsWith('vbscript:')) {
          throw new SecurityError(`Unsafe URL: ${value}`);
        }
      }
    });
  }
}
```

## Development Workflow

### 1. Git Workflow
```bash
# Feature development
git checkout -b feature/new-feature
git add .
git commit -m "feat: add new feature"
git push origin feature/new-feature

# Bug fixes
git checkout -b fix/bug-description
git add .
git commit -m "fix: fix bug description"
git push origin fix/bug-description

# Commit message format
# type(scope): description
#
# Types:
# - feat: New feature
# - fix: Bug fix
# - docs: Documentation
# - style: Code style changes
# - refactor: Code refactoring
# - perf: Performance improvements
# - test: Tests
# - chore: Build, tools, etc.
```

### 2. Code Review Guidelines
- Check for:
  - Code style compliance
  - Test coverage
  - Performance implications
  - Security considerations
  - Documentation
  - Error handling
  - Browser compatibility

### 3. Release Process
1. Version Bump:
   ```bash
   npm version patch|minor|major
   ```

2. Build Process:
   ```javascript
   // rollup.config.js
   export default {
     input: 'src/index.js',
     output: [
       {
         file: 'dist/coral-render.js',
         format: 'umd',
         name: 'CoralRender'
       },
       {
         file: 'dist/coral-render.esm.js',
         format: 'esm'
       }
     ],
     plugins: [
       resolve(),
       commonjs(),
       babel({
         exclude: 'node_modules/**'
       }),
       terser({
         compress: {
           pure_getters: true,
           unsafe: true,
           unsafe_comps: true
         }
       })
     ]
   };
   ```

3. Release Checklist:
   - [ ] All tests passing
   - [ ] Documentation updated
   - [ ] CHANGELOG.md updated
   - [ ] Bundle size within limits
   - [ ] Performance benchmarks passing
   - [ ] Browser compatibility verified
   - [ ] Security audit passed

### 4. Documentation
- API Documentation
- Usage Guides
- Examples
- Migration Guides
- Contributing Guidelines
- Security Policy

## Project Timeline
1. Phase 1 (Weeks 1-2):
   - Core template engine
   - Basic parsing
   - Variable interpolation
   - Helper system

2. Phase 2 (Weeks 3-4):
   - Control structures
   - DOM integration
   - Event handling
   - Error handling

3. Phase 3 (Weeks 5-6):
   - Performance optimization
   - Caching system
   - Virtual DOM (optional)
   - Browser compatibility

4. Phase 4 (Weeks 7-8):
   - Plugin system
   - Documentation
   - Testing
   - First release

## Future Enhancements
1. Server-Side Rendering Support
2. TypeScript Support
3. React/Vue Integration
4. Build-time Optimization Tools
5. Developer Tools/Extensions
6. Performance Monitoring
7. Automated Testing Tools
8. CI/CD Integration

## Template Syntax & Usage Guide

### 1. Basic Variable Interpolation
```html
<!-- Simple variables -->
<div>{{name}}</div>                     <!-- Output: <div>John</div> -->
<span>{{user.age}}</span>               <!-- Output: <span>25</span> -->
<p>{{company.address.city}}</p>         <!-- Output: <p>New York</p> -->

<!-- Array access -->
<div>{{items.[0].name}}</div>           <!-- Output: <div>First Item</div> -->
<span>{{users.[5].email}}</span>        <!-- Output: <span>user5@email.com</span> -->

<!-- Expressions -->
{{user.name.toUpperCase()}}             <!-- Output: JOHN DOE -->
{{item.price * 1.2}}                    <!-- Output: 24 -->
{{user.age >= 18 ? 'Adult' : 'Minor'}}  <!-- Output: Adult -->
{{~sum array}}                          <!-- Output: 15 -->
```

### 2. Built-in Helpers
```html
<!-- String Helpers -->
{{~uppercase "hello"}}                  <!-- Output: HELLO -->
{{~lowercase "WORLD"}}                  <!-- Output: world -->
{{~capitalize "john doe"}}              <!-- Output: John Doe -->
{{~truncate text 100}}                  <!-- Output: Long text... -->
{{~trim "  hello  "}}                   <!-- Output: hello -->
{{~replace "hi-there" "-" " "}}         <!-- Output: hi there -->

<!-- Number Helpers -->
{{~currency 99.99 "USD"}}               <!-- Output: $99.99 -->
{{~number 1234.567 2}}                  <!-- Output: 1,234.57 -->
{{~percentage 0.125}}                   <!-- Output: 12.5% -->
{{~round 3.14159 2}}                    <!-- Output: 3.14 -->

<!-- Date Helpers -->
{{~formatDate date "DD/MM/YYYY"}}       <!-- Output: 25/12/2023 -->
{{~timeAgo date}}                       <!-- Output: 2 hours ago -->
{{~calendar date}}                      <!-- Output: Today at 2:30 PM -->
{{~dayOfWeek date}}                     <!-- Output: Monday -->

<!-- Array Helpers -->
{{~join items ", "}}                    <!-- Output: item1, item2, item3 -->
{{~sort items "name"}}                  <!-- Output: [Sorted array by name] -->
{{~filter items "active"}}              <!-- Output: [Only active items] -->
{{~map users "name"}}                   <!-- Output: [Array of names] -->
{{~first array}}                        <!-- Output: First element -->
{{~last array}}                         <!-- Output: Last element -->
{{~count items}}                        <!-- Output: 5 -->

<!-- Logic Helpers -->
{{~eq value1 value2}}                   <!-- Output: true/false -->
{{~gt value1 value2}}                   <!-- Output: true/false -->
{{~lt value1 value2}}                   <!-- Output: true/false -->
{{~and value1 value2}}                  <!-- Output: true/false -->
{{~or value1 value2}}                   <!-- Output: true/false -->
{{~not value}}                          <!-- Output: true/false -->

<!-- Object Helpers -->
{{~keys object}}                        <!-- Output: [Array of keys] -->
{{~values object}}                      <!-- Output: [Array of values] -->
{{~json object}}                        <!-- Output: {"key": "value"} -->
{{~has object "key"}}                   <!-- Output: true/false -->
```

### 3. Control Structures
```html
<!-- If Conditions -->
{{#if user.isAdmin}}
  <div>Welcome Admin!</div>
{{else if user.isModerator}}
  <div>Welcome Moderator!</div>
{{else}}
  <div>Welcome User!</div>
{{/if}}

<!-- Unless (Inverse If) -->
{{#unless user.isLoggedIn}}
  <div>Please log in</div>
{{/unless}}

<!-- Each Loops -->
{{#each item in items}}
  <div class="item">
    <span>{{@index}}</span>             <!-- Loop index -->
    <span>{{item.name}}</span>
    {{#if @first}}First Item!{{/if}}    <!-- Special first item -->
    {{#if @last}}Last Item!{{/if}}      <!-- Special last item -->
  </div>
{{else}}
  <div>No items found</div>             <!-- Empty case -->
{{/each}}

<!-- With Block -->
{{#with user}}
  <div>{{name}}</div>                   <!-- Scoped to user -->
  <div>{{email}}</div>
  <div>{{address.city}}</div>
{{/with}}
```

### 4. Partials
```html
<!-- Define partial -->
{{#partial "userCard"}}
  <div class="card">
    <h3>{{name}}</h3>
    <p>{{email}}</p>
  </div>
{{/partial}}

<!-- Use partial -->
{{> userCard}}

<!-- Partial with context -->
{{> userCard user}}

<!-- Partial with custom data -->
{{> userCard name="John" email="john@example.com"}}
```

### 5. Event Handling
```html
<!-- Basic Events -->
<button onclick="{{~onClick}}">Click Me</button>
<input onchange="{{~onChange}}" value="{{value}}">
<form onsubmit="{{~onSubmit}}">...</form>

<!-- Event with Parameters -->
<button onclick="{{~onClick @index item.id}}">Delete</button>

<!-- Event Delegation -->
<div data-click="{{~onClick}}">
  <button>Click delegated</button>
</div>
```

### 6. Two-Way Binding
```html
<!-- Input Binding -->
<input value="{{user.name}}" data-bind="user.name">

<!-- Select Binding -->
<select data-bind="selectedOption">
  {{#each option in options}}
    <option value="{{option.value}}">{{option.label}}</option>
  {{/each}}
</select>

<!-- Checkbox Binding -->
<input type="checkbox" checked="{{user.active}}" data-bind="user.active">
```

### 7. Component Usage
```html
<!-- Basic Component -->
<coral-card 
  title="{{item.title}}"
  description="{{item.description}}"
  image="{{item.image}}"
></coral-card>

<!-- List Component -->
<coral-list items="{{items}}" template="item">
  {{#partial "item"}}
    <div>{{name}}</div>
  {{/partial}}
</coral-list>

<!-- Form Component -->
<coral-form onsubmit="{{~onSubmit}}">
  <coral-input name="username" value="{{form.username}}"></coral-input>
  <coral-select name="country" options="{{countries}}"></coral-select>
  <button type="submit">Submit</button>
</coral-form>
```

### 8. Complete Examples

#### Basic Todo List
```html
<div id="todo-app">
  <!-- Form -->
  <form onsubmit="{{~addTodo}}">
    <input value="{{newTodo}}" data-bind="newTodo" placeholder="New todo">
    <button type="submit">Add</button>
  </form>

  <!-- List -->
  <ul class="todo-list">
    {{#each todo in todos}}
      <li class="{{#if todo.completed}}completed{{/if}}">
        <input 
          type="checkbox" 
          checked="{{todo.completed}}"
          onchange="{{~toggleTodo todo.id}}"
        >
        <span>{{todo.text}}</span>
        <button onclick="{{~deleteTodo todo.id}}">Delete</button>
      </li>
    {{else}}
      <li>No todos yet!</li>
    {{/each}}
  </ul>

  <!-- Footer -->
  <div class="footer">
    <span>{{~count todos}} items left</span>
    <button onclick="{{~clearCompleted}}">Clear Completed</button>
  </div>
</div>

<script>
const app = new CoralRender({
  el: '#todo-app',
  data: {
    todos: [],
    newTodo: ''
  },
  helpers: {
    addTodo(e) {
      e.preventDefault();
      this.todos.push({
        id: Date.now(),
        text: this.newTodo,
        completed: false
      });
      this.newTodo = '';
    },
    toggleTodo(id) {
      const todo = this.todos.find(t => t.id === id);
      if (todo) todo.completed = !todo.completed;
    },
    deleteTodo(id) {
      this.todos = this.todos.filter(t => t.id !== id);
    },
    clearCompleted() {
      this.todos = this.todos.filter(t => !t.completed);
    }
  }
});
</script>
```

#### User Dashboard
```html
<div id="dashboard">
  <!-- Header -->
  <header class="dashboard-header">
    {{#with user}}
      <div class="user-info">
        <img src="{{avatar}}" alt="{{name}}">
        <h2>{{~capitalize name}}</h2>
        <span>{{email}}</span>
      </div>
    {{/with}}
  </header>

  <!-- Stats -->
  <div class="stats-grid">
    {{#each stat in stats}}
      <div class="stat-card">
        <h3>{{stat.label}}</h3>
        <div class="stat-value">
          {{#if stat.isCurrency}}
            {{~currency stat.value "USD"}}
          {{else if stat.isPercentage}}
            {{~percentage stat.value}}
          {{else}}
            {{~number stat.value}}
          {{/if}}
        </div>
        <div class="stat-change {{#if stat.change > 0}}positive{{else}}negative{{/if}}">
          {{stat.change}}%
        </div>
      </div>
    {{/each}}
  </div>

  <!-- Recent Activity -->
  <div class="activity-list">
    <h3>Recent Activity</h3>
    {{#each activity in activities}}
      <div class="activity-item">
        <span class="time">{{~timeAgo activity.date}}</span>
        <span class="type">{{activity.type}}</span>
        <span class="description">{{activity.description}}</span>
      </div>
    {{/each}}
  </div>
</div>

<script>
const dashboard = new CoralRender({
  el: '#dashboard',
  data: {
    user: {
      name: 'john doe',
      email: 'john@example.com',
      avatar: 'avatar.jpg'
    },
    stats: [
      { label: 'Revenue', value: 15234.50, isCurrency: true, change: 12.5 },
      { label: 'Users', value: 1234, change: 23.1 },
      { label: 'Conversion', value: 0.123, isPercentage: true, change: -5.2 }
    ],
    activities: [
      {
        date: '2023-12-25T10:30:00',
        type: 'Sale',
        description: 'New sale: $99.99'
      }
      // More activities...
    ]
  }
});
</script>
```