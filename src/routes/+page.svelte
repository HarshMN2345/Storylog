<script lang="ts">
	import { Storylog } from '$lib/index.js';

	// Automatically discover all .story.svelte and .stories.svelte files
	const storyFiles = import.meta.glob('./**/*.story.svelte');
	const storiesFiles = import.meta.glob('./**/*.stories.svelte');
	const stories = { ...storyFiles, ...storiesFiles };
</script>

<Storylog title="Storylog" {stories} />

