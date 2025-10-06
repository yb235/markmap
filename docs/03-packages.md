# Packages

This document provides detailed information about each package in the Markmap monorepo.

## Package Overview

| Package | Version | Purpose | Dependencies |
|---------|---------|---------|--------------|
| markmap-common | Base | Shared utilities and types | npm2url |
| markmap-lib | Core | Markdown transformation | markdown-it, markmap-common, markmap-html-parser |
| markmap-html-parser | Core | HTML to tree conversion | markmap-common |
| markmap-view | Core | Interactive visualization | d3, d3-flextree, markmap-common |
| markmap-render | Core | HTML template generation | markmap-common, markmap-view |
| markmap-cli | App | Command-line interface | All core packages, Hono, commander |
| markmap-autoloader | App | Browser auto-loader | markmap-lib, markmap-view |
| markmap-toolbar | Optional | Interactive toolbar | markmap-view |

---

## markmap-common

**Path**: `packages/markmap-common`

### Purpose
Foundation package providing shared types, utilities, and helper functions used across all Markmap packages.

### Key Exports

#### Types
```typescript
// Node structures
interface IPureNode {
  content: string;
  payload?: { fold?: number; [key: string]: unknown };
  children: IPureNode[];
}

interface INode extends IPureNode {
  state: INodeState;
  children: INode[];
}

interface INodeState {
  id: number;           // Auto-increment ID
  path: string;         // Dot-separated path (e.g., "1.2.3")
  key: string;          // Unique identifier
  depth: number;        // 0-based depth
  size: [number, number];
  rect: { x, y, width, height };
}

// Assets
interface IAssets {
  styles?: CSSItem[];
  scripts?: JSItem[];
}

type JSItem = JSScriptItem | JSIIFEItem;
type CSSItem = CSSStyleItem | CSSStylesheetItem;
```

#### Hook System
```typescript
class Hook<T extends unknown[]> {
  tap(fn: (...args: T) => void): () => void;
  call(...args: T): void;
}
```

#### Asset Loaders
```typescript
// Load JavaScript dynamically
function loadJS(item: JSItem, context?: unknown): Promise<void>

// Load CSS dynamically  
function loadCSS(item: CSSItem): Promise<void>

// Persist assets to HTML strings
function persistJS(items: JSItem[], context?: unknown): string[]
function persistCSS(items: CSSItem[]): string[]
```

#### UrlBuilder
```typescript
class UrlBuilder {
  provider: string;
  providers: Record<string, (path: string) => string>;
  
  setProvider(name: string, factory: (path: string) => string): void;
  getFullUrl(path: string): string;
  async findFastestProvider(): Promise<void>;
}
```

### Key Functions

- **`defer<T>()`**: Create a deferred promise
- **`getId()`**: Generate unique IDs
- **`walkTree()`**: Tree traversal utility
- **`addClass()`**: Class name manipulation
- **`debounce()`**: Debounce function calls
- **`extractAssets()`**: Extract asset URLs from IAssets

### Usage Example
```typescript
import { Hook, UrlBuilder, loadJS } from 'markmap-common';

// Create a hook
const myHook = new Hook<[string]>();
myHook.tap((message) => console.log(message));
myHook.call('Hello'); // Logs: Hello

// Build URLs
const builder = new UrlBuilder();
const url = builder.getFullUrl('d3@7/dist/d3.min.js');
```

---

## markmap-lib

**Path**: `packages/markmap-lib`

### Purpose
Core transformation engine that converts Markdown content into hierarchical data structures suitable for visualization.

### Key Components

#### Transformer Class
```typescript
class Transformer {
  hooks: ITransformHooks;
  md: MarkdownIt;
  assetsMap: Record<string, IAssets>;
  urlBuilder: UrlBuilder;
  plugins: ITransformPlugin[];
  
  constructor(plugins?: ITransformPlugin[]);
  transform(content: string, options?: Partial<IHtmlParserOptions>): ITransformResult;
  getAssets(keys?: string[]): IAssets;
  getUsedAssets(features: IFeatures): IAssets;
  resolveJS(item: JSItem): JSItem;
  resolveCSS(item: CSSItem): CSSItem;
}
```

#### Transform Result
```typescript
interface ITransformResult {
  root: IPureNode;              // Transformed tree
  features: IFeatures;          // Detected features (plugins used)
  frontmatter?: {               // Extracted frontmatter
    title?: string;
    markmap?: Partial<IMarkmapJSONOptions>;
  };
  content: string;              // Original content
}
```

#### Transform Hooks
```typescript
interface ITransformHooks {
  parser: Hook<[MarkdownIt]>;           // Initialize parser
  beforeParse: Hook<[MarkdownIt, ITransformContext]>;  // Before parse
  afterParse: Hook<[MarkdownIt, ITransformContext]>;   // After parse
  retransform: Hook<[]>;                // Force retransform
}
```

