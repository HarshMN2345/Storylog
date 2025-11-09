<script lang="ts">
	interface Props {
		width?: number;
		height?: number;
	}

	let { width = 16, height = 16 }: Props = $props();
</script>

<svg {width} {height} viewBox="0 0 16 16" fill="none">
	<path
		d="M8 2v2M8 12v2M2 8h2M12 8h2M3.757 3.757l1.414 1.414M10.829 10.829l1.414 1.414M3.757 12.243l1.414-1.414M10.829 5.171l1.414-1.414"
		stroke="currentColor"
		stroke-width="1.5"
		stroke-linecap="round"
	/>
	<circle cx="8" cy="8" r="2.5" stroke="currentColor" stroke-width="1.5" />
</svg>

