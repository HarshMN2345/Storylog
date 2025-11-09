# Usage Guide

This guide shows you how to use Svelte Catalog to showcase your own components.

## Quick Start

### 1. Install the package

```bash
bun install storylog
```

### 2. Create a catalog page

Create a new file (e.g., `src/routes/catalog/+page.svelte`):

```svelte
<script>
	import { Storylog, Story } from 'storylog';
	import MyButton from '$lib/components/MyButton.svelte';
	import MyCard from '$lib/components/MyCard.svelte';
</script>

<Catalog title="My Design System">
	<Section title="Buttons">
		<Variant title="Primary">
			<MyButton variant="primary">Click me</MyButton>
		</Variant>
	</Section>
</Catalog>
```

### 3. Run your dev server

```bash
npm run dev
```

Navigate to `/catalog` to see your component showcase!

## Real-World Examples

### Example 1: Button Library

```svelte
<script>
	import { Storylog, Story } from 'storylog';
	import Button from '$lib/Button.svelte';
</script>

<Catalog title="Button Components">
	<Section title="Variants" description="Different button styles">
		<Variant title="Primary" description="Main action button">
			<Button variant="primary">Primary Button</Button>
		</Variant>

		<Variant title="Secondary" description="Secondary action">
			<Button variant="secondary">Secondary Button</Button>
		</Variant>

		<Variant title="Outline" description="Outlined style">
			<Button variant="outline">Outline Button</Button>
		</Variant>
	</Section>

	<Section title="Sizes" description="Available button sizes">
		<Variant title="All Sizes">
			<div style="display: flex; gap: 1rem; align-items: center;">
				<Button size="sm">Small</Button>
				<Button size="md">Medium</Button>
				<Button size="lg">Large</Button>
			</div>
		</Variant>
	</Section>

	<Section title="States" description="Button states">
		<Variant title="Disabled">
			<Button disabled>Disabled Button</Button>
		</Variant>

		<Variant title="Loading">
			<Button loading>Loading...</Button>
		</Variant>
	</Section>
</Catalog>
```

### Example 2: Form Components

```svelte
<script>
	import { Storylog, Story } from 'storylog';
	import TextInput from '$lib/forms/TextInput.svelte';
	import Select from '$lib/forms/Select.svelte';
	import Checkbox from '$lib/forms/Checkbox.svelte';

	let textValue = $state('');
	let selectValue = $state('option1');
	let checked = $state(false);
</script>

<Catalog title="Form Components">
	<Section title="Text Inputs">
		<Variant title="Default Input">
			<TextInput bind:value={textValue} placeholder="Enter text..." />
		</Variant>

		<Variant title="With Label">
			<TextInput label="Email Address" type="email" placeholder="you@example.com" />
		</Variant>

		<Variant title="Error State">
			<TextInput label="Username" error="This username is already taken" />
		</Variant>
	</Section>

	<Section title="Select Dropdowns">
		<Variant title="Basic Select">
			<Select
				bind:value={selectValue}
				options={[
					{ value: 'option1', label: 'Option 1' },
					{ value: 'option2', label: 'Option 2' },
					{ value: 'option3', label: 'Option 3' }
				]}
			/>
		</Variant>
	</Section>

	<Section title="Checkboxes">
		<Variant title="Checkbox">
			<Checkbox bind:checked>I agree to the terms and conditions</Checkbox>
		</Variant>
	</Section>
</Catalog>
```

### Example 3: Card Variations

```svelte
<script>
	import { Storylog, Story } from 'storylog';
	import Card from '$lib/Card.svelte';
	import Avatar from '$lib/Avatar.svelte';
</script>

<Catalog title="Card Components">
	<Section title="Basic Cards">
		<Variant title="Simple Card">
			<Card>
				<h3>Card Title</h3>
				<p>This is a simple card with some content.</p>
			</Card>
		</Variant>

		<Variant title="Card with Footer">
			<Card>
				<h3>Profile</h3>
				<p>John Doe</p>
				<svelte:fragment slot="footer">
					<button>View Profile</button>
				</svelte:fragment>
			</Card>
		</Variant>
	</Section>

	<Section title="Interactive Cards">
		<Variant title="Clickable Card">
			<Card clickable onclick={() => alert('Card clicked!')}>
				<h3>Click Me</h3>
				<p>This card is interactive</p>
			</Card>
		</Variant>
	</Section>

	<Section title="User Cards">
		<Variant title="User Profile Card">
			<Card>
				<div style="display: flex; align-items: center; gap: 1rem;">
					<Avatar src="/avatar.jpg" name="Jane Doe" />
					<div>
						<h4 style="margin: 0;">Jane Doe</h4>
						<p style="margin: 0.25rem 0 0; color: #666;">Product Designer</p>
					</div>
				</div>
			</Card>
		</Variant>
	</Section>
</Catalog>
```

