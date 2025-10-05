# Overview

## What is Markmap?

Markmap is a powerful tool that transforms Markdown documents into interactive, visual mindmaps. It takes hierarchical Markdown content (headings, lists, etc.) and renders them as an interactive tree visualization using D3.js.

## Key Features

### 1. **Markdown to Mindmap Conversion**
- Automatically converts Markdown headings and lists into hierarchical mindmap structures
- Preserves Markdown formatting (bold, italic, links, code, etc.)
- Supports various Markdown features through plugins

### 2. **Interactive Visualization**
- Pan and zoom functionality
- Click to expand/collapse nodes
- Smooth animations and transitions
- Responsive design that adapts to different screen sizes

### 3. **Extensible Plugin System**
- Built-in plugins for:
  - **Frontmatter**: Extract metadata from Markdown frontmatter
  - **KaTeX**: Render mathematical equations
  - **Highlight.js/Prism**: Syntax highlighting for code blocks
  - **Checkboxes**: Interactive task lists
  - **NPM URLs**: Automatic URL resolution for npm packages
  - **Source Lines**: Track source line numbers

### 4. **Multiple Integration Options**
- **CLI Tool**: Generate standalone HTML files from Markdown
- **JavaScript Library**: Integrate into web applications
- **Auto-loader**: Automatically render markmaps in HTML pages
- **Development Server**: Live reload during development

### 5. **Offline Support**
- Can inline all assets for offline usage
- No external dependencies required in offline mode

## Use Cases

### 1. **Documentation**
Transform technical documentation into navigable mindmaps for easier understanding of complex topics.

### 2. **Note-Taking**
Convert hierarchical notes into visual mindmaps for better knowledge organization.

### 3. **Project Planning**
Create visual project structures and task breakdowns from Markdown outlines.

### 4. **Education**
Present course materials, lecture notes, or study guides in an interactive format.

### 5. **Knowledge Management**
Build personal or team knowledge bases with visual navigation.

## How It Works (High-Level)

```
┌─────────────┐
│  Markdown   │
│   Content   │
└──────┬──────┘
       │
       ▼
┌─────────────────┐
│   Transformer   │  ← Plugins process content
│  (markmap-lib)  │
└──────┬──────────┘
       │
       ▼
┌─────────────────┐
│  Data Structure │  ← Hierarchical tree (IPureNode)
│  (JSON Tree)    │
└──────┬──────────┘
       │
       ▼
┌─────────────────┐
│   Renderer      │  ← D3.js visualization
│ (markmap-view)  │
└──────┬──────────┘
       │
       ▼
┌─────────────────┐
│  Interactive    │
│    Mindmap      │
└─────────────────┘
```

## Core Concepts

### 1. **Transformation**
The process of converting Markdown text into a hierarchical data structure:
- Markdown is parsed using markdown-it
- Plugins enhance the parsing (add features like math, code highlighting)
- HTML output is converted into a tree structure
- Each node contains content and metadata

### 2. **Rendering**
The process of visualizing the data structure:
- Uses D3.js for SVG-based visualization
- Implements tree layout algorithms (flextree)
- Handles user interactions (zoom, pan, click)
- Manages animations and transitions

### 3. **Assets Management**
Handling external dependencies (CSS/JS):
- URL resolution for CDN resources
- Asset inlining for offline mode
- Version management for dependencies
- Plugin-specific asset loading

### 4. **Plugin System**
Extensibility mechanism:
- Plugins hook into the transformation process
- Can modify Markdown parsing behavior
- Can add assets (CSS/JS) for rendering
- Enable features like math rendering, code highlighting

## Supported Markdown Features

### Basic Formatting
- **Headings**: `# H1` to `###### H6`
- **Bold**: `**bold**` or `__bold__`
- **Italic**: `*italic*` or `_italic_`
- **Code**: `` `inline code` `` or ``` code blocks ```
- **Links**: `[text](url)`
- **Images**: `![alt](src)` (embedded in content)

### Advanced Features (via Plugins)
- **Math**: `$inline math$` or `$$display math$$` (KaTeX plugin)
- **Code Highlighting**: Language-specific syntax highlighting (hljs/prism)
- **Checkboxes**: `- [ ]` unchecked, `- [x]` checked
- **Frontmatter**: YAML metadata at the beginning of files

### Structure
- **Lists**: Unordered (`-`, `*`, `+`) and ordered (`1.`, `2.`)
- **Nested Lists**: Create hierarchy through indentation
- **Headings**: Create top-level structure

## Architecture Overview

Markmap is built as a monorepo with multiple packages, each serving a specific purpose:

- **markmap-lib**: Core transformation logic (Markdown → Data)
- **markmap-view**: Visualization engine (Data → Interactive UI)
- **markmap-cli**: Command-line interface
- **markmap-render**: HTML template generation
- **markmap-common**: Shared utilities and types
- **markmap-autoloader**: Automatic rendering in web pages
- **markmap-toolbar**: Optional interactive toolbar
- **markmap-html-parser**: HTML to tree structure converter

## Next Steps

- Continue to [Architecture](./02-architecture.md) for detailed system design
- Jump to [Getting Started](./07-getting-started.md) to begin using Markmap
- Explore [Examples](./08-examples.md) for practical use cases
