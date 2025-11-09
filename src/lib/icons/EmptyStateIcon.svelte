<script lang="ts">
	interface Props {
		width?: number;
		height?: number;
	}

	let { width = 64, height = 64 }: Props = $props();
</script>

<svg {width} {height} viewBox="0 0 24 24" fill="none" stroke="currentColor">
	<rect x="3" y="3" width="18" height="18" rx="2" ry="2" stroke-width="2" />
	<line x1="9" y1="9" x2="15" y2="9" stroke-width="2" />
	<line x1="9" y1="15" x2="15" y2="15" stroke-width="2" />
</svg>