## Tips & Best Practices

### 1. Grouping Related Variants

Keep related variations together in a single Section:

```svelte
<Section title="Alerts">
	<Variant title="Success">
		<Alert type="success">Operation successful!</Alert>
	</Variant>

	<Variant title="Error">
		<Alert type="error">Something went wrong</Alert>
	</Variant>

	<Variant title="Warning">
		<Alert type="warning">Please review your input</Alert>
	</Variant>
</Section>
```

### 2. Showing Multiple States Side-by-Side

Use flexbox to show variations horizontally:

```svelte
<Variant title="All Badge Colors">
	<div style="display: flex; gap: 0.5rem; flex-wrap: wrap;">
		<Badge color="blue">Blue</Badge>
		<Badge color="green">Green</Badge>
		<Badge color="red">Red</Badge>
		<Badge color="yellow">Yellow</Badge>
	</div>
</Variant>
```

### 3. Interactive Components

Components can be fully interactive:

```svelte
<script>
	let count = $state(0);
	let modalOpen = $state(false);
</script>

<Section title="Interactive Examples">
	<Variant title="Counter">
		<Button onclick={() => count++}>
			Clicked {count} times
		</Button>
	</Variant>

	<Variant title="Modal">
		<Button onclick={() => (modalOpen = true)}>Open Modal</Button>
		{#if modalOpen}
			<Modal onclose={() => (modalOpen = false)}>
				<h2>Modal Content</h2>
			</Modal>
		{/if}
	</Variant>
</Section>
```

### 4. Dark Mode Previews

Show components on different backgrounds:

```svelte
<Variant title="Light Button on Dark Background">
	<div style="background: #1a1a1a; padding: 2rem; border-radius: 8px;">
		<Button variant="light">Light Button</Button>
	</div>
</Variant>
```

### 5. Responsive Components

Show how components look at different sizes:

```svelte
<Variant title="Responsive Card">
	<div style="max-width: 400px; width: 100%;">
		<Card>
			<h3>Responsive Card</h3>
			<p>This card has a max-width for better readability</p>
		</Card>
	</div>
</Variant>
```

### 6. Complex Layouts

You can showcase complex component compositions:

```svelte
<Variant title="Dashboard Widget">
	<Card>
		<div
			style="display: flex; justify-content: space-between; align-items: center; margin-bottom: 1rem;"
		>
			<h3 style="margin: 0;">Total Sales</h3>
			<Badge color="green">+12%</Badge>
		</div>
		<div style="font-size: 2rem; font-weight: bold; margin-bottom: 0.5rem;">$12,345</div>
		<div style="color: #666; font-size: 0.875rem;">Compared to last month</div>
	</Card>
</Variant>
```

## Organizing Your Catalog

### Single Page

Simple projects can use a single catalog page:

```
src/routes/catalog/+page.svelte
```

### Multiple Pages

Larger projects can split into multiple catalog pages:

```
src/routes/catalog/buttons/+page.svelte
src/routes/catalog/forms/+page.svelte
src/routes/catalog/cards/+page.svelte
```

### Shared Layout

Create a shared layout for all catalog pages:

```svelte
<!-- src/routes/catalog/+layout.svelte -->
<script>
	import { page } from '$app/stores';
</script>

<nav>
	<a href="/catalog/buttons">Buttons</a>
	<a href="/catalog/forms">Forms</a>
	<a href="/catalog/cards">Cards</a>
</nav>

<slot />
```

## Deploying Your Catalog

You can deploy your catalog as:

1. **Part of your main app** - Add it as a route (e.g., `/catalog`)
2. **Separate deployment** - Deploy it separately for your team
3. **GitHub Pages** - Use `@sveltejs/adapter-static` for static hosting

Example for static deployment:

```bash
npm run build
# Upload the `build` directory to your hosting provider
```

## Integration with Existing Projects

### Adding to Existing SvelteKit App

1. Install: `npm install storylog`
2. Create: `src/routes/design-system/+page.svelte`
3. Import your components and create sections
4. Done!

### Standalone Catalog Project

Create a dedicated project just for your component catalog:

```bash
npm create svelte@latest my-catalog
cd my-catalog
npm install storylog
# Create your catalog pages
npm run dev
```

## Need Help?

- Check the README for API documentation
- Look at the demo page in the package
- Open an issue on GitHub

Happy cataloging! 🎨
