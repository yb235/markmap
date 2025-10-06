# Workflow

This document explains the complete workflow of how Markmap processes Markdown content and renders it as an interactive mindmap.

## Overview

The Markmap workflow consists of three main phases:

1. **Transformation Phase**: Markdown → Tree Data Structure
2. **Rendering Phase**: Tree Data → Visual Mindmap
3. **Interaction Phase**: User Interactions → Dynamic Updates

## Detailed Workflow

### Phase 1: Transformation (Markdown → Tree)

#### Step 1: Initialize Transformer

```typescript
const transformer = new Transformer(plugins);
```

**What happens**:
1. Creates transform hooks (parser, beforeParse, afterParse, retransform)
2. Instantiates all plugins
3. Initializes markdown-it parser
4. Calls `parser` hook to let plugins configure markdown-it
5. Builds assets map from plugin configurations

#### Step 2: Transform Content

```typescript
const result = transformer.transform(markdownContent);
```

**Detailed Process**:

```
Input: "# Main\n## Sub\n- Item"
    ↓
┌─────────────────────────────────────┐
│ 1. Create Transform Context         │
│    - features: {}                   │
│    - content: original markdown     │
│    - frontmatter: undefined         │
└─────────────────┬───────────────────┘
                  ↓
┌─────────────────────────────────────┐
│ 2. beforeParse Hook                 │
│    Plugins process content:         │
│    - pluginFrontmatter: Extract     │
│      YAML metadata if present       │
│    - Other plugins: Preprocess      │
└─────────────────┬───────────────────┘
                  ↓
┌─────────────────────────────────────┐
│ 3. markdown-it Parse                │
│    Convert Markdown to HTML:        │
│    "<h1>Main</h1>                   │
│     <h2>Sub</h2>                    │
│     <ul><li>Item</li></ul>"         │
└─────────────────┬───────────────────┘
                  ↓
┌─────────────────────────────────────┐
│ 4. afterParse Hook                  │
│    Plugins analyze HTML:            │
│    - pluginKatex: Detect math       │
│    - pluginHljs: Detect code blocks │
│    - Set context.features flags     │
└─────────────────┬───────────────────┘
                  ↓
┌─────────────────────────────────────┐
│ 5. buildTree (HTML → Tree)          │
│    Parse HTML structure:            │
│    - Find headings (h1-h6)          │
│    - Build hierarchy                │
│    - Create tree nodes              │
└─────────────────┬───────────────────┘
                  ↓
┌─────────────────────────────────────┐
│ 6. cleanNode (Optimize)             │
│    Remove redundant structure:      │
│    - Skip nodes with single child   │
│    - Flatten unnecessary nesting    │
└─────────────────┬───────────────────┘
                  ↓
Output: {
  root: IPureNode,
  features: { katex: true, ... },
  frontmatter: { title: '...' },
  content: original
}
```

#### Step 3: Collect Assets

```typescript
const assets = transformer.getUsedAssets(result.features);
```

**Process**:
1. Filter plugins by detected features
2. Collect CSS/JS from active plugins
3. Resolve URLs using UrlBuilder
4. Return combined assets

**Example**:
```typescript
// If features = { katex: true, hljs: true }
assets = {
  styles: [
    { type: 'stylesheet', data: { href: 'katex.min.css' } },
    { type: 'stylesheet', data: { href: 'highlight.css' } }
  ],
  scripts: [
    { type: 'script', data: { src: 'katex.min.js' } },
    { type: 'script', data: { src: 'highlight.min.js' } }
  ]
}
```

### Phase 2: Rendering (Tree → Visual)

#### CLI Path: Generate HTML File

