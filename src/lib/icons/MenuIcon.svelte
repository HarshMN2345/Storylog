<script lang="ts">
	interface Props {
		width?: number;
		height?: number;
	}

	let { width = 20, height = 20 }: Props = $props();
</script>

<svg {width} {height} viewBox="0 0 20 20" fill="none">
	<path
		d="M3 5H17M3 10H17M3 15H17"
		stroke="currentColor"
		stroke-width="2"
		stroke-linecap="round"
	/>
</svg>

