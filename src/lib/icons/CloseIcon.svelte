<script lang="ts">
	interface Props {
		width?: number;
		height?: number;
	}

	let { width = 14, height = 14 }: Props = $props();
</script>

<svg {width} {height} viewBox="0 0 14 14" fill="none">
	<path
		d="M10 4L4 10M4 4l6 6"
		stroke="currentColor"
		stroke-width="1.5"
		stroke-linecap="round"
	/>
</svg>

