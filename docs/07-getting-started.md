# Getting Started

Welcome to Markmap! This guide will help you start visualizing your Markdown as interactive mindmaps.

## Table of Contents

- [Quick Start](#quick-start)
- [Installation](#installation)
- [Basic Usage](#basic-usage)
- [CLI Usage](#cli-usage)
- [Browser Usage](#browser-usage)
- [Configuration](#configuration)
- [Tips and Tricks](#tips-and-tricks)

## Quick Start

### Option 1: Try Online (No Installation)

Visit [https://markmap.js.org/repl](https://markmap.js.org/repl) to try Markmap immediately in your browser.

### Option 2: Use CLI (Recommended for Beginners)

```bash
# Install globally
npm install -g markmap-cli

# Create a markdown file
echo "# My First Mindmap
## Branch 1
- Leaf 1
- Leaf 2
## Branch 2
- Leaf 3" > example.md

# Generate HTML
markmap example.md

# Opens in browser automatically!
```

### Option 3: HTML with Autoloader

Create an HTML file:

```html
<!DOCTYPE html>
<html>
<head>
  <meta charset="UTF-8">
  <title>My Markmap</title>
  <script src="https://cdn.jsdelivr.net/npm/markmap-autoloader"></script>
</head>
<body>
  <div class="markmap">
# My Project

## Planning
- Define goals
- Set timeline
- Assign resources

## Execution
- Development
- Testing
- Deployment

## Review
- Analyze results
- Document lessons
- Plan improvements
  </div>
</body>
</html>
```

Open in browser - done!

---

## Installation

### CLI Tool

```bash
# Global installation (recommended for CLI usage)
npm install -g markmap-cli

# Or use with npx (no installation)
npx markmap input.md
```

### JavaScript Library

```bash
# For Node.js projects
npm install markmap-lib markmap-view

# For browser projects (via CDN)
# See Browser Usage section
```

### Development Dependencies

```bash
# All packages (for development)
npm install markmap-lib markmap-view markmap-render markmap-common
```

---

## Basic Usage

### Your First Markdown

Create a file `notes.md`:

```markdown
# Project Overview

## Phase 1: Planning
- Requirements gathering
- Design mockups
- Technical specification

## Phase 2: Development
- Frontend development
- Backend development
- Integration

## Phase 3: Testing
- Unit tests
- Integration tests
- User acceptance testing

## Phase 4: Deployment
- Staging environment
- Production deployment
- Monitoring setup
```

### Generate Mindmap

```bash
markmap notes.md
```

This creates `notes.html` and opens it in your browser.

### What You'll See

An interactive mindmap with:
- **Zoom**: Scroll to zoom in/out
- **Pan**: Click and drag to move around
- **Collapse/Expand**: Click circles to toggle branches
- **Smooth Animations**: Watch nodes expand and contract

---

## CLI Usage

### Basic Commands

```bash
# Generate HTML from Markdown
markmap input.md

# Specify output file
markmap input.md -o output.html

# Don't open browser automatically
markmap input.md --no-open

# Remove toolbar
markmap input.md --no-toolbar
```

### Advanced Options

```bash
# Offline mode (inline all assets)
markmap input.md --offline

# Development mode (watch for changes)
markmap input.md --watch

# Development mode with custom port
markmap input.md --watch --port 8080
```

### Watch Mode

Perfect for live editing:

```bash
markmap notes.md --watch
```

Now edit `notes.md` in your text editor, and the browser updates automatically!

### Offline Mode

Create a standalone file with no external dependencies:

```bash
markmap notes.md --offline -o standalone.html
```

Share `standalone.html` - works without internet!

---

## Browser Usage

### Method 1: Autoloader (Easiest)

```html
<!DOCTYPE html>
<html>
<head>
  <meta charset="UTF-8">
  <script src="https://cdn.jsdelivr.net/npm/markmap-autoloader"></script>
</head>
<body>
  <div class="markmap">
# Your Markdown Here
  </div>
</body>
</html>
```

### Method 2: Programmatic (More Control)

```html
<!DOCTYPE html>
<html>
<head>
  <meta charset="UTF-8">
  <script src="https://cdn.jsdelivr.net/npm/d3@7"></script>
  <script src="https://cdn.jsdelivr.net/npm/markmap-view"></script>
  <script src="https://cdn.jsdelivr.net/npm/markmap-lib"></script>
  <style>
    #markmap {
      width: 100vw;
      height: 100vh;
    }
  </style>
</head>
<body>
  <svg id="markmap"></svg>
  <script>
    const { Transformer } = window.markmap;
    const transformer = new Transformer();
    
    const markdown = `
# Main Topic
## Subtopic 1
- Point 1
- Point 2
## Subtopic 2
- Point 3
    `;
    
    const { root } = transformer.transform(markdown);
    const { Markmap } = window.markmap;
    Markmap.create('#markmap', null, root);
  </script>
</body>
</html>
```

### Method 3: With Module Bundler

```javascript
// app.js
import { Transformer } from 'markmap-lib';
import { Markmap } from 'markmap-view';

const transformer = new Transformer();
const { root } = transformer.transform(markdownContent);

const mm = Markmap.create('#markmap', {
  duration: 500,
  maxWidth: 300
}, root);
```

---

## Configuration

### Markmap Options

```javascript
const options = {
  // Animation duration (ms)
  duration: 500,
  
  // Maximum node width (0 = unlimited)
  maxWidth: 0,
  
  // Auto-expand depth (-1 = all, 0 = root only)
  initialExpandLevel: -1,
  
  // Node spacing
  spacingHorizontal: 80,
  spacingVertical: 5,
  paddingX: 8,
  
  // Interactions
  zoom: true,
  pan: true,
  scrollForPan: false,
  toggleRecursively: false,
  
  // Auto-fit to viewport
  autoFit: true,
  fitRatio: 0.95,
  
  // Styling
  embedGlobalCSS: true,
  
  // Color function
  color: (node) => {
    const colors = ['#5e91f2', '#41b883', '#ff6b6b', '#4ecdc4'];
    return colors[node.state.depth % colors.length];
  }
};

Markmap.create('#markmap', options, treeData);
```

### Frontmatter Configuration

Add configuration in your Markdown:

```markdown
---
title: My Mindmap
markmap:
  maxWidth: 300
  initialExpandLevel: 2
  colorFreezeLevel: 2
---

# Content starts here
```

### Transformer Configuration

```javascript
import { Transformer, pluginKatex, pluginHljs } from 'markmap-lib';

// Custom plugins
const transformer = new Transformer([
  pluginKatex,
  pluginHljs
]);

// Custom URL builder
transformer.urlBuilder.provider = 'unpkg';

const result = transformer.transform(markdown);
```

---

## Tips and Tricks

### 1. Structure Your Content

**Good:**
```markdown
# Main Topic
## Category 1
- Item 1
- Item 2
## Category 2
- Item 3
```

**Not Recommended:**
```markdown
# Title
Random text
More random text
- A list item
```

**Why?** Markmap works best with hierarchical structures (headings and lists).

### 2. Use Markdown Features

```markdown
# Styled Content

## Formatting
- **Bold text**
- *Italic text*
- `Code snippet`
- [Links](https://example.com)

## Code Blocks
```javascript
function greet() {
  return "Hello!";
}
```

## Math (with KaTeX)
- Inline: $E=mc^2$
- Display: $$\int_0^1 x dx = \frac{1}{2}$$

## Tasks
- [ ] Todo item
- [x] Completed item
```

### 3. Control Node Expansion

```markdown
# Root (always visible)
## Auto-expand to level 2
### This level collapsed by default
#### Deep nesting collapsed
```

Set `initialExpandLevel: 2` to expand first two levels only.

### 4. Organize Large Mindmaps

```markdown
# Company
## Departments
### Engineering
#### Frontend Team
- Developer 1
- Developer 2
#### Backend Team
- Developer 3
- Developer 4
### Marketing
#### Content
- Writer 1
#### Design
- Designer 1
```

Click circles to navigate large structures.

### 5. Custom Styling

```html
<style>
  .markmap {
    background: #1a1a1a;
  }
  .markmap-node {
    cursor: pointer;
  }
  .markmap-foreign {
    padding: 10px;
  }
</style>
```

### 6. Keyboard Shortcuts

When using toolbar:
- **+** / **-**: Zoom in/out
- **F**: Fit to screen
- **R**: Toggle recursive mode

### 7. Export Options

With toolbar enabled:
- **Download SVG**: Vector format
- **Download PNG**: Raster image
- **Copy Text**: Plain text outline

### 8. Development Workflow

```bash
# Terminal 1: Watch mode
markmap notes.md --watch

# Terminal 2: Edit with your favorite editor
vim notes.md

# Browser auto-refreshes on save!
```

### 9. Integration with Note-Taking Apps

Many apps support Markdown export:
- **Obsidian**: Export note → run markmap
- **Notion**: Export as Markdown → visualize
- **Roam Research**: Export → convert to mindmap

### 10. Version Control

```bash
# Track markdown files
git add notes.md

# Ignore generated HTML
echo "*.html" >> .gitignore

# Regenerate HTML anytime
markmap notes.md
```

---

## Common Use Cases

### 1. Meeting Notes

```markdown
# Team Meeting - 2024-01-15

## Agenda
- Project updates
- Budget review
- Q&A

## Decisions
- Approved budget increase
- New hire approved
- Changed deadline

## Action Items
- [ ] Update project plan (John)
- [ ] Review candidates (Sarah)
- [ ] Notify stakeholders (Mike)
```

### 2. Study Notes

```markdown
# Biology - Cell Structure

## Prokaryotic Cells
### Components
- Cell membrane
- Cytoplasm
- DNA (no nucleus)
### Examples
- Bacteria
- Archaea

## Eukaryotic Cells
### Components
- Nucleus
- Organelles
  - Mitochondria
  - Endoplasmic reticulum
### Examples
- Animal cells
- Plant cells
```

### 3. Project Planning

```markdown
# Website Redesign Project

## Research Phase
- User surveys
- Competitor analysis
- Analytics review

## Design Phase
- Wireframes
- Mockups
- Style guide

## Development Phase
- Frontend
- Backend
- Testing

## Launch Phase
- Staging review
- Production deploy
- Marketing campaign
```

### 4. Knowledge Base

```markdown
# Programming Languages

## Compiled Languages
### C/C++
- High performance
- System programming
- Manual memory management
### Rust
- Memory safety
- Modern syntax
- Growing ecosystem

## Interpreted Languages
### Python
- Easy to learn
- Extensive libraries
- Slower execution
### JavaScript
- Web standard
- Node.js for backend
- Large community
```

---

## Troubleshooting

### Issue: Mindmap doesn't render

**Check:**
1. Valid Markdown syntax
2. At least one heading (`#`)
3. Proper HTML structure (if using autoloader)

### Issue: Math not rendering

**Solution:**
```markdown
---
markmap:
  # Math enabled by default, but verify
---

# Content with $math$
```

### Issue: Watch mode not updating

**Solution:**
```bash
# Kill and restart
markmap notes.md --watch --port 3001
```

### Issue: Offline mode file too large

**Solution:**
Use selective plugins:
```javascript
// Only include needed plugins
const transformer = new Transformer([
  pluginFrontmatter
  // Skip katex, hljs if not needed
]);
```

### Issue: Custom styles not applying

**Solution:**
Use frontmatter:
```markdown
---
markmap:
  style: |
    .markmap-node { font-size: 16px; }
---
```

---

## Next Steps

1. **Learn More**: 
   - [Architecture](./02-architecture.md) - Understand how it works
   - [APIs](./05-apis.md) - Detailed API reference
   - [Plugins](./06-plugins.md) - Extend functionality

2. **See Examples**:
   - [Examples](./08-examples.md) - Complete code examples

3. **Get Help**:
   - [GitHub Issues](https://github.com/markmap/markmap/issues)
   - [Gitter Chat](https://gitter.im/gera2ld/markmap)

4. **Contribute**:
   - Report bugs
   - Suggest features
   - Submit pull requests

Happy mind mapping! 🎉
