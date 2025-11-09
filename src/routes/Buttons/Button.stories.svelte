<script>
	import { Story } from '$lib/index.js';
	import { setContext } from 'svelte';

	// Receive props from Catalog
	let { section, fileName } = $props();

	// Provide context to Story children
	setContext('storiesFile', { section, fileName });

	// Interactive state
	let count = $state(0);
</script>

<Story title="Primary">
	<button
		class="btn btn-primary"
		onclick={() => {
			count++;
		}}
	>
		Primary Button ({count})
	</button>
</Story>

<Story title="Secondary">
	<button class="btn btn-secondary">Secondary Button</button>
</Story>

<Story title="Danger">
	<button class="btn btn-danger">Delete</button>
</Story>

<Story title="Disabled">
	<button class="btn btn-primary" disabled>Disabled</button>
</Story>

<Story title="Small">
	<button class="btn btn-primary btn-sm">Small Button</button>
</Story>

<Story title="Large">
	<button class="btn btn-primary btn-lg">Large Button</button>
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
		font-family: inherit;
	}

	.btn:hover:not(:disabled) {
		transform: translateY(-1px);
	}

	.btn:disabled {
		opacity: 0.5;
		cursor: not-allowed;
	}

	.btn-primary {
		background: #3b82f6;
		color: white;
	}

	.btn-primary:hover:not(:disabled) {
		background: #2563eb;
	}

	.btn-secondary {
		background: #6b7280;
		color: white;
	}

	.btn-secondary:hover:not(:disabled) {
		background: #4b5563;
	}

	.btn-danger {
		background: #ef4444;
		color: white;
	}

	.btn-danger:hover:not(:disabled) {
		background: #dc2626;
	}

	.btn-sm {
		padding: 0.375rem 0.75rem;
		font-size: 0.875rem;
	}

	.btn-lg {
		padding: 0.75rem 1.5rem;
		font-size: 1.05rem;
	}
</style>