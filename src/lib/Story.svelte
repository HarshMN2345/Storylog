<script lang="ts">
	import type { Snippet } from 'svelte';
	import { getContext, onMount } from 'svelte';

	interface Props {
		title: string;
		children: Snippet;
	}

	let { title, children }: Props = $props();

	const catalog = getContext<{
		registerStory: (id: string, title: string, section: string, render: () => any) => void;
		selectedId: string;
	}>('catalog');

	const storiesFile = getContext<{ section: string; fileName: string }>('storiesFile');

	let storyId = $derived(
		storiesFile
			? `${storiesFile.section}-${storiesFile.fileName}-${title}`
					.toLowerCase()
					.replace(/[^a-z0-9]+/g, '-')
			: ''
	);

	let isSelected = $derived(catalog?.selectedId === storyId);

	onMount(() => {
		if (catalog && storiesFile) {
			catalog.registerStory(storyId, title, storiesFile.section, () => children);
		}
	});
</script>

{#if isSelected}
	<div class="story-wrapper">
		{@render children()}
	</div>
{/if}

<style>
	.story-wrapper {
		width: 100%;
		display: flex;
		align-items: center;
		justify-content: center;
	}
</style>