### Built-in Plugins

#### 1. **pluginFrontmatter**
- Parses YAML frontmatter from Markdown
- Extracts metadata (title, markmap options)
- Sets context.frontmatter

#### 2. **pluginKatex**
- Renders mathematical equations
- Uses KaTeX library
- Supports `$inline$` and `$$display$$` syntax
- Assets: katex.min.css, katex.min.js

#### 3. **pluginHljs**
- Syntax highlighting for code blocks
- Uses Highlight.js
- Detects language from fence info
- Assets: highlight.js styles and scripts

#### 4. **pluginPrism** (alternative to hljs)
- Syntax highlighting using Prism
- Lighter weight option
- Language-specific loading

#### 5. **pluginNpmUrl**
- Resolves npm package URLs
- Converts package names to CDN links
- Format: `package@version/path`

#### 6. **pluginCheckbox**
- Renders task list checkboxes
- Supports `- [ ]` and `- [x]`
- Interactive in rendered output

#### 7. **pluginSourceLines**
- Tracks source line numbers
- Useful for editors/debugging
- Adds line metadata to nodes

### Usage Example
```typescript
import { Transformer } from 'markmap-lib';

const transformer = new Transformer();
const markdown = `
# Project
## Phase 1
- Task 1
- Task 2
## Phase 2
- Task 3
`;

const { root, features } = transformer.transform(markdown);
const assets = transformer.getUsedAssets(features);

console.log(root);    // Tree structure
console.log(assets);  // Required CSS/JS
```

### Configuration
```typescript
// Custom plugins
import { Transformer, pluginKatex } from 'markmap-lib';

const transformer = new Transformer([
  pluginKatex,
  // Add custom plugin
  {
    name: 'myPlugin',
    transform: (hooks) => {
      hooks.beforeParse.tap((md, context) => {
        // Custom logic
      });
      return { scripts: [], styles: [] };
    }
  }
]);
```

---

## markmap-html-parser

**Path**: `packages/markmap-html-parser`

### Purpose
Converts HTML into hierarchical tree structures based on heading levels.

### Key Function

#### buildTree
```typescript
function buildTree(
  html: string, 
  options?: Partial<IHtmlParserOptions>
): IPureNode
```

**Options**:
```typescript
interface IHtmlParserOptions {
  selector?: string;      // Node selector (default: heading tags)
  getAttrs?: (el: Element) => Record<string, unknown>;
}
```

### Algorithm
1. Parse HTML string to DOM
2. Find all heading elements (h1-h6)
3. Build hierarchy based on heading levels
4. Create tree nodes with content
5. Return root node

### Example
```typescript
import { buildTree } from 'markmap-html-parser';

const html = `
  <h1>Root</h1>
  <h2>Child 1</h2>
  <p>Content</p>
  <h2>Child 2</h2>
`;

const tree = buildTree(html);
// Returns:
// {
//   content: 'Root',
//   children: [
//     { content: 'Child 1<p>Content</p>', children: [] },
//     { content: 'Child 2', children: [] }
//   ]
// }
```

---

## markmap-view

**Path**: `packages/markmap-view`

### Purpose
Interactive visualization engine using D3.js to render mindmaps from tree data.

### Key Components

#### Markmap Class
```typescript
class Markmap {
  options: IMarkmapOptions;
  state: IMarkmapState;
  svg: ID3SVGElement;
  
  constructor(svg: string | SVGElement, opts?: Partial<IMarkmapOptions>);
  
  // Data management
  setData(data?: IPureNode, opts?: Partial<IMarkmapOptions>): Promise<void>;
  setOptions(opts?: Partial<IMarkmapOptions>): void;
  
  // Interactions
  toggleNode(node: INode, recursive?: boolean): Promise<void>;
  fit(maxScale?: number): Promise<void>;
  ensureVisible(node: INode, padding?: Partial<IPadding>): Promise<void>;
  centerNode(node: INode, padding?: Partial<IPadding>): Promise<void>;
  rescale(scale: number): Promise<void>;
  
  // Highlighting
  setHighlight(node?: INode): Promise<void>;
  
  // Utilities
  findElement(node: INode): { data: INode; g: SVGGElement } | undefined;
  destroy(): void;
  
  // Factory
  static create(svg, opts?, data?): Markmap;
}
```

