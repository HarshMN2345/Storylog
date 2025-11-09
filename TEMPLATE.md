# Quick Start Template

Copy this template to get started with your component catalog quickly!

## Basic Template

```svelte
<script>
	import { Catalog, Section, Variant } from 'svelte-catalog';
	// Import your components here
	// import Button from '$lib/Button.svelte';
	// import Card from '$lib/Card.svelte';
</script>

<Catalog title="My Component Library">
	<Section title="Component Category 1" description="Description of this category">
		<Variant title="Variant Name" description="What this variant shows">
			<!-- Your component here -->
			<div>Your component goes here</div>
		</Variant>

		<Variant title="Another Variant">
			<!-- Another variation -->
		</Variant>
	</Section>

	<Section title="Component Category 2">
		<Variant title="Example">
			<!-- More components -->
		</Variant>
	</Section>
</Catalog>
```

## Complete Example Template

```svelte
<script>
	import { Catalog, Section, Variant } from 'svelte-catalog';

	// Import your actual components
	// import Button from '$lib/Button.svelte';
	// import Card from '$lib/Card.svelte';
	// import Input from '$lib/Input.svelte';

	// Add any state you need for interactive demos
	let count = $state(0);
	let inputValue = $state('');
</script>

<Catalog title="Design System">
	<!-- Buttons Section -->
	<Section title="Buttons" description="Interactive button components for user actions">
		<Variant title="Primary Button" description="Main call-to-action button">
			<button
				style="padding: 0.5rem 1rem; background: #3b82f6; color: white; border: none; border-radius: 6px; cursor: pointer;"
				onclick={() => count++}
			>
				Click me ({count})
			</button>
		</Variant>

		<Variant title="Button Sizes" description="Small, medium, and large buttons">
			<div style="display: flex; gap: 1rem; align-items: center;">
				<button
					style="padding: 0.375rem 0.75rem; background: #3b82f6; color: white; border: none; border-radius: 6px;"
					>Small</button
				>
				<button
					style="padding: 0.5rem 1rem; background: #3b82f6; color: white; border: none; border-radius: 6px;"
					>Medium</button
				>
				<button
					style="padding: 0.75rem 1.5rem; background: #3b82f6; color: white; border: none; border-radius: 6px;"
					>Large</button
				>
			</div>
		</Variant>

		<Variant title="Disabled State" description="Non-interactive button">
			<button
				disabled
				style="padding: 0.5rem 1rem; background: #9ca3af; color: white; border: none; border-radius: 6px; opacity: 0.6; cursor: not-allowed;"
			>
				Disabled
			</button>
		</Variant>
	</Section>

	<!-- Cards Section -->
	<Section title="Cards" description="Content containers with various styles">
		<Variant title="Basic Card">
			<div
				style="border: 1px solid #e5e7eb; border-radius: 8px; padding: 1.5rem; max-width: 400px; background: white;"
			>
				<h3 style="margin: 0 0 0.5rem 0;">Card Title</h3>
				<p style="margin: 0; color: #6b7280;">
					This is a simple card component with a title and content.
				</p>
			</div>
		</Variant>

		<Variant title="Card with Shadow">
			<div
				style="border-radius: 8px; padding: 1.5rem; max-width: 400px; background: white; box-shadow: 0 4px 6px -1px rgb(0 0 0 / 0.1);"
			>
				<h3 style="margin: 0 0 0.5rem 0;">Elevated Card</h3>
				<p style="margin: 0; color: #6b7280;">This card has a shadow for a floating effect.</p>
			</div>
		</Variant>
	</Section>

	<!-- Forms Section -->
	<Section title="Form Elements" description="Input fields and form controls">
		<Variant title="Text Input">
			<div style="max-width: 400px;">
				<label style="display: block; margin-bottom: 0.5rem; font-weight: 500; color: #374151;">
					Email
				</label>
				<input
					type="email"
					bind:value={inputValue}
					placeholder="you@example.com"
					style="width: 100%; padding: 0.5rem 0.75rem; border: 1px solid #d1d5db; border-radius: 6px;"
				/>
			</div>
		</Variant>

		<Variant title="Checkbox">
			<label style="display: flex; align-items: center; gap: 0.5rem; cursor: pointer;">
				<input type="checkbox" style="width: 18px; height: 18px;" />
				<span>I agree to the terms and conditions</span>
			</label>
		</Variant>
	</Section>

	<!-- Badges Section -->
	<Section title="Badges" description="Labels and status indicators">
		<Variant title="Badge Colors">
			<div style="display: flex; gap: 0.5rem; flex-wrap: wrap;">
				<span
					style="padding: 0.25rem 0.75rem; background: #dbeafe; color: #1e40af; border-radius: 9999px; font-size: 0.875rem; font-weight: 500;"
					>Primary</span
				>
				<span
					style="padding: 0.25rem 0.75rem; background: #dcfce7; color: #166534; border-radius: 9999px; font-size: 0.875rem; font-weight: 500;"
					>Success</span
				>
				<span
					style="padding: 0.25rem 0.75rem; background: #fef3c7; color: #92400e; border-radius: 9999px; font-size: 0.875rem; font-weight: 500;"
					>Warning</span
				>
				<span
					style="padding: 0.25rem 0.75rem; background: #fee2e2; color: #991b1b; border-radius: 9999px; font-size: 0.875rem; font-weight: 500;"
					>Danger</span
				>
			</div>
		</Variant>
	</Section>
</Catalog>
```

## File Structure

Place your catalog in your SvelteKit project:

```
src/
├── lib/
│   └── components/      # Your actual components
│       ├── Button.svelte
│       ├── Card.svelte
│       └── ...
└── routes/
    └── catalog/         # Your catalog page(s)
        └── +page.svelte # Main catalog
```

## Tips

1. **Replace placeholder content**: Swap the inline styles with your actual components
2. **Add more sections**: Copy the `<Section>` block for each component category
3. **Add more variants**: Copy the `<Variant>` block for each variation
4. **Make it interactive**: Use `$state()` for reactive demos
5. **Organize by category**: Group related components together

## Next Steps

1. Copy this template to `src/routes/catalog/+page.svelte`
2. Import your components
3. Replace the placeholder content
4. Run `npm run dev`
5. Navigate to `/catalog`

Happy cataloging! 🎨
