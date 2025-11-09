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
	<path d="M4 2h6l4 4v8H4V2z" />
	<polyline points="10 2 10 6 14 6" />
</svg>

