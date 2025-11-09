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
	<rect x="2" y="3" width="12" height="9" rx="1" />
	<line x1="5" y1="12" x2="5" y2="13" />
	<line x1="11" y1="12" x2="11" y2="13" />
	<line x1="2" y1="12" x2="14" y2="12" />
</svg>