```
┌─────────────────────────────────────┐
│ createMarkmap({                     │
│   content: markdown,                │
│   output: 'map.html',               │
│   offline: true                     │
│ })                                  │
└─────────────────┬───────────────────┘
                  ↓
┌─────────────────────────────────────┐
│ 1. Transform Content                │
│    const { root, features,          │
│              frontmatter } =        │
│      transformer.transform(content) │
└─────────────────┬───────────────────┘
                  ↓
┌─────────────────────────────────────┐
│ 2. Collect Assets                   │
│    - Base: d3, markmap-view         │
│    - Optional: toolbar              │
│    - Features: katex, hljs, etc.    │
└─────────────────┬───────────────────┘
                  ↓
┌─────────────────────────────────────┐
│ 3. Inline Assets (if offline)       │
│    - Fetch remote resources         │
│    - Convert to inline scripts      │
│    - Embed in HTML                  │
└─────────────────┬───────────────────┘
                  ↓
┌─────────────────────────────────────┐
│ 4. Fill Template                    │
│    fillTemplate(root, assets, {     │
│      jsonOptions: frontmatter       │
│    })                               │
└─────────────────┬───────────────────┘
                  ↓
┌─────────────────────────────────────┐
│ 5. Generate HTML                    │
│    <!DOCTYPE html>                  │
│    <html>                           │
│      <head>                         │
│        <style>...</style>           │
│      </head>                        │
│      <body>                         │
│        <svg id="markmap"></svg>     │
│        <script>                     │
│          const data = {...};        │
│          Markmap.create('#markmap', │
│            options, data);          │
│        </script>                    │
│      </body>                        │
│    </html>                          │
└─────────────────┬───────────────────┘
                  ↓
┌─────────────────────────────────────┐
│ 6. Write File                       │
│    fs.writeFile(output, html)       │
└─────────────────────────────────────┘
```

#### Browser Path: Direct Rendering

```
┌─────────────────────────────────────┐
│ 1. Create Markmap Instance          │
│    const mm = new Markmap(          │
│      '#svg-element',                │
│      options                        │
│    )                                │
└─────────────────┬───────────────────┘
                  ↓
┌─────────────────────────────────────┐
│ 2. Set Data                         │
│    mm.setData(treeData)             │
└─────────────────┬───────────────────┘
                  ↓
┌─────────────────────────────────────┐
│ 3. Initialize Data                  │
│    _initializeData(node):           │
│    - Assign unique IDs              │
│    - Calculate depths               │
│    - Set paths (e.g., "1.2.3")      │
│    - Preload colors                 │
│    - Apply fold states              │
└─────────────────┬───────────────────┘
                  ↓
┌─────────────────────────────────────┐
│ 4. Render Data                      │
│    renderData()                     │
└─────────────────┬───────────────────┘
                  ↓
┌─────────────────────────────────────┐
│ 5. Relayout                         │
│    _relayout():                     │
│    - Measure node sizes             │
│    - Run flextree layout            │
│    - Calculate positions            │
│    - Update node.state.rect         │
└─────────────────┬───────────────────┘
                  ↓
┌─────────────────────────────────────┐
│ 6. D3 Update Pattern                │
│    - Enter: New nodes               │
│    - Update: Changed nodes          │
│    - Exit: Removed nodes            │
└─────────────────┬───────────────────┘
                  ↓
┌─────────────────────────────────────┐
│ 7. Render Elements                  │
│    - Create/update <g> groups       │
│    - Create/update <line> elements  │
│    - Create/update <circle> toggles │
│    - Create/update <foreignObject>  │
│      with HTML content              │
│    - Create/update <path> links     │
└─────────────────┬───────────────────┘
                  ↓
┌─────────────────────────────────────┐
│ 8. Animations                       │
│    - Transition positions           │
│    - Fade in/out                    │
│    - Smooth transforms              │
└─────────────────┬───────────────────┘
                  ↓
┌─────────────────────────────────────┐
│ 9. Auto Fit (if enabled)            │
│    fit():                           │
│    - Calculate bounding box         │
│    - Compute scale factor           │
│    - Center in viewport             │
└─────────────────────────────────────┘
```

### Phase 3: Interaction (User → Updates)

#### Click to Toggle Node

