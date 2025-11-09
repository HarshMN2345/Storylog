<script lang="ts">
	interface Props {
		width?: number;
		height?: number;
	}

	let { width = 16, height = 16 }: Props = $props();
</script>

<svg {width} {height} viewBox="0 0 16 16" fill="none">
	<rect x="2" y="2" width="12" height="12" rx="1" stroke="currentColor" stroke-width="1.5" />
	<line x1="4" y1="4" x2="12" y2="4" stroke="currentColor" stroke-width="1.5" />
</svg>

