# Storylog

A lightweight, zero-config alternative to Storybook for previewing and exploring Svelte components. Built with Svelte 5 and designed for simplicity and speed.

![Storylog](https://img.shields.io/badge/Svelte-5.0+-ff3e00?logo=svelte)
![License](https://img.shields.io/badge/license-MIT-blue)

## ✨ Features

- 🎨 **Dark Mode Toggle** - Switch between light and dark themes with system preference detection
- 📱 **Responsive Viewport Controls** - Test components on small mobile, large mobile, tablet, and full view
- 🔍 **Component Search** - Quickly find stories using keyboard shortcuts
- ⌨️ **Keyboard Shortcuts** - Navigate efficiently with keyboard commands
- 🎯 **Minimal Interface** - No controls panel, focused purely on component preview
- 📂 **File-based Stories** - Organize stories in `.story.svelte` or `.stories.svelte` files
- 🌲 **Organized Navigation** - Expandable sections with icons for easy browsing
- ⚡ **Fast & Lightweight** - No external Storybook dependencies, self-contained
- 🎨 **Beautiful Icons** - Clean SVG icons for all UI elements
- 📏 **Resizable Height** - Interactive height adjustment for viewport containers

## 🚀 Quick Start

### Installation

```bash
npm install storylog
# or
yarn add storylog
# or
pnpm add storylog
# or
bun add storylog
```

### Basic Setup

1. Create a main page (e.g., `src/routes/+page.svelte` in SvelteKit):

```svelte
<script lang="ts">
	import { Storylog } from '$lib/index.js';

	// Automatically discover all .story.svelte and .stories.svelte files
	const storyFiles = import.meta.glob('./**/*.story.svelte');
	const storiesFiles = import.meta.glob('./**/*.stories.svelte');
	const stories = { ...storyFiles, ...storiesFiles };
</script>

<Storylog title="Storylog" {stories} />
```

2. Create a story file (e.g., `src/routes/Buttons/Button.stories.svelte`):

```svelte
<script>
	import { Story } from '$lib/index.js';
	
	let { section, fileName } = $props();
</script>

<Story title="Primary">
	<button class="btn btn-primary">Primary Button</button>
</Story>

<Story title="Secondary">
	<button class="btn btn-secondary">Secondary Button</button>
</Story>

<Story title="Disabled">
	<button class="btn btn-primary" disabled>Disabled</button>
</Story>
```

That's it! Storylog will automatically discover and display your stories.

## 📚 Usage

### Story Component

The `Story` component is used to define individual component variations within a story file.

```svelte
<script>
	import { Story } from '$lib/index.js';
	
	let { section, fileName } = $props();
</script>

<Story title="Story Title">
	<!-- Your component or content here -->
	<MyComponent prop="value" />
</Story>
```

**Props:**
- `title` (string, required) - The name of the story/variant

### Storylog Component

The main component that renders the Storylog interface.

```svelte
<script lang="ts">
	import { Storylog } from '$lib/index.js';

	const stories = import.meta.glob('./**/*.stories.svelte');
</script>

<Storylog title="My Component Library" {stories} />
```

**Props:**
- `title` (string, optional) - Title displayed in the sidebar header (default: "Storylog")
- `stories` (Record<string, () => Promise<any>>, required) - Object of story file paths and their loaders (from `import.meta.glob`)

### File Organization

Stories are automatically organized by their file path:

```
src/routes/
  ├── Buttons/
  │   └── Button.stories.svelte  → Section: "Buttons"
  ├── Cards/
  │   └── Card.stories.svelte    → Section: "Cards"
  └── Forms/
      └── Input.stories.svelte   → Section: "Forms"
```

The section name is derived from the parent directory name. For files directly in `routes/`, they're grouped under "Components".

## ⌨️ Keyboard Shortcuts

- `K` or `Cmd/Ctrl + K` - Focus search input
- `D` or `Cmd/Ctrl + D` - Toggle dark/light theme
- `1` - Switch to small mobile viewport (320px)
- `2` - Switch to large mobile viewport (375px)
- `3` - Switch to tablet viewport (834px)
- `4` - Switch to full viewport (no constraints)
- `↑` / `↓` - Navigate between stories
- `Esc` - Clear search or close sidebar (mobile)

## 🎨 Viewport Controls

Storylog provides four viewport modes:

1. **Small Mobile** (320px) - Test on small mobile devices
2. **Large Mobile** (375px) - Test on large mobile devices
3. **Tablet** (834px) - Test on tablet devices
4. **Full View** - No width/height constraints, fills available space

For constrained viewports (1-3), you can:
- Adjust height by dragging the resize handle at the bottom
- Default height is 500px
- Height changes are preserved when switching between constrained viewports

## Features

### Dark Mode

Dark mode is automatically initialized from:
1. LocalStorage preference (if previously set)
2. System preference (if no saved preference)

The theme persists across page reloads and can be toggled via:
- Theme toggle button in the header
- Keyboard shortcut (`D` or `Cmd/Ctrl + D`)

### Component Search

- Real-time search across story titles, sections, and IDs
- Auto-expands sections containing matching stories
- Clear search with the X button or `Esc` key
- Focus search with `K` or `Cmd/Ctrl + K`

### Navigation

- **Expandable Sections** - Click section headers to expand/collapse
- **Icons** - Grid icons for sections, bookmark icons for stories
- **Active Highlighting** - Currently selected story is highlighted
- **Auto-expand** - Sections automatically expand when:
  - A story within them is selected
  - Search results include stories from that section
  - The first story is loaded

### Responsive Design

- **Desktop** - Fixed sidebar (280px) with collapsible toggle
- **Mobile/Tablet** - Hamburger menu that slides sidebar in/out
- **Sidebar Toggle** - Click the chevron icon or hamburger menu to toggle

## Complete Example

### Button.stories.svelte

```svelte
<script>
	import { Story } from '$lib/index.js';
	
	let { section, fileName } = $props();
	
	// Interactive state example
	let count = $state(0);
</script>

<Story title="Primary">
	<button
		class="btn btn-primary"
		onclick={() => count++}
	>
		Clicked {count} times
	</button>
</Story>

<Story title="Secondary">
	<button class="btn btn-secondary">Secondary Button</button>
</Story>

<Story title="Disabled">
	<button class="btn btn-primary" disabled>Disabled</button>
</Story>

<Story title="Button Sizes">
	<div style="display: flex; gap: 1rem; align-items: center;">
		<button class="btn btn-primary btn-sm">Small</button>
		<button class="btn btn-primary">Medium</button>
		<button class="btn btn-primary btn-lg">Large</button>
	</div>
</Story>

<style>
	.btn {
		padding: 0.5rem 1rem;
		border: none;
		border-radius: 6px;
		font-size: 0.95rem;
		font-weight: 500;
		cursor: pointer;
		transition: all 0.2s;
	}
	
	.btn-primary {
		background: #3b82f6;
		color: white;
	}
	
	.btn-primary:hover {
		background: #2563eb;
	}
	
	/* ... more styles ... */
</style>
```

### Card.stories.svelte

```svelte
<script>
	import { Story } from '$lib/index.js';
	
	let { section, fileName } = $props();
</script>

<Story title="Basic Card">
	<div class="card">
		<h3>Card Title</h3>
		<p>Card content goes here</p>
	</div>
</Story>

<Story title="Card with Shadow">
	<div class="card card-shadow">
		<h3>Elevated Card</h3>
		<p>This card has a shadow effect</p>
	</div>
</Story>

<style>
	.card {
		padding: 1.5rem;
		background: white;
		border-radius: 8px;
		border: 1px solid #e5e7eb;
	}
	
	.card-shadow {
		box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
	}
</style>
```

## 🎨 Customization

### Styling

Storylog uses CSS variables for theming. You can customize the appearance by overriding these variables:

```css
:global(html[data-theme='light']) {
	--storylog-bg: #ffffff;
	--storylog-sidebar-bg: #f6f9fc;
	--storylog-text: #1a1a1a;
	--storylog-active: #dbeafe;
	/* ... more variables ... */
}

:global(html[data-theme='dark']) {
	--storylog-bg: #1a1a1a;
	--storylog-sidebar-bg: #111827;
	--storylog-text: #f9fafb;
	--storylog-active: #1e3a8a;
	/* ... more variables ... */
}
```

### Custom Viewport Sizes

Viewport sizes are defined in the Storylog component. To customize, you'll need to modify the source or fork the project.

## 🏗️ Architecture

### Components

- **Storylog.svelte** - Main container component with sidebar and canvas
- **Story.svelte** - Individual story/variant component
- **Icons/** - SVG icon components for UI elements

### File Discovery

Storylog uses Vite's `import.meta.glob` to discover story files:

```typescript
const stories = import.meta.glob('./**/*.stories.svelte');
```

This creates an object where:
- Keys are file paths (e.g., `./Buttons/Button.stories.svelte`)
- Values are async functions that load the module

### Story Registration

Stories register themselves via Svelte's context API:

1. Story files are loaded and rendered
2. Each `Story` component registers itself with the catalog context
3. The catalog builds navigation from registered stories
4. Stories are grouped by section (derived from file path)

## 🧪 Development

```bash
# Install dependencies
npm install

# Start dev server
npm run dev

# Build for production
npm run build

# Type checking
npm run check

# Linting
npm run lint
```

## 📦 Requirements

- **Svelte 5.0+** - Built with Svelte 5 runes (`$state`, `$derived`, `$effect`)
- **Vite** - Uses Vite's `import.meta.glob` for file discovery
- **Modern Browser** - Requires modern browser features (CSS Grid, Flexbox, etc.)

## 🤝 Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

## 📄 License

MIT

## 🙏 Acknowledgments

Inspired by Storybook, but designed to be simpler and more lightweight for Svelte projects.

## 🆘 Troubleshooting

### Stories not appearing

1. Check that your story files end with `.story.svelte` or `.stories.svelte`
2. Verify the glob pattern matches your file structure
3. Ensure stories export a default component
4. Check browser console for errors

### Viewport not working

1. Ensure you're using a modern browser
2. Check that CSS is loading properly
3. Verify viewport container has proper styles

### Search not working

1. Check that stories have registered properly
2. Verify search input is receiving keyboard events
3. Check browser console for JavaScript errors

## 📚 Additional Resources

- [Svelte Documentation](https://svelte.dev)
- [Vite Documentation](https://vitejs.dev)
- [SvelteKit Documentation](https://kit.svelte.dev)

---

Made with ❤️ for the Svelte community
