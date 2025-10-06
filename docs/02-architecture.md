# Architecture

## System Architecture

Markmap follows a modular, layered architecture designed for flexibility and extensibility. The system is organized as a monorepo with distinct packages, each responsible for specific functionality.

## Architecture Diagram

```
                         ┌──────────────────────────────────┐
                         │     User Interfaces              │
                         ├──────────────────────────────────┤
                         │  CLI  │  Browser  │  VSCode Ext  │
                         └────┬──────────┬─────────┬────────┘
                              │          │         │
        ┌─────────────────────┴──────────┴─────────┴────────────────┐
        │                                                             │
        ▼                                                             ▼
┌────────────────┐                                         ┌──────────────────┐
│  markmap-cli   │                                         │ markmap-autoloader│
└───────┬────────┘                                         └────────┬─────────┘
        │                                                            │
        │  ┌─────────────────────────────────────────────────┐      │
        └─▶│          Core Transformation Layer              │◀─────┘
           ├─────────────────────────────────────────────────┤
           │            markmap-lib                          │
           │  ┌──────────────────────────────────────────┐   │
           │  │  Transformer                             │   │
           │  │  - Markdown Parser (markdown-it)         │   │
           │  │  - Plugin System                         │   │
           │  │  - Asset Management (UrlBuilder)         │   │
           │  └──────────────────────────────────────────┘   │
           └──────────────┬──────────────────────────────────┘
                          │
                          ▼
           ┌──────────────────────────────────────────────┐
           │       markmap-html-parser                    │
           │  - Converts HTML to Tree Structure           │
           └──────────────┬───────────────────────────────┘
                          │
                          ▼
           ┌──────────────────────────────────────────────┐
           │         Data Structure (IPureNode)           │
           │  - Hierarchical tree representation          │
           └──────────────┬───────────────────────────────┘
                          │
        ┌─────────────────┴─────────────────┐
        ▼                                   ▼
┌──────────────────┐              ┌──────────────────────┐
│  markmap-render  │              │    markmap-view      │
│  - HTML Template │              │  - D3.js Renderer    │
│  - Asset Bundling│              │  - Interactive UI    │
└──────────────────┘              │  - Zoom/Pan/Click    │
                                  └──────────────────────┘
                                             │
                                             ▼
                                  ┌──────────────────────┐
                                  │  markmap-toolbar     │
                                  │  - Optional Controls │
                                  └──────────────────────┘
                                             │
                                             ▼
                    ┌─────────────────────────────────────────┐
                    │           markmap-common                │
                    │  - Shared Types & Utilities             │
                    │  - Hook System                          │
                    │  - Asset Loaders (CSS/JS)               │
                    │  - URL Builder (npm2url)                │
                    └─────────────────────────────────────────┘
```

## Package Layers

### 1. Core Layer (Foundation)

#### **markmap-common**
- **Purpose**: Shared utilities, types, and helper functions
- **Key Components**:
  - Type definitions (`IPureNode`, `INode`, `IAssets`, `JSItem`, `CSSItem`)
  - `Hook` system for extensibility
  - Asset loaders (`loadJS`, `loadCSS`)
  - `UrlBuilder` for CDN URL resolution (npm2url integration)
  - HTML utilities for asset persistence
- **Dependencies**: npm2url
- **Used By**: All other packages

### 2. Transformation Layer

#### **markmap-lib**
- **Purpose**: Transform Markdown content into data structures
- **Key Components**:
  - `Transformer` class: Main transformation engine
  - `markdown-it` integration: Markdown parsing
  - Plugin system: Extensible transformation hooks
  - Asset management: Track and resolve required CSS/JS
- **Process**:
  1. Initialize markdown-it parser
  2. Apply plugins to enhance parsing
  3. Parse Markdown to HTML
  4. Convert HTML to tree structure
  5. Return data + required assets
- **Built-in Plugins**:
  - `pluginFrontmatter`: Parse YAML frontmatter
  - `pluginKatex`: Math equation rendering
  - `pluginHljs`: Code syntax highlighting (highlight.js)
  - `pluginNpmUrl`: NPM package URL resolution
  - `pluginCheckbox`: Task list support
  - `pluginSourceLines`: Track source line numbers
- **Dependencies**: markdown-it, markmap-common, markmap-html-parser

#### **markmap-html-parser**
- **Purpose**: Convert HTML to hierarchical tree structure
- **Key Components**:
  - `buildTree`: Main parsing function
  - DOM traversal and analysis
  - Node structure generation
- **Process**:
  1. Parse HTML string
  2. Identify heading hierarchy (h1-h6)
  3. Build tree based on heading levels
  4. Extract content for each node
