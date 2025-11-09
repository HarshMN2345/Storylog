<script lang="ts">
	interface Props {
		width?: number;
		height?: number;
		class?: string;
	}

	let { width = 16, height = 16, class: className = '' }: Props = $props();
</script>

<svg
	{width}
	{height}
	viewBox="0 0 16 16"
	fill="none"
	stroke="currentColor"
	stroke-width="1.5"
	stroke-linecap="round"
	stroke-linejoin="round"
	class={className || undefined}
>
	<rect x="2" y="2" width="5" height="5" rx="1" />
	<rect x="9" y="2" width="5" height="5" rx="1" />
	<rect x="2" y="9" width="5" height="5" rx="1" />
	<rect x="9" y="9" width="5" height="5" rx="1" />
</svg>

