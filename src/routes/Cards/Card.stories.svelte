<script>
	import { Story } from '$lib/index.js';
	import { setContext } from 'svelte';

	// Receive props from Catalog
	let { section, fileName } = $props();

	// Provide context to Story children
	setContext('storiesFile', { section, fileName });
</script>

<Story title="Basic">
	<div class="card">
		<h3>Card Title</h3>
		<p>This is a basic card component with simple styling.</p>
	</div>
</Story>

<Story title="With Shadow">
	<div class="card card-shadow">
		<h3>Elevated Card</h3>
		<p>This card has a shadow for a floating effect.</p>
	</div>
</Story>

<Story title="With Image">
	<div class="card card-shadow card-image">
		<div class="card-image-header"></div>
		<div class="card-content">
			<h3>Image Card</h3>
			<p>Cards can contain images and other content.</p>
		</div>
	</div>
</Story>

<style>
	.card {
		border: 1px solid #e5e7eb;
		border-radius: 8px;
		padding: 1.5rem;
		background: white;
		max-width: 400px;
	}

	.card h3 {
		margin: 0 0 0.75rem 0;
		font-size: 1.25rem;
		color: #1a1a1a;
	}

	.card p {
		margin: 0;
		color: #6b7280;
		line-height: 1.6;
	}

	.card-shadow {
		border: none;
		box-shadow:
			0 4px 6px -1px rgb(0 0 0 / 0.1),
			0 2px 4px -2px rgb(0 0 0 / 0.1);
	}

	.card-image {
		padding: 0;
		overflow: hidden;
	}

	.card-image-header {
		height: 160px;
		background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
	}

	.card-content {
		padding: 1.5rem;
	}

	.card-image .card-content h3 {
		margin-top: 0;
	}

	.card-image .card-content p {
		margin-bottom: 0;
	}
</style>