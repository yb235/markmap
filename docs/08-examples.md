# Examples

Practical examples demonstrating various Markmap use cases and implementations.

## Table of Contents

- [Basic Examples](#basic-examples)
- [CLI Examples](#cli-examples)
- [Browser Examples](#browser-examples)
- [Advanced Examples](#advanced-examples)
- [Integration Examples](#integration-examples)
- [Real-World Use Cases](#real-world-use-cases)

---

## Basic Examples

### Example 1: Simple Mindmap

**Input (notes.md):**
```markdown
# Learning Web Development

## Frontend
- HTML
- CSS
- JavaScript

## Backend
- Node.js
- Python
- Ruby

## Database
- MongoDB
- PostgreSQL
- Redis
```

**Generate:**
```bash
markmap notes.md
```

**Result:** Interactive mindmap with three main branches.

---

### Example 2: Nested Structure

**Input:**
```markdown
# Company Organization

## Engineering
### Development
#### Frontend Team
- React Developer
- Vue Developer
#### Backend Team
- Node.js Developer
- Python Developer
### QA
- Test Engineer
- Automation Engineer

## Marketing
### Content
- Writer
- Editor
### Design
- UI Designer
- UX Designer
```

**Usage:**
```bash
markmap company.md --no-toolbar
```

---

### Example 3: Task List

**Input:**
```markdown
# Sprint Planning

## In Progress
- [x] Design review
- [x] API implementation
- [ ] Frontend integration

## Planned
- [ ] Testing
- [ ] Documentation
- [ ] Deployment

## Backlog
- [ ] Performance optimization
- [ ] Accessibility improvements
```

**Generate:**
```bash
markmap sprint.md
```

---

## CLI Examples

### Example 4: Development Workflow

**setup.sh:**
```bash
#!/bin/bash

# Create markdown file
cat > project.md << 'EOF'
# Project Roadmap

## Q1 2024
- Feature A
- Feature B

## Q2 2024
- Feature C
- Feature D
EOF

# Start watch mode
markmap project.md --watch --port 8080 &

# Open in browser
sleep 2
open http://localhost:8080

echo "Edit project.md to see live updates!"
```

**Run:**
```bash
chmod +x setup.sh
./setup.sh
```

---

### Example 5: Batch Processing

**generate-all.sh:**
```bash
#!/bin/bash

# Process all markdown files
for file in *.md; do
  output="${file%.md}.html"
  echo "Processing $file → $output"
  markmap "$file" -o "$output" --no-open --offline
done

echo "Generated ${#} mindmaps"
```

**Usage:**
```bash
./generate-all.sh
```

---

### Example 6: Custom Configuration

**build.sh:**
```bash
#!/bin/bash

# Create config file
cat > config.md << 'EOF'
---
title: Custom Mindmap
markmap:
  maxWidth: 300
  initialExpandLevel: 2
  colorFreezeLevel: 2
---

# Custom Styled Mindmap

## Section 1
- Item 1
- Item 2

## Section 2
- Item 3
- Item 4
EOF

# Generate with options
markmap config.md \
  --offline \
  -o custom-output.html \
  --no-toolbar

echo "Generated custom-output.html"
```

---

## Browser Examples

### Example 7: Autoloader Basic

**mindmap.html:**
```html
<!DOCTYPE html>
<html>
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>My Mindmap</title>
  <script src="https://cdn.jsdelivr.net/npm/markmap-autoloader"></script>
  <style>
    body {
      margin: 0;
      font-family: Arial, sans-serif;
    }
    .markmap {
      height: 100vh;
    }
  </style>
</head>
<body>
  <div class="markmap">
# Knowledge Base

## Programming
### Languages
- JavaScript
- Python
- Rust
### Frameworks
- React
- Django
- Actix

## DevOps
### Tools
- Docker
- Kubernetes
### Cloud
- AWS
- Azure
- GCP
  </div>
</body>
</html>
```

---

### Example 8: Programmatic Rendering

**app.html:**
```html
<!DOCTYPE html>
<html>
<head>
  <meta charset="UTF-8">
  <title>Dynamic Markmap</title>
  <script src="https://cdn.jsdelivr.net/npm/d3@7"></script>
  <script src="https://cdn.jsdelivr.net/npm/markmap-view"></script>
  <script src="https://cdn.jsdelivr.net/npm/markmap-lib"></script>
  <style>
    body { margin: 0; }
    #markmap { width: 100vw; height: 100vh; }
    #controls { 
      position: fixed; 
      top: 10px; 
      left: 10px; 
      z-index: 1000;
      background: white;
      padding: 10px;
      border-radius: 4px;
      box-shadow: 0 2px 8px rgba(0,0,0,0.15);
    }
  </style>
</head>
<body>
  <div id="controls">
    <button onclick="zoomIn()">Zoom In</button>
    <button onclick="zoomOut()">Zoom Out</button>
    <button onclick="fit()">Fit</button>
    <button onclick="toggleNode()">Toggle First Child</button>
  </div>
  <svg id="markmap"></svg>
  
  <script>
    const { Transformer } = window.markmap;
    const { Markmap } = window.markmap;
    
    const markdown = `
# Interactive Mindmap

## Features
- Zoom controls
- Pan support
- Node toggling

## Data
- Dynamic content
- Easy updates
    `;
    
    const transformer = new Transformer();
    const { root } = transformer.transform(markdown);
    
    const mm = Markmap.create('#markmap', {
      duration: 500,
      maxWidth: 300
    }, root);
    
    // Control functions
    function zoomIn() {
      const svg = document.querySelector('#markmap');
      const currentScale = mm.svg.node().__zoom?.k || 1;
      mm.rescale(currentScale * 1.2);
    }
    
    function zoomOut() {
      const svg = document.querySelector('#markmap');
      const currentScale = mm.svg.node().__zoom?.k || 1;
      mm.rescale(currentScale / 1.2);
    }
    
    function fit() {
      mm.fit();
    }
    
    function toggleNode() {
      const firstChild = root.children[0];
      mm.toggleNode(firstChild);
    }
  </script>
</body>
</html>
```

---

### Example 9: Multiple Markmaps

**multi.html:**
```html
<!DOCTYPE html>
<html>
<head>
  <meta charset="UTF-8">
  <title>Multiple Markmaps</title>
  <script src="https://cdn.jsdelivr.net/npm/markmap-autoloader"></script>
  <style>
    .container {
      display: grid;
      grid-template-columns: 1fr 1fr;
      gap: 20px;
      padding: 20px;
      height: 100vh;
      box-sizing: border-box;
    }
    .markmap {
      border: 1px solid #ddd;
      border-radius: 8px;
      overflow: hidden;
    }
  </style>
</head>
<body>
  <div class="container">
    <div class="markmap">
# Frontend Stack
## React
- Components
- Hooks
- Context
## Vue
- Composition API
- Reactivity
    </div>
    
    <div class="markmap">
# Backend Stack
## Node.js
- Express
- NestJS
## Python
- Django
- FastAPI
    </div>
  </div>
</body>
</html>
```

---

## Advanced Examples

### Example 10: With Math Equations

**math.md:**
```markdown
---
markmap:
  maxWidth: 400
---

# Mathematics

## Algebra
- Linear equations: $ax + b = 0$
- Quadratic: $ax^2 + bx + c = 0$
- Solution: $$x = \frac{-b \pm \sqrt{b^2-4ac}}{2a}$$

## Calculus
- Derivative: $\frac{df}{dx}$
- Integral: $\int_a^b f(x)dx$
- Fundamental theorem: $$\int_a^b f'(x)dx = f(b) - f(a)$$

## Statistics
- Mean: $\bar{x} = \frac{1}{n}\sum_{i=1}^n x_i$
- Variance: $\sigma^2 = \frac{1}{n}\sum_{i=1}^n (x_i - \bar{x})^2$
```

**Generate:**
```bash
markmap math.md
```

---

### Example 11: With Code Highlighting

**code.md:**
````markdown
# Programming Concepts

## Functions

### JavaScript
```javascript
function fibonacci(n) {
  if (n <= 1) return n;
  return fibonacci(n-1) + fibonacci(n-2);
}
```

### Python
```python
def fibonacci(n):
    if n <= 1:
        return n
    return fibonacci(n-1) + fibonacci(n-2)
```

## Data Structures

### Array Operations
```javascript
const arr = [1, 2, 3];
arr.map(x => x * 2);  // [2, 4, 6]
arr.filter(x => x > 1);  // [2, 3]
```

### Hash Maps
```python
user = {
    'name': 'John',
    'age': 30
}
```
````

**Generate:**
```bash
markmap code.md --offline
```

---

### Example 12: Dynamic Content Loading

**dynamic.html:**
```html
<!DOCTYPE html>
<html>
<head>
  <meta charset="UTF-8">
  <title>Dynamic Markmap</title>
  <script src="https://cdn.jsdelivr.net/npm/d3@7"></script>
  <script src="https://cdn.jsdelivr.net/npm/markmap-view"></script>
  <script src="https://cdn.jsdelivr.net/npm/markmap-lib"></script>
  <style>
    body { margin: 0; }
    #markmap { width: 100%; height: 100vh; }
  </style>
</head>
<body>
  <svg id="markmap"></svg>
  
  <script>
    const { Transformer, Markmap } = window.markmap;
    
    // Simulate loading data from API
    async function loadData() {
      // In real app, fetch from server
      const response = await fetch('/api/mindmap-data');
      return await response.json();
    }
    
    async function updateMarkmap(data) {
      const transformer = new Transformer();
      const { root } = transformer.transform(data.markdown);
      
      if (!window.mm) {
        window.mm = Markmap.create('#markmap', {
          maxWidth: 300
        }, root);
      } else {
        await window.mm.setData(root);
      }
    }
    
    // Initial load
    updateMarkmap({
      markdown: '# Loading...\n## Please wait'
    });
    
    // Update with real data after 1 second
    setTimeout(() => {
      updateMarkmap({
        markdown: `
# Real Data Loaded

## Users
- John Doe
- Jane Smith

## Projects
- Project A
- Project B
        `
      });
    }, 1000);
    
    // Auto-refresh every 5 seconds
    setInterval(async () => {
      const data = await loadData();
      updateMarkmap(data);
    }, 5000);
  </script>
</body>
</html>
```

---

## Integration Examples

### Example 13: React Integration

**MarkmapComponent.jsx:**
```jsx
import React, { useEffect, useRef } from 'react';
import { Transformer } from 'markmap-lib';
import { Markmap } from 'markmap-view';

function MarkmapComponent({ markdown }) {
  const svgRef = useRef(null);
  const mmRef = useRef(null);
  
  useEffect(() => {
    if (!svgRef.current) return;
    
    const transformer = new Transformer();
    const { root } = transformer.transform(markdown);
    
    if (!mmRef.current) {
      mmRef.current = Markmap.create(svgRef.current, {
        duration: 500,
        maxWidth: 300
      }, root);
    } else {
      mmRef.current.setData(root);
    }
    
    return () => {
      if (mmRef.current) {
        mmRef.current.destroy();
        mmRef.current = null;
      }
    };
  }, [markdown]);
  
  return (
    <svg
      ref={svgRef}
      style={{ width: '100%', height: '600px' }}
    />
  );
}

export default MarkmapComponent;
```

**Usage:**
```jsx
import MarkmapComponent from './MarkmapComponent';

function App() {
  const markdown = `
# React Mindmap
## Components
- Functional
- Class-based
## Hooks
- useState
- useEffect
  `;
  
  return (
    <div>
      <h1>My App</h1>
      <MarkmapComponent markdown={markdown} />
    </div>
  );
}
```

---

### Example 14: Vue Integration

**MarkmapComponent.vue:**
```vue
<template>
  <svg ref="svgRef" :style="{ width: '100%', height: height }"></svg>
</template>

<script>
import { ref, onMounted, onUnmounted, watch } from 'vue';
import { Transformer } from 'markmap-lib';
import { Markmap } from 'markmap-view';

export default {
  name: 'MarkmapComponent',
  props: {
    markdown: {
      type: String,
      required: true
    },
    height: {
      type: String,
      default: '600px'
    }
  },
  setup(props) {
    const svgRef = ref(null);
    let mm = null;
    
    const updateMarkmap = () => {
      const transformer = new Transformer();
      const { root } = transformer.transform(props.markdown);
      
      if (!mm) {
        mm = Markmap.create(svgRef.value, {
          duration: 500
        }, root);
      } else {
        mm.setData(root);
      }
    };
    
    onMounted(() => {
      updateMarkmap();
    });
    
    onUnmounted(() => {
      if (mm) {
        mm.destroy();
      }
    });
    
    watch(() => props.markdown, () => {
      updateMarkmap();
    });
    
    return { svgRef };
  }
};
</script>
```

---

### Example 15: Node.js Server

**server.js:**
```javascript
const express = require('express');
const { Transformer } = require('markmap-lib');
const { fillTemplate } = require('markmap-render');
const fs = require('fs').promises;

const app = express();
app.use(express.json());

// Endpoint to generate mindmap HTML
app.post('/api/generate', async (req, res) => {
  try {
    const { markdown } = req.body;
    
    const transformer = new Transformer();
    const { root, features } = transformer.transform(markdown);
    const assets = transformer.getUsedAssets(features);
    
    const html = fillTemplate(root, assets);
    
    res.json({ html, success: true });
  } catch (error) {
    res.status(500).json({ 
      error: error.message,
      success: false 
    });
  }
});

// Endpoint to generate and save file
app.post('/api/save', async (req, res) => {
  try {
    const { markdown, filename } = req.body;
    
    const transformer = new Transformer();
    const { root, features } = transformer.transform(markdown);
    const assets = transformer.getUsedAssets(features);
    
    const html = fillTemplate(root, assets);
    
    await fs.writeFile(filename, html);
    
    res.json({ 
      filename,
      success: true 
    });
  } catch (error) {
    res.status(500).json({ 
      error: error.message,
      success: false 
    });
  }
});

app.listen(3000, () => {
  console.log('Server running on http://localhost:3000');
});
```

**Client usage:**
```javascript
// Generate HTML
const response = await fetch('http://localhost:3000/api/generate', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    markdown: '# My Mindmap\n## Section 1'
  })
});

const { html } = await response.json();

// Save to file
await fetch('http://localhost:3000/api/save', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    markdown: '# My Mindmap\n## Section 1',
    filename: 'output.html'
  })
});
```

---

## Real-World Use Cases

### Example 16: Documentation Site

**docs.md:**
```markdown
# API Documentation

## Authentication
### Endpoints
- POST /auth/login
- POST /auth/logout
- POST /auth/refresh

### Methods
- JWT tokens
- OAuth 2.0
- API keys

## Resources
### Users
- GET /users
- POST /users
- PUT /users/:id
- DELETE /users/:id

### Posts
- GET /posts
- POST /posts
- PUT /posts/:id
- DELETE /posts/:id

## Error Handling
### HTTP Codes
- 200: Success
- 400: Bad Request
- 401: Unauthorized
- 404: Not Found
- 500: Server Error

### Error Format
```json
{
  "error": {
    "code": "ERROR_CODE",
    "message": "Error description"
  }
}
```
```

---

### Example 17: Learning Roadmap

**learning-path.md:**
```markdown
# Full-Stack Developer Roadmap

## Fundamentals
### Programming Basics
- [ ] Variables and data types
- [ ] Control structures
- [ ] Functions
- [ ] OOP concepts

### Web Basics
- [ ] HTML5
- [ ] CSS3
- [ ] JavaScript ES6+
- [ ] DOM manipulation

## Frontend Development
### Core Skills
- [x] React.js
- [x] State management (Redux)
- [ ] TypeScript
- [ ] Testing (Jest, RTL)

### Advanced
- [ ] Next.js
- [ ] GraphQL
- [ ] Performance optimization
- [ ] Accessibility (a11y)

## Backend Development
### Server-side
- [x] Node.js
- [x] Express.js
- [ ] Database design
- [ ] REST APIs

### Advanced
- [ ] Microservices
- [ ] Message queues
- [ ] Caching strategies
- [ ] Security best practices

## DevOps
### Basics
- [ ] Git & GitHub
- [ ] Linux basics
- [ ] Docker
- [ ] CI/CD

### Cloud
- [ ] AWS fundamentals
- [ ] Kubernetes
- [ ] Monitoring & logging
- [ ] Infrastructure as Code
```

**Generate with progress tracking:**
```bash
markmap learning-path.md --watch
```

---

### Example 18: Meeting Minutes

**meeting-template.md:**
```markdown
# Team Meeting - {{DATE}}

## Attendees
- John Doe (Lead)
- Jane Smith (Developer)
- Bob Johnson (Designer)

## Agenda
### 1. Sprint Review
- Completed: 15 stories
- In progress: 3 stories
- Blocked: 1 story

### 2. Technical Discussion
- Architecture decision
- Database schema changes
- API versioning strategy

### 3. Planning
- Next sprint goals
- Resource allocation
- Timeline adjustments

## Decisions Made
- ✅ Approved database migration
- ✅ Selected React for frontend
- ⏳ Pending: Cloud provider selection

## Action Items
- [ ] Update technical spec (John) - Due: Friday
- [ ] Review PR #123 (Jane) - Due: Today
- [ ] Create mockups (Bob) - Due: Next Monday

## Next Meeting
- Date: Next Monday 2PM
- Topics: Sprint planning, Architecture review
```

**Automation script:**
```bash
#!/bin/bash

# Generate meeting mindmap with current date
DATE=$(date +"%Y-%m-%d")
sed "s/{{DATE}}/$DATE/g" meeting-template.md | \
  markmap - -o "meeting-$DATE.html"

echo "Generated meeting-$DATE.html"
```

---

## Best Practices from Examples

### 1. Structure
- Use clear heading hierarchy
- Keep nesting reasonable (max 4-5 levels)
- Group related items under common headings

### 2. Content
- Be concise - mindmaps work best with brief items
- Use bullet points for lists
- Leverage Markdown formatting

### 3. Visualization
- Set `maxWidth` for long text
- Use `initialExpandLevel` for large maps
- Configure colors for different depths

### 4. Performance
- Limit nodes in browser (< 1000 for smooth performance)
- Use offline mode for standalone files
- Lazy load assets when possible

### 5. Maintenance
- Keep Markdown in version control
- Generate HTML as needed
- Use templates for consistency

---

## Additional Resources

### Code Repositories
- [Markmap Examples Repo](https://github.com/markmap/markmap-examples) - More examples
- [VSCode Extension](https://marketplace.visualstudio.com/items?itemName=gera2ld.markmap-vscode)

### Community Examples
- [Obsidian Integration](https://github.com/markmap/markmap-obsidian)
- [Gatsby Plugin](https://github.com/markmap/gatsby-remark-markmap)

### Live Demos
- [Official REPL](https://markmap.js.org/repl)
- [CodePen Collection](https://codepen.io/collection/markmap)

---

## Next Steps

1. Try examples from this guide
2. Modify them for your use case
3. Share your creations with the community
4. Contribute examples back to the project

For more information:
- [Getting Started](./07-getting-started.md)
- [API Reference](./05-apis.md)
- [Plugin Development](./06-plugins.md)