#### Options
```typescript
interface IMarkmapOptions {
  // Display
  duration: number;              // Animation duration (default: 500ms)
  color: (node: INode) => string; // Color function
  paddingX: number;              // Horizontal padding (default: 8)
  spacingHorizontal: number;     // Horizontal spacing (default: 80)
  spacingVertical: number;       // Vertical spacing (default: 5)
  maxWidth: number;              // Max node width (default: 0 = unlimited)
  initialExpandLevel: number;    // Auto-expand depth (default: -1 = all)
  
  // Interaction
  zoom: boolean;                 // Enable zoom (default: true)
  pan: boolean;                  // Enable pan (default: true)
  scrollForPan: boolean;         // Scroll to pan (default: false)
  toggleRecursively: boolean;    // Default toggle mode (default: false)
  
  // Styling
  style: ((id: string) => string) | null;
  embedGlobalCSS: boolean;       // Include base CSS (default: true)
  
  // Layout
  autoFit: boolean;              // Auto-fit on render (default: true)
  fitRatio: number;              // Fit scale factor (default: 0.95)
  maxInitialScale: number;       // Max initial zoom (default: 2)
  lineWidth: (node: INode) => number; // Line width function
}
```

### Features

#### 1. Layout Algorithm
- Uses `d3-flextree` for flexible tree layout
- Calculates node positions based on content size
- Handles variable node widths
- Respects spacing options

#### 2. Interactions
- **Click**: Toggle node expand/collapse
- **Ctrl+Click**: Toggle recursively
- **Zoom**: Mouse wheel or pinch
- **Pan**: Drag or scroll (if enabled)

#### 3. Animations
- Smooth transitions between states
- Enter/update/exit animations
- Configurable duration

#### 4. Rendering
- SVG-based visualization
- Responsive to container size
- Supports custom styling
- Dark mode compatible

### Usage Example
```typescript
import { Markmap } from 'markmap-view';

// Create markmap
const mm = Markmap.create('#markmap', {
  duration: 500,
  maxWidth: 300,
  initialExpandLevel: 2,
  zoom: true,
  pan: true
}, treeData);

// Update data
await mm.setData(newTreeData);

// Interactions
await mm.fit();
await mm.toggleNode(someNode);
await mm.centerNode(someNode);

// Cleanup
mm.destroy();
```

---

## markmap-render

**Path**: `packages/markmap-render`

### Purpose
Generates standalone HTML files with embedded mindmaps.

### Key Function

#### fillTemplate
```typescript
function fillTemplate(
  root: IPureNode | null,
  assets: IAssets,
  extra?: {
    baseJs?: JSItem[];
    jsonOptions?: Partial<IMarkmapJSONOptions>;
    getOptions?: (jsonOptions) => Partial<IMarkmapOptions>;
    urlBuilder?: UrlBuilder;
  }
): string
```

### Process
1. Serialize tree data to JSON
2. Bundle required CSS assets
3. Bundle required JS assets
4. Inject base dependencies (D3.js, markmap-view)
5. Generate initialization script
6. Return complete HTML

### Template Structure
```html
<!DOCTYPE html>
<html>
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Markmap</title>
  <!--CSS-->
</head>
<body>
  <svg id="markmap"></svg>
  <!--JS-->
</body>
</html>
```

### Base Dependencies
```typescript
const baseJsPaths = [
  'd3@7/dist/d3.min.js',
  'markmap-view@VERSION/dist/browser/index.js'
];
```

### Usage Example
```typescript
import { fillTemplate } from 'markmap-render';

const html = fillTemplate(
  treeData,
  { styles: [...], scripts: [...] },
  {
    jsonOptions: { 
      maxWidth: 300,
      initialExpandLevel: 2
    }
  }
);

// Write to file
await fs.writeFile('output.html', html);
```

---

## markmap-cli

**Path**: `packages/markmap-cli`

### Purpose
Command-line interface for generating markmaps from Markdown files.

### Commands

#### Basic Usage
```bash
markmap <input.md>
```

#### Options
```bash
-o, --output <file>    # Output file path
--no-open              # Don't open in browser
--no-toolbar           # Hide toolbar
--offline              # Inline all assets
-w, --watch            # Watch mode (dev server)
--port <port>          # Dev server port
```

### Key Functions

#### createMarkmap
```typescript
async function createMarkmap(options: {
  content?: string;
  output?: string;
  open?: boolean;
  toolbar?: boolean;
  offline?: boolean;
}): Promise<void>
```

**Process**:
1. Create transformer
2. Transform Markdown content
3. Collect required assets
4. Inline assets if offline mode
5. Generate HTML with fillTemplate
6. Write to file
7. Open in browser if requested

#### develop (Watch Mode)
```typescript
async function develop(options: {
  toolbar?: boolean;
  offline?: boolean;
  port?: number;
}): Promise<MarkmapDevServer>
```

**Features**:
- File watcher for auto-reload
- Web server with live updates
- WebSocket for real-time updates
- Multiple file support