- **Dependencies**: markmap-common

### 3. Rendering Layer

#### **markmap-view**
- **Purpose**: Visualize tree data as interactive mindmap
- **Key Components**:
  - `Markmap` class: Main visualization controller
  - D3.js integration: SVG rendering
  - `flextree` layout: Tree positioning algorithm
  - Event handlers: Zoom, pan, click interactions
  - Animation system: Smooth transitions
- **Features**:
  - Dynamic layout calculation
  - Fold/unfold nodes
  - Auto-fit to viewport
  - Highlight nodes
  - Keyboard/mouse navigation
- **Dependencies**: d3, d3-flextree, markmap-common

#### **markmap-render**
- **Purpose**: Generate HTML templates with embedded mindmaps
- **Key Components**:
  - `fillTemplate`: Inject data and assets into HTML
  - Template management
  - Asset bundling
  - Base JavaScript paths
- **Process**:
  1. Take tree data and assets
  2. Serialize data to JSON
  3. Bundle CSS/JS assets
  4. Generate complete HTML
- **Dependencies**: markmap-common, markmap-view

#### **markmap-toolbar**
- **Purpose**: Optional toolbar for mindmap controls
- **Features**:
  - Zoom in/out buttons
  - Fit to screen
  - Download as SVG/PNG
  - Fullscreen toggle
- **Dependencies**: markmap-view

### 4. Application Layer

#### **markmap-cli**
- **Purpose**: Command-line interface for generating markmaps
- **Key Components**:
  - `createMarkmap`: Generate HTML from Markdown
  - `MarkmapDevServer`: Development server with live reload
  - CLI argument parsing
  - File watching
  - Asset fetching for offline mode
- **Features**:
  - Generate standalone HTML files
  - Watch mode for development
  - Offline asset bundling
  - Auto-open in browser
- **Commands**:
  ```bash
  markmap <input.md>                  # Generate HTML
  markmap <input.md> --watch          # Development mode
  markmap <input.md> --offline        # Inline all assets
  markmap <input.md> -o output.html   # Custom output
  ```
- **Dependencies**: All other markmap packages, Hono (web server), commander (CLI)

#### **markmap-autoloader**
- **Purpose**: Automatically render markmaps in HTML pages
- **Key Components**:
  - Auto-detection of `.markmap` elements
  - Dynamic loading of markmap-lib and markmap-view
  - Plugin system integration
  - Configuration options
- **Usage**:
  ```html
  <script src="markmap-autoloader"></script>
  <div class="markmap">
    # Your Markdown Here
  </div>
  ```
- **Dependencies**: markmap-lib, markmap-view

## Data Flow

### 1. Transformation Flow (Markdown → Tree)

```
Input: Markdown String
        │
        ▼
┌───────────────────────┐
│ Transformer.transform │
└───────────┬───────────┘
            │
            ▼
    ┌──────────────┐
    │ beforeParse  │ ← Plugins can process content
    │    Hook      │
    └──────┬───────┘
           │
           ▼
    ┌─────────────────┐
    │  markdown-it    │ ← Parse Markdown to HTML
    │    .render()    │
    └──────┬──────────┘
           │
           ▼
    ┌──────────────┐
    │  afterParse  │ ← Plugins can process HTML
    │     Hook     │
    └──────┬───────┘
           │
           ▼
    ┌─────────────────┐
    │   buildTree()   │ ← Convert HTML to tree
    │ (html-parser)   │
    └──────┬──────────┘
           │
           ▼
    ┌─────────────────┐
    │   cleanNode()   │ ← Optimize tree structure
    └──────┬──────────┘
           │
           ▼
Output: { root: IPureNode, features: IFeatures, frontmatter, assets }
```

### 2. Rendering Flow (Tree → Visualization)

```
Input: IPureNode + IAssets
        │
        ▼
┌──────────────────────┐
│ Markmap.setData()    │
└──────────┬───────────┘
           │
           ▼
    ┌──────────────────┐
    │ _initializeData  │ ← Add state, assign IDs
    └──────┬───────────┘
           │
           ▼
    ┌──────────────────┐
    │  renderData()    │
    └──────┬───────────┘
           │
           ▼
    ┌──────────────────┐
    │   _relayout()    │ ← Calculate positions
    │   (flextree)     │
    └──────┬───────────┘
           │
           ▼
    ┌──────────────────┐
    │   D3 Update      │ ← Enter/Update/Exit pattern
    │   (Nodes/Links)  │
    └──────┬───────────┘
           │
           ▼
    ┌──────────────────┐
    │   Transitions    │ ← Animate changes
    └──────┬───────────┘
           │
           ▼
    ┌──────────────────┐
    │   autoFit()      │ ← Fit to viewport
    └──────────────────┘
           │
           ▼
Output: Interactive SVG Mindmap
```

