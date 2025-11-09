<script lang="ts">
	interface Props {
		width?: number;
		height?: number;
	}

	let { width = 16, height = 16 }: Props = $props();
</script>

<svg {width} {height} viewBox="0 0 16 16" fill="none">
	<rect x="3.5" y="2" width="9" height="12" rx="1" stroke="currentColor" stroke-width="1.5" />
	<line x1="5.5" y1="4" x2="10.5" y2="4" stroke="currentColor" stroke-width="1.5" />
</svg>

