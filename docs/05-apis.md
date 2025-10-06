# API Reference

Complete API documentation for all Markmap packages.

## Table of Contents

- [markmap-lib API](#markmap-lib-api)
- [markmap-view API](#markmap-view-api)
- [markmap-render API](#markmap-render-api)
- [markmap-cli API](#markmap-cli-api)
- [markmap-common API](#markmap-common-api)
- [markmap-html-parser API](#markmap-html-parser-api)
- [markmap-autoloader API](#markmap-autoloader-api)
- [markmap-toolbar API](#markmap-toolbar-api)

---

## markmap-lib API

### Transformer

Main class for transforming Markdown to tree structures.

#### Constructor

```typescript
new Transformer(plugins?: Array<ITransformPlugin | (() => ITransformPlugin)>)
```

**Parameters:**
- `plugins` - Array of plugin instances or plugin factory functions. Defaults to `builtInPlugins`.

**Example:**
```typescript
import { Transformer, pluginKatex } from 'markmap-lib';

const transformer = new Transformer([pluginKatex]);
```

#### Properties

| Property | Type | Description |
|----------|------|-------------|
| `hooks` | `ITransformHooks` | Transform lifecycle hooks |
| `md` | `MarkdownIt` | markdown-it instance |
| `assetsMap` | `Record<string, IAssets>` | Plugin assets mapping |
| `urlBuilder` | `UrlBuilder` | URL resolution utility |
| `plugins` | `ITransformPlugin[]` | Active plugins |

#### Methods

##### transform()

```typescript
transform(
  content: string,
  fallbackParserOptions?: Partial<IHtmlParserOptions>
): ITransformResult
```

Transforms Markdown content into a tree structure.

**Parameters:**
- `content` - Markdown string to transform
- `fallbackParserOptions` - Optional HTML parser configuration

**Returns:** `ITransformResult`
```typescript
{
  root: IPureNode;              // Tree structure
  features: IFeatures;          // Detected features
  frontmatter?: {               // Extracted metadata
    title?: string;
    markmap?: Partial<IMarkmapJSONOptions>;
  };
  content: string;              // Original content
  frontmatterInfo?: {           // Frontmatter position info
    lines: number;
    offset: number;
  };
}
```

**Example:**
```typescript
const { root, features, frontmatter } = transformer.transform(`
# My Project
## Phase 1
- Task 1
- Task 2
`);
```

##### getAssets()

```typescript
getAssets(keys?: string[]): IAssets
```

Get assets from plugins.

**Parameters:**
- `keys` - Optional array of plugin names to filter. If omitted, returns all plugin assets.

**Returns:** `IAssets`

**Example:**
```typescript
// Get all assets
const allAssets = transformer.getAssets();

// Get specific plugins
const mathAssets = transformer.getAssets(['katex']);
```

##### getUsedAssets()

```typescript
getUsedAssets(features: IFeatures): IAssets
```

Get assets for detected features only.

**Parameters:**
- `features` - Features object from transform result

**Returns:** `IAssets`

**Example:**
```typescript
const { features } = transformer.transform(markdown);
const assets = transformer.getUsedAssets(features);
```

##### resolveJS()

```typescript
resolveJS(item: JSItem): JSItem
```

Resolve JavaScript item URLs.

**Parameters:**
- `item` - JavaScript item to resolve

**Returns:** Resolved `JSItem`

##### resolveCSS()

```typescript
resolveCSS(item: CSSItem): CSSItem
```

Resolve CSS item URLs.

**Parameters:**
- `item` - CSS item to resolve

**Returns:** Resolved `CSSItem`

### Interfaces

#### ITransformHooks

```typescript
interface ITransformHooks {
  transformer: ITransformer;
  parser: Hook<[MarkdownIt]>;
  beforeParse: Hook<[MarkdownIt, ITransformContext]>;
  afterParse: Hook<[MarkdownIt, ITransformContext]>;
  retransform: Hook<[]>;
}
```

#### ITransformPlugin

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

### Built-in Plugins

#### pluginFrontmatter

Parses YAML frontmatter.

```typescript
import { pluginFrontmatter } from 'markmap-lib';
```

**Features:**
- Extracts `---` delimited YAML
- Sets `context.frontmatter`
- Supports `title` and `markmap` options

#### pluginKatex

Math equation rendering.

```typescript
import { pluginKatex } from 'markmap-lib';
```

**Syntax:**
- Inline: `$E=mc^2$`
- Display: `$$\int_0^1 x^2 dx$$`

**Assets:**
- `katex.min.css`
- `katex.min.js`

#### pluginHljs

Code syntax highlighting using Highlight.js.

```typescript
import { pluginHljs } from 'markmap-lib';
```

**Assets:**
- `highlight.js/styles/[theme].css`
- `highlight.js/lib/core.js`
- Language-specific modules

#### pluginCheckbox

Task list checkboxes.

```typescript
import { pluginCheckbox } from 'markmap-lib';
```

**Syntax:**
- `- [ ]` Unchecked
- `- [x]` Checked

#### pluginNpmUrl

NPM package URL resolution.

```typescript
import { pluginNpmUrl } from 'markmap-lib';
```

**Transforms:** `package@version/path` → CDN URL

#### pluginSourceLines

Track source line numbers.

```typescript
import { pluginSourceLines } from 'markmap-lib';
```

**Adds:** Line number metadata to nodes

---

## markmap-view API

### Markmap

Main visualization class.

#### Constructor

```typescript
new Markmap(
  svg: string | SVGElement | ID3SVGElement,
  opts?: Partial<IMarkmapOptions>
)
```

**Parameters:**
- `svg` - CSS selector, SVG element, or D3 selection
- `opts` - Optional configuration

**Example:**
```typescript
import { Markmap } from 'markmap-view';

const mm = new Markmap('#markmap', {
  duration: 500,
  maxWidth: 300
});
```

#### Static Methods

##### Markmap.create()

```typescript
static create(
  svg: string | SVGElement | ID3SVGElement,
  opts?: Partial<IMarkmapOptions>,
  data?: IPureNode | null
): Markmap
```

Factory method for creating and initializing a Markmap.

**Example:**
```typescript
const mm = Markmap.create('#markmap', options, treeData);
// Automatically fits on first render
```

#### Properties

| Property | Type | Description |
|----------|------|-------------|
| `options` | `IMarkmapOptions` | Current options |
| `state` | `IMarkmapState` | Internal state |
| `svg` | `ID3SVGElement` | D3 SVG selection |
| `styleNode` | `D3Selection<HTMLStyleElement>` | Style element |
| `g` | `D3Selection<SVGGElement>` | Main group element |
| `zoom` | `D3.ZoomBehavior` | Zoom behavior |

#### Methods

##### setData()

```typescript
async setData(
  data?: IPureNode | null,
  opts?: Partial<IMarkmapOptions>
): Promise<void>
```

Set or update the tree data.

**Parameters:**
- `data` - Tree data to render
- `opts` - Optional options to merge

**Example:**
```typescript
await mm.setData(newTreeData, { maxWidth: 400 });
```

##### setOptions()

```typescript
setOptions(opts?: Partial<IMarkmapOptions>): void
```

Update options without re-rendering.

**Parameters:**
- `opts` - Options to merge

**Example:**
```typescript
mm.setOptions({ duration: 1000 });
```

##### toggleNode()

```typescript
async toggleNode(node: INode, recursive?: boolean): Promise<void>
```

Expand or collapse a node.

**Parameters:**
- `node` - Node to toggle
- `recursive` - If true, toggle all descendants

**Example:**
```typescript
await mm.toggleNode(someNode, true);
```

##### fit()

```typescript
async fit(maxScale?: number): Promise<void>
```

Fit content to viewport.

**Parameters:**
- `maxScale` - Maximum zoom scale (default: from options)

**Example:**
```typescript
await mm.fit(2);
```

##### ensureVisible()

```typescript
async ensureVisible(
  node: INode,
  padding?: Partial<IPadding>
): Promise<void>
```

Pan to make a node visible.

**Parameters:**
- `node` - Node to show
- `padding` - Viewport padding

**Example:**
```typescript
await mm.ensureVisible(node, { left: 50, top: 50 });
```

##### centerNode()

```typescript
async centerNode(
  node: INode,
  padding?: Partial<IPadding>
): Promise<void>
```

Center a node in the viewport.

**Example:**
```typescript
await mm.centerNode(node);
```

##### rescale()

```typescript
async rescale(scale: number): Promise<void>
```

Change zoom level.

**Parameters:**
- `scale` - New zoom scale

**Example:**
```typescript
await mm.rescale(1.5);
```

##### setHighlight()

```typescript
async setHighlight(node?: INode | null): Promise<void>
```

Highlight a specific node.

**Parameters:**
- `node` - Node to highlight, or null to clear

**Example:**
```typescript
await mm.setHighlight(someNode);
```

##### findElement()

```typescript
findElement(node: INode): { data: INode; g: SVGGElement } | undefined
```

Find DOM element for a node.

**Parameters:**
- `node` - Node to find

**Returns:** Element data or undefined

##### destroy()

```typescript
destroy(): void
```

Clean up and remove the markmap.

**Example:**
```typescript
mm.destroy();
```

### Interfaces

#### IMarkmapOptions

```typescript
interface IMarkmapOptions {
  // Animation
  duration: number;                    // Default: 500

  // Colors
  color: (node: INode) => string;      // Color function
  
  // Layout
  paddingX: number;                    // Default: 8
  spacingHorizontal: number;           // Default: 80
  spacingVertical: number;             // Default: 5
  maxWidth: number;                    // Default: 0 (unlimited)
  initialExpandLevel: number;          // Default: -1 (all)
  
  // Interaction
  zoom: boolean;                       // Default: true
  pan: boolean;                        // Default: true
  scrollForPan: boolean;               // Default: false
  toggleRecursively: boolean;          // Default: false
  
  // Styling
  style: ((id: string) => string) | null;
  embedGlobalCSS: boolean;             // Default: true
  
  // Auto-fit
  autoFit: boolean;                    // Default: true
  fitRatio: number;                    // Default: 0.95
  maxInitialScale: number;             // Default: 2
  
  // Line width
  lineWidth: (node: INode) => number;  // Default: () => 2
  
  // ID
  id?: string;
}
```

#### IMarkmapState

```typescript
interface IMarkmapState {
  id: string;
  data?: INode;
  highlight?: INode;
  rect: {
    x1: number;
    y1: number;
    x2: number;
    y2: number;
  };
}
```

#### IPadding

```typescript
interface IPadding {
  left: number;
  right: number;
  top: number;
  bottom: number;
}
```

### Global Functions

#### refreshHook

```typescript
import { refreshHook } from 'markmap-view';

// Refresh all markmaps
refreshHook.call();

// Listen for refresh
const dispose = refreshHook.tap(() => {
  console.log('Refreshing...');
});
```

---

## markmap-render API

### fillTemplate()

```typescript
function fillTemplate(
  root: IPureNode | null,
  assets: IAssets,
  extra?: {
    baseJs?: JSItem[];
    jsonOptions?: Partial<IMarkmapJSONOptions>;
    getOptions?: (jsonOptions: Partial<IMarkmapJSONOptions>) => Partial<IMarkmapOptions>;
    urlBuilder?: UrlBuilder;
  }
): string
```

Generate standalone HTML file.

**Parameters:**
- `root` - Tree data
- `assets` - Required CSS/JS assets
- `extra` - Additional configuration

**Returns:** Complete HTML string

**Example:**
```typescript
import { fillTemplate } from 'markmap-render';

const html = fillTemplate(treeData, assets, {
  jsonOptions: {
    maxWidth: 300,
    initialExpandLevel: 2
  }
});

await fs.writeFile('output.html', html);
```

### Constants

#### baseJsPaths

```typescript
const baseJsPaths: string[]
```

Default JavaScript dependencies.

**Value:**
```typescript
[
  'd3@7/dist/d3.min.js',
  'markmap-view@VERSION/dist/browser/index.js'
]
```

#### template

```typescript
const template: string
```

HTML template string with placeholders.

---

## markmap-cli API

### createMarkmap()

```typescript
async function createMarkmap(options: {
  content?: string;
  output?: string;
  open?: boolean;
  toolbar?: boolean;
  offline?: boolean;
}): Promise<void>
```

Generate HTML file from Markdown.

**Parameters:**
- `content` - Markdown content
- `output` - Output file path (default: 'markmap.html')
- `open` - Open in browser (default: true)
- `toolbar` - Include toolbar (default: true)
- `offline` - Inline all assets (default: false)

**Example:**
```typescript
import { createMarkmap } from 'markmap-cli';

await createMarkmap({
  content: '# My Mindmap\n## Section 1',
  output: 'map.html',
  offline: true
});
```

### develop()

```typescript
async function develop(options: {
  toolbar?: boolean;
  offline?: boolean;
  port?: number;
}): Promise<MarkmapDevServer>
```

Start development server.

**Parameters:**
- `toolbar` - Include toolbar (default: true)
- `offline` - Use offline assets (default: false)
- `port` - Server port (default: auto)

**Returns:** `MarkmapDevServer` instance

**Example:**
```typescript
import { develop } from 'markmap-cli';

const server = await develop({ port: 3000 });
const provider = server.addProvider({ 
  filePath: 'notes.md' 
});

console.log(`http://localhost:3000/?key=${provider.key}`);
```

### MarkmapDevServer

```typescript
class MarkmapDevServer {
  options: IDevelopOptions;
  providers: Record<string, IContentProvider>;
  serverInfo: { server: ServerType; address: AddressInfo } | null;
  
  async setup(): Promise<void>;
  async shutdown(): Promise<void>;
  async destroy(): Promise<void>;
  
  addProvider(options?: {
    key?: string;
    filePath?: string;
  }): IContentProvider;
  
  delProvider(key: string): void;
}
```

---

## markmap-common API

### Types

#### IPureNode

```typescript
interface IPureNode {
  content: string;
  payload?: {
    fold?: number;  // 0: open, 1: folded, 2: recursive fold
    [key: string]: unknown;
  };
  children: IPureNode[];
}
```

#### INode

```typescript
interface INode extends IPureNode {
  state: INodeState;
  children: INode[];
}
```

#### INodeState

```typescript
interface INodeState {
  id: number;
  path: string;
  key: string;
  depth: number;
  size: [number, number];
  rect: {
    x: number;
    y: number;
    width: number;
    height: number;
  };
}
```

#### IAssets

```typescript
interface IAssets {
  styles?: CSSItem[];
  scripts?: JSItem[];
}
```

#### JSItem

```typescript
type JSItem = JSScriptItem | JSIIFEItem;

type JSScriptItem = {
  type: 'script';
  loaded?: Promise<void>;
  data: {
    src?: string;
    textContent?: string;
    async?: boolean;
    defer?: boolean;
  };
};

type JSIIFEItem = {
  type: 'iife';
  loaded?: Promise<void>;
  data: {
    fn: (...args: unknown[]) => void;
    getParams?: (context: unknown) => void | unknown[];
  };
};
```

#### CSSItem

```typescript
type CSSItem = CSSStyleItem | CSSStylesheetItem;

type CSSStyleItem = {
  type: 'style';
  loaded?: Promise<void>;
  data: string;
};

type CSSStylesheetItem = {
  type: 'stylesheet';
  loaded?: Promise<void>;
  data: {
    href: string;
  };
};
```

### Classes

#### Hook

```typescript
class Hook<T extends unknown[]> {
  tap(fn: (...args: T) => void): () => void;
  call(...args: T): void;
}
```

**Example:**
```typescript
import { Hook } from 'markmap-common';

const hook = new Hook<[string, number]>();

const dispose = hook.tap((str, num) => {
  console.log(str, num);
});

hook.call('hello', 42);  // Logs: hello 42

dispose();  // Unregister
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

**Example:**
```typescript
import { UrlBuilder } from 'markmap-common';

const builder = new UrlBuilder();

// Add custom provider
builder.setProvider('custom', (path) => {
  return `https://my-cdn.com/${path}`;
});

builder.provider = 'custom';
const url = builder.getFullUrl('d3@7/dist/d3.min.js');
```

### Functions

#### loadJS()

```typescript
function loadJS(item: JSItem, context?: unknown): Promise<void>
```

Dynamically load JavaScript.

**Example:**
```typescript
import { loadJS, buildJSItem } from 'markmap-common';

await loadJS(buildJSItem('https://cdn.example.com/lib.js'));
```

#### loadCSS()

```typescript
function loadCSS(item: CSSItem): Promise<void>
```

Dynamically load CSS.

**Example:**
```typescript
import { loadCSS, buildCSSItem } from 'markmap-common';

await loadCSS(buildCSSItem('https://cdn.example.com/style.css'));
```

#### persistJS()

```typescript
function persistJS(items: JSItem[], context?: unknown): string[]
```

Convert JS items to HTML strings.

#### persistCSS()

```typescript
function persistCSS(items: CSSItem[]): string[]
```

Convert CSS items to HTML strings.

#### walkTree()

```typescript
function walkTree<T extends { children?: T[] }>(
  tree: T,
  callback: (
    item: T,
    next: () => void,
    parent?: T
  ) => void
): void
```

Traverse tree structure.

**Example:**
```typescript
import { walkTree } from 'markmap-common';

walkTree(treeData, (node, next, parent) => {
  console.log(node.content);
  next();  // Continue to children
});
```

#### Other Utilities

```typescript
function defer<T>(): IDeferred<T>
function getId(): string
function addClass(className: string, ...rest: (string | undefined)[]): string
function debounce<T extends (...args: any[]) => void>(fn: T, delay: number): T
function extractAssets(assets: IAssets): string[]
```

---

## markmap-html-parser API

### buildTree()

```typescript
function buildTree(
  html: string,
  options?: Partial<IHtmlParserOptions>
): IPureNode
```

Convert HTML to tree structure.

**Parameters:**
- `html` - HTML string
- `options` - Parser options

**Options:**
```typescript
interface IHtmlParserOptions {
  selector?: string;
  getAttrs?: (el: Element) => Record<string, unknown>;
}
```

**Example:**
```typescript
import { buildTree } from 'markmap-html-parser';

const html = '<h1>Root</h1><h2>Child</h2>';
const tree = buildTree(html);
```

---

## markmap-autoloader API

### Configuration

```typescript
window.markmap = {
  autoLoader: {
    manual: boolean;
    toolbar: boolean;
    provider: string | ((path: string) => string);
    transformPlugins: ITransformPlugin[];
    onReady: () => void;
  }
};
```

### Functions

#### renderAll()

```typescript
function renderAll(): void
```

Render all `.markmap` elements.

#### render()

```typescript
function render(el: HTMLElement): void
```

Render specific element.

**Example:**
```typescript
import { render } from 'markmap-autoloader';

const el = document.querySelector('.my-markmap');
render(el);
```

---

## markmap-toolbar API

### Toolbar Class

```typescript
class Toolbar {
  constructor();
  attach(mm: Markmap): void;
  detach(): void;
  getAssets(): IAssets;
}
```

**Example:**
```typescript
import { Toolbar } from 'markmap-toolbar';

const toolbar = new Toolbar();
toolbar.attach(markmapInstance);
```

### Assets

```typescript
import { toolbarAssets } from 'markmap-toolbar';
```

---

## Type Definitions

### Common Types

```typescript
// Node fold states
type FoldState = 0 | 1 | 2;  // 0: open, 1: folded, 2: recursive fold

// Feature flags
interface IFeatures {
  [pluginName: string]: boolean;
}

// Deferred promise
interface IDeferred<T> {
  promise: Promise<T>;
  resolve: (value: T) => void;
  reject: (error?: unknown) => void;
}
```

## Usage Patterns

### Pattern 1: Transform and Render

```typescript
import { Transformer } from 'markmap-lib';
import { Markmap } from 'markmap-view';

const transformer = new Transformer();
const { root } = transformer.transform(markdown);
const mm = Markmap.create('#svg', {}, root);
```

### Pattern 2: Load Assets

```typescript
import { Transformer } from 'markmap-lib';
import { loadJS, loadCSS } from 'markmap-common';

const transformer = new Transformer();
const { root, features } = transformer.transform(markdown);
const assets = transformer.getUsedAssets(features);

// Load assets
await Promise.all([
  ...assets.scripts?.map(item => loadJS(item)) || [],
  ...assets.styles?.map(item => loadCSS(item)) || []
]);

// Now render
const mm = Markmap.create('#svg', {}, root);
```

### Pattern 3: Custom Plugin

```typescript
import { Transformer } from 'markmap-lib';

const myPlugin = {
  name: 'myPlugin',
  transform: (hooks) => {
    hooks.beforeParse.tap((md, context) => {
      // Preprocess
    });
    hooks.afterParse.tap((md, context) => {
      // Post-process
      context.features.myPlugin = true;
    });
    return {
      scripts: [{ type: 'script', data: { src: 'my-lib.js' } }],
      styles: []
    };
  }
};

const transformer = new Transformer([myPlugin]);
```

## Next Steps

- See [Plugins](./06-plugins.md) for plugin development
- See [Getting Started](./07-getting-started.md) for quick start
- See [Examples](./08-examples.md) for practical code samples