### 3. CLI Flow

```
Input: Markdown File Path
        │
        ▼
┌─────────────────┐
│  readFile()     │
└────────┬────────┘
         │
         ▼
┌─────────────────────────┐
│  createMarkmap() or     │
│  develop()              │
└────────┬────────────────┘
         │
    ┌────┴────┐
    │         │
    ▼         ▼
[Watch]   [Generate]
    │         │
    │         ▼
    │    ┌──────────────┐
    │    │ Transformer  │
    │    └──────┬───────┘
    │           │
    │           ▼
    │    ┌──────────────────┐
    │    │  fillTemplate()  │
    │    └──────┬───────────┘
    │           │
    │           ▼
    │    ┌──────────────┐
    │    │ writeFile()  │
    │    └──────────────┘
    │
    ▼
┌────────────────────┐
│ MarkmapDevServer   │
│ - File watcher     │
│ - Web server       │
│ - Live reload      │
└────────────────────┘
```

## Plugin Architecture

### Plugin Interface

```typescript
interface ITransformPlugin {
  name: string;
  config?: {
    versions?: Record<string, string>;
    preloadScripts?: JSItem[];
    resources?: string[];
    styles?: CSSItem[];
    scripts?: JSItem[];
  };
  transform: (hooks: ITransformHooks) => IAssets;
}
```

### Plugin Hooks

1. **parser**: Called once when markdown-it is initialized
   - Modify markdown-it configuration
   - Add custom rules or plugins

2. **beforeParse**: Called before each Markdown parse
   - Preprocess Markdown content
   - Extract frontmatter
   - Set context variables

3. **afterParse**: Called after HTML generation
   - Post-process HTML
   - Detect used features
   - Add metadata

4. **retransform**: Trigger re-transformation
   - Used by autoloader
   - Force refresh when assets load

### Plugin Example Flow

```
Plugin: pluginKatex
    │
    ├─→ parser hook
    │   └─→ Add markdown-it-katex to parser
    │
    ├─→ beforeParse hook
    │   └─→ No action
    │
    ├─→ afterParse hook
    │   └─→ Check if math detected (context.features.katex = true)
    │
    └─→ transform() returns
        └─→ { scripts: [katex.js], styles: [katex.css] }
```

## Asset Management

### URL Resolution Strategy

1. **Default Provider**: Uses CDNs (jsDelivr, unpkg)
2. **Local Provider**: For offline mode, uses local files
3. **Custom Provider**: Can be configured per use case

### Asset Types

- **JSItem**: JavaScript resources
  - `type: 'script'`: External script tags
  - `type: 'iife'`: Inline immediately-invoked functions

- **CSSItem**: Stylesheet resources
  - `type: 'stylesheet'`: External link tags  
  - `type: 'style'`: Inline style tags

### Asset Flow

```
Plugin declares assets
        │
        ▼
UrlBuilder resolves URLs
        │
        ▼
Assets collected by feature detection
        │
        ▼
Renderer persists to HTML
```

## Key Design Patterns

### 1. **Hook System**
- Allows plugins to tap into transformation lifecycle
- Synchronous execution
- Ordered plugin execution

### 2. **Builder Pattern**
- `Transformer` builds transformation pipeline
- `UrlBuilder` constructs resource URLs
- `Markmap` builds visualization

### 3. **Observer Pattern**
- File watching in dev server
- ResizeObserver for layout updates
- Event handlers for user interaction

### 4. **Strategy Pattern**
- URL provider selection
- Layout algorithms
- Asset resolution

### 5. **Factory Pattern**
- `Markmap.create()` static factory
- Plugin instantiation

## Performance Considerations

1. **Lazy Loading**: Assets loaded only when features are used
2. **Tree Optimization**: `cleanNode()` removes redundant structure
3. **Virtual DOM**: D3's enter/update/exit pattern
4. **Debouncing**: Resize and watch events debounced
5. **Memoization**: Color assignments cached

## Security Considerations

1. **Content Sanitization**: HTML content is controlled through markdown-it
2. **CSP Compatibility**: Supports Content Security Policy
3. **XSS Prevention**: No direct HTML injection from user input
4. **Safe URL Building**: Validates package names and versions

## Next Steps

- See [Packages](./03-packages.md) for detailed package documentation
- See [Workflow](./04-workflow.md) for transformation process details
- See [Plugins](./06-plugins.md) for plugin development guide