```
User clicks node circle
    ↓
┌─────────────────────────────────────┐
│ handleClick(event, node)            │
│    - Check modifiers (Ctrl/Cmd)     │
│    - Determine recursive mode       │
└─────────────────┬───────────────────┘
                  ↓
┌─────────────────────────────────────┐
│ toggleNode(node, recursive)         │
│    - Get current fold state         │
│    - Calculate new fold value       │
│    - If recursive: walk tree and    │
│      apply to all descendants       │
│    - Else: toggle single node       │
└─────────────────┬───────────────────┘
                  ↓
┌─────────────────────────────────────┐
│ renderData(originNode)              │
│    - Re-run layout with new state   │
│    - Animate from origin position   │
└─────────────────────────────────────┘
```

#### Zoom and Pan

```
User scrolls/drags
    ↓
┌─────────────────────────────────────┐
│ D3 Zoom Behavior                    │
│    - Capture event                  │
│    - Calculate transform            │
└─────────────────┬───────────────────┘
                  ↓
┌─────────────────────────────────────┐
│ handleZoom(event)                   │
│    - Apply transform to <g> element │
│    - Update zoom state              │
└─────────────────────────────────────┘
```

#### Programmatic Navigation

```
Code calls ensureVisible(node)
    ↓
┌─────────────────────────────────────┐
│ findElement(node)                   │
│    - Locate DOM element for node    │
└─────────────────┬───────────────────┘
                  ↓
┌─────────────────────────────────────┐
│ Calculate Visibility                │
│    - Get node position in SVG space │
│    - Transform to screen space      │
│    - Check viewport bounds          │
└─────────────────┬───────────────────┘
                  ↓
┌─────────────────────────────────────┐
│ Calculate Pan Offset                │
│    - Determine required translation │
│    - Respect padding                │
└─────────────────┬───────────────────┘
                  ↓
┌─────────────────────────────────────┐
│ Animate Pan                         │
│    - Transition to new transform    │
│    - Smooth animation               │
└─────────────────────────────────────┘
```

## CLI Workflows

### Workflow 1: Static Generation

```bash
markmap input.md -o output.html
```

```
1. Read Markdown file
    ↓
2. Create Transformer
    ↓
3. Transform content
    ↓
4. Collect assets
    ↓
5. Generate HTML template
    ↓
6. Write output file
    ↓
7. (Optional) Open in browser
```

### Workflow 2: Development Mode

```bash
markmap input.md --watch
```

```
1. Start MarkmapDevServer
    ↓
2. Create FileSystemProvider
    - Initial file read
    - Setup file watcher
    ↓
3. Start HTTP server
    - Serve HTML template
    - Serve API endpoints
    ↓
4. Browser connects
    - Loads initial state
    - Establishes SSE connection
    ↓
5. File changes detected
    - Read updated content
    - Transform content
    - Notify browser via SSE
    ↓
6. Browser updates
    - Fetch new data
    - Re-render mindmap
```

### Workflow 3: Offline Mode

```bash
markmap input.md --offline
```

```
1. Transform Markdown
    ↓
2. Collect all assets
    ↓
3. For each asset URL:
    - Fetch content
    - Convert to inline script/style
    ↓
4. Generate HTML with inlined assets
    ↓
5. Write standalone file
    (No external dependencies)
```

## Autoloader Workflow

### Page Load Sequence

```html
<script src="markmap-autoloader.js"></script>
<div class="markmap">
# Content
</div>
```

```
1. Script loads
    ↓
2. Wait for DOMContentLoaded
    ↓
3. Find all .markmap elements
    ↓
4. For each element:
    ├─→ Load markmap-lib (if needed)
    ├─→ Load markmap-view (if needed)
    ├─→ Create Transformer
    ├─→ Transform markdown content
    ├─→ Load required assets
    ├─→ Create Markmap instance
    └─→ Render mindmap
```

## Data Flow Diagram

### Complete Flow

