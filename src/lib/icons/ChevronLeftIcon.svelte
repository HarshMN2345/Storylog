<script lang="ts">
	interface Props {
		width?: number;
		height?: number;
	}

	let { width = 16, height = 16 }: Props = $props();
</script>

<svg {width} {height} viewBox="0 0 16 16" fill="none">
	<path
		d="M10 12L6 8L10 4"
		stroke="currentColor"
		stroke-width="2"
		stroke-linecap="round"
		stroke-linejoin="round"
	/>
</svg>