### MarkmapDevServer

```typescript
class MarkmapDevServer {
  options: IDevelopOptions;
  providers: Record<string, IContentProvider>;
  
  async setup(): Promise<void>;
  async shutdown(): Promise<void>;
  addProvider(options?: { key?: string; filePath?: string }): IContentProvider;
  delProvider(key: string): void;
}
```

**Endpoints**:
- `GET /` - Main HTML page
- `GET /~api?key=xxx` - Get file state
- `POST /~api?key=xxx` - Update file
- `GET /~client.*` - Static assets

### Asset Management

#### Offline Mode
```typescript
// Fetch and inline all assets
async function fetchAssets(options?: {
  assetsDir?: string;
  verbose?: boolean;
}): Promise<void>
```

- Downloads all required assets
- Stores in local directory
- Replaces URLs with local paths

### Usage Examples

```bash
# Generate HTML
markmap README.md

# Custom output
markmap README.md -o docs/map.html

# Offline mode (all assets inlined)
markmap README.md --offline

# Development mode
markmap README.md --watch --port 3000

# No toolbar
markmap README.md --no-toolbar
```

### Programmatic Usage
```typescript
import { createMarkmap } from 'markmap-cli';

await createMarkmap({
  content: '# My Content\n## Section 1',
  output: 'output.html',
  toolbar: true,
  offline: true,
  open: true
});
```

---

## markmap-autoloader

**Path**: `packages/markmap-autoloader`

### Purpose
Automatically detect and render markmaps in HTML pages.

### Usage

#### 1. Include Script
```html
<script src="https://cdn.jsdelivr.net/npm/markmap-autoloader"></script>
```

#### 2. Add Markdown Content
```html
<div class="markmap">
# My Mindmap
## Section 1
- Item 1
- Item 2
## Section 2
- Item 3
</div>
```

### Configuration
```typescript
interface AutoLoaderOptions {
  baseJs: (string | JSItem)[];
  baseCss: (string | CSSItem)[];
  provider: string | ((path: string) => string);
  onReady: () => void;
  transformPlugins: ITransformPlugin[];
  manual: boolean;        // Manual rendering
  toolbar: boolean;       // Show toolbar
}
```

#### Configure Globally
```javascript
window.markmap = {
  autoLoader: {
    manual: true,
    toolbar: true,
    transformPlugins: [/* custom plugins */]
  }
};
```

### Manual Rendering
```javascript
// Set manual mode
window.markmap = { autoLoader: { manual: true } };

// Render specific elements
import { renderAll, render } from 'markmap-autoloader';
renderAll();  // Render all .markmap elements
render(element);  // Render specific element
```

### Features
- Auto-detection on DOMContentLoaded
- Lazy loading of dependencies
- Plugin support
- Toolbar integration
- Manual control option

---

## markmap-toolbar

**Path**: `packages/markmap-toolbar`

### Purpose
Optional interactive toolbar for mindmap controls.

### Features
- Zoom in/out buttons
- Fit to screen
- Download (SVG, PNG, text)
- Fullscreen toggle
- Expandable/collapsible
- Customizable position

### Usage
```typescript
import { Toolbar } from 'markmap-toolbar';

const toolbar = new Toolbar();
toolbar.attach(markmapInstance);

// Get toolbar assets
const assets = toolbar.getAssets();
```

### Toolbar Items
- **Zoom In**: Increase zoom level
- **Zoom Out**: Decrease zoom level
- **Fit**: Auto-fit to viewport
- **Recurse**: Toggle recursive mode
- **Download**: Export options

### Styling
```css
.markmap-toolbar {
  position: absolute;
  /* customizable */
}
```

---

## Package Dependencies Graph

```
markmap-common (foundation)
    ↓
    ├─→ markmap-lib
    │       ↓
    │   markmap-html-parser
    │
    ├─→ markmap-view
    │       ↓
    │   markmap-toolbar
    │
    └─→ markmap-render
            ↓
        ┌───┴───┐
        ↓       ↓
   markmap-cli  markmap-autoloader
```

## Installation

### Individual Packages
```bash
npm install markmap-lib      # Transformation
npm install markmap-view     # Visualization
npm install markmap-cli      # CLI tool
```

### All Packages
```bash
npm install markmap-lib markmap-view markmap-render
```

## Development

### Build All Packages
```bash
pnpm install
pnpm build
```

### Test
```bash
pnpm test
```

### Lint
```bash
pnpm lint
```

## Next Steps

- See [APIs](./05-apis.md) for detailed API reference
- See [Workflow](./04-workflow.md) for data flow details
- See [Plugins](./06-plugins.md) for creating custom plugins