```
┌─────────────┐
│   Markdown  │
│     File    │
└──────┬──────┘
       │
       ▼
┌─────────────────────────────┐
│     Transformer             │
│                             │
│  ┌────────────────────┐     │
│  │ beforeParse Hooks  │     │
│  └─────────┬──────────┘     │
│            ↓                │
│  ┌────────────────────┐     │
│  │   markdown-it      │     │
│  │   (MD → HTML)      │     │
│  └─────────┬──────────┘     │
│            ↓                │
│  ┌────────────────────┐     │
│  │  afterParse Hooks  │     │
│  └─────────┬──────────┘     │
│            ↓                │
│  ┌────────────────────┐     │
│  │   buildTree        │     │
│  │  (HTML → Tree)     │     │
│  └─────────┬──────────┘     │
│            ↓                │
│  ┌────────────────────┐     │
│  │   cleanNode        │     │
│  └────────────────────┘     │
└─────────────┬───────────────┘
              │
              ▼
   ┌────────────────────┐
   │  IPureNode + Meta  │
   │   + IAssets        │
   └──────────┬─────────┘
              │
   ┌──────────┴──────────┐
   │                     │
   ▼                     ▼
┌────────────┐    ┌──────────────┐
│ Renderer   │    │  Markmap     │
│ (CLI/SSR)  │    │  (Browser)   │
└─────┬──────┘    └──────┬───────┘
      │                  │
      ▼                  ▼
┌────────────┐    ┌──────────────────┐
│ HTML File  │    │  Interactive SVG │
└────────────┘    └──────────────────┘
```

## Plugin Workflow

### Plugin Lifecycle

```
1. Plugin Registration
    ↓
2. Transformer Creation
    - Plugin.transform() called
    - Returns IAssets
    - Hooks configured
    ↓
3. Content Transformation
    - beforeParse: Preprocess
    - Parser: Markdown → HTML
    - afterParse: Feature detection
    ↓
4. Asset Collection
    - Check features
    - Collect plugin assets
    ↓
5. Rendering
    - Load plugin assets
    - Render with features
```

### Example: KaTeX Plugin Flow

```
Content: "The formula $E=mc^2$ is famous"
    ↓
┌─────────────────────────────────────┐
│ beforeParse Hook (KaTeX)            │
│    - No preprocessing needed        │
└─────────────────┬───────────────────┘
                  ↓
┌─────────────────────────────────────┐
│ markdown-it Parse                   │
│    - markdown-it-katex processes    │
│      $...$ syntax                   │
│    - Generates <span class="katex"> │
└─────────────────┬───────────────────┘
                  ↓
┌─────────────────────────────────────┐
│ afterParse Hook (KaTeX)             │
│    - Detect katex elements in HTML  │
│    - Set features.katex = true      │
└─────────────────┬───────────────────┘
                  ↓
┌─────────────────────────────────────┐
│ Asset Collection                    │
│    - Because features.katex = true  │
│    - Include katex CSS/JS           │
└─────────────────┬───────────────────┘
                  ↓
┌─────────────────────────────────────┐
│ Rendering                           │
│    - Load KaTeX library             │
│    - KaTeX renders equations        │
└─────────────────────────────────────┘
```

## Performance Optimizations

### 1. Lazy Asset Loading
```
Only load assets for detected features
├─→ No math? No KaTeX
├─→ No code? No highlight.js
└─→ Minimal bundle size
```

### 2. Tree Optimization
```
cleanNode() removes:
├─→ Single-child nodes
├─→ Empty intermediate nodes
└─→ Redundant structure
```

### 3. Efficient Updates
```
D3 Enter/Update/Exit:
├─→ Only update changed nodes
├─→ Reuse existing elements
└─→ Minimal DOM manipulation
```

### 4. Debounced Events
```
File watcher:
├─→ Debounce 100ms
└─→ Batch multiple changes

Resize observer:
├─→ Debounce 100ms
└─→ Throttle layout recalc
```

## Error Handling

### Transformation Errors
```
try {
  transformer.transform(content)
} catch (err) {
  // Markdown parse error
  // Invalid syntax
  // Plugin error
}
```

### Rendering Errors
```
- Invalid tree structure → Fallback to error display
- Missing assets → Graceful degradation
- Browser incompatibility → Feature detection
```

### Dev Server Errors
```
- File not found → 404 response
- Watch error → Log and continue
- Transform error → Show in browser
```

## Next Steps

- See [APIs](./05-apis.md) for detailed API reference
- See [Plugins](./06-plugins.md) for creating custom plugins
- See [Examples](./08-examples.md) for practical implementations
