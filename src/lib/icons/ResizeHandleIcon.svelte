<script lang="ts">
	interface Props {
		width?: number;
		height?: number;
	}

	let { width = 20, height = 8 }: Props = $props();
</script>

<svg {width} {height} viewBox="0 0 20 8" fill="none">
	<path
		d="M2 4H18M6 2H14M6 6H14"
		stroke="currentColor"
		stroke-width="1.5"
		stroke-linecap="round"
	/>
</svg>

