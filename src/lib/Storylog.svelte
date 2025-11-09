<script lang="ts">
	import { setContext, onMount } from 'svelte';
	import {
		SunIcon,
		MoonIcon,
		MobileIcon,
		LargeMobileIcon,
		TabletIcon,
		DesktopIcon,
		MenuIcon,
		ResizeHandleIcon,
		ChevronLeftIcon,
		ChevronRightIcon,
		ChevronDownIcon,
		SearchIcon,
		CloseIcon,
		EmptyStateIcon,
		GridIcon,
		BookmarkIcon,
		DocumentIcon
	} from './icons/index.js';

	interface Props {
		title?: string;
		stories: Record<string, () => Promise<any>>;
	}

	interface StoryItem {
		id: string;
		title: string;
		section: string;
		render: () => any;
	}

	type Theme = 'light' | 'dark';
	type Viewport = 'small-mobile' | 'large-mobile' | 'tablet' | 'full';

	const VIEWPORT_SIZES: Record<Exclude<Viewport, 'full'>, number> = {
		'small-mobile': 320,
		'large-mobile': 375,
		tablet: 834
	};

	const DEFAULT_HEIGHT = 500;

	let { title = 'Storylog', stories }: Props = $props();
	let sidebarOpen = $state(true);
	let registeredStories = $state<StoryItem[]>([]);
	let selectedId = $state<string>('');
	let loading = $state(true);
	let loadedModules = $state<any[]>([]);
	let searchQuery = $state('');
	let searchInputRef: HTMLInputElement | null = $state(null);
	let theme = $state<Theme>('light');
	let viewport = $state<Viewport>('tablet');
	let expandedSections = $state<Set<string>>(new Set());
	let customHeight = $state<number | null>(null);
	let isResizing = $state(false);
	let resizeStartY = $state(0);
	let resizeStartHeight = $state(0);

	// Initialize theme from localStorage or system preference
	function initializeTheme() {
		if (typeof window === 'undefined') return;
		
		const saved = localStorage.getItem('storylog-theme') as Theme | null;
		if (saved === 'light' || saved === 'dark') {
			theme = saved;
		} else {
			const prefersDark = window.matchMedia('(prefers-color-scheme: dark)').matches;
			theme = prefersDark ? 'dark' : 'light';
		}
		applyTheme();
	}

	function applyTheme() {
		if (typeof document === 'undefined') return;
		document.documentElement.setAttribute('data-theme', theme);
		if (typeof window !== 'undefined') {
			localStorage.setItem('storylog-theme', theme);
		}
	}

	function toggleTheme() {
		theme = theme === 'light' ? 'dark' : 'light';
		applyTheme();
	}

	function setViewport(vp: Viewport) {
		viewport = vp;
		// Reset custom height when changing viewport
		customHeight = null;
	}

	function startResize(event: MouseEvent) {
		if (typeof window === 'undefined') return;
		isResizing = true;
		resizeStartY = event.clientY;
		const container = (event.target as HTMLElement).closest('.viewport-container') as HTMLElement;
		if (container) {
			// Use current custom height if set, otherwise use container's current height
			if (customHeight !== null) {
				resizeStartHeight = customHeight;
			} else {
				resizeStartHeight = container.offsetHeight;
				// Initialize customHeight with current height so it becomes "custom" from now on
				customHeight = resizeStartHeight;
			}
		}
		event.preventDefault();
		event.stopPropagation();
	}

	function handleResize(event: MouseEvent) {
		if (!isResizing) return;
		const deltaY = event.clientY - resizeStartY;
		const newHeight = resizeStartHeight + deltaY;
		// Minimum height of 200px, maximum reasonable height
		if (newHeight >= 200 && newHeight <= 5000) {
			customHeight = Math.round(newHeight);
		}
		event.preventDefault();
	}

	function stopResize() {
		isResizing = false;
	}

	function toggleSidebar() {
		sidebarOpen = !sidebarOpen;
	}

	function selectStory(id: string) {
		selectedId = id;
		// Auto-expand section containing this story
		const story = registeredStories.find((s) => s.id === id);
		if (story && !expandedSections.has(story.section)) {
			expandedSections.add(story.section);
			expandedSections = new Set(expandedSections);
		}
		// Close sidebar on mobile after selection
		if (typeof window !== 'undefined' && window.innerWidth <= 768) {
			sidebarOpen = false;
		}
	}

	function focusSearch() {
		searchInputRef?.focus();
	}

	function clearSearch() {
		searchQuery = '';
	}

	// Parse story path to extract section name - support both .story.svelte and .stories.svelte
	function parseStoryPath(path: string): { section: string; fileName: string } {
		// Remove leading ./ and trailing .story.svelte or .stories.svelte
		const cleaned = path
			.replace(/^\.\//, '')
			.replace(/\.stories?\.svelte$/, '');
		const parts = cleaned.split('/');

		if (parts.length === 1) {
			// Single file like "Button.stories.svelte" -> section: "Components", fileName: "Button"
			return {
				section: 'Components',
				fileName: parts[0]
			};
		} else {
			// Path like "buttons/Button.stories.svelte" -> section: "Buttons", fileName: "Button"
			const section = parts[0]
				.split('-')
				.map((word) => word.charAt(0).toUpperCase() + word.slice(1))
				.join(' ');
			const fileName = parts[parts.length - 1];
			return { section, fileName };
		}
	}

	// Context for Story components to register themselves
	const catalogContext = {
		registerStory: (id: string, storyTitle: string, section: string, render: () => any) => {
			registeredStories = [
				...registeredStories,
				{
					id,
					title: storyTitle,
					section,
					render
				}
			];
			// Select first story by default and expand its section
			if (selectedId === '') {
				selectedId = id;
				expandedSections.add(section);
				expandedSections = new Set(expandedSections);
			}
		},
		get selectedId() {
			return selectedId;
		}
	};

	setContext('catalog', catalogContext);

	// Load all story files
	onMount(async () => {
		initializeTheme();

		const modules = [];

		for (const [path, loader] of Object.entries(stories)) {
			const module = await loader();
			const { section, fileName } = parseStoryPath(path);

			// Set context for this stories file
			modules.push({
				component: module.default,
				section,
				fileName,
				path
			});
		}

		loadedModules = modules;
		loading = false;
	});

	// Filter stories by search query
	let filteredStories = $derived.by(() => {
		if (!searchQuery.trim()) {
			return registeredStories;
		}
		const query = searchQuery.toLowerCase().trim();
		return registeredStories.filter(
			(story) =>
				story.title.toLowerCase().includes(query) ||
				story.section.toLowerCase().includes(query) ||
				story.id.toLowerCase().includes(query)
		);
	});

	// Group filtered stories by section
	let groupedStories = $derived.by(() => {
		const sections: { [key: string]: StoryItem[] } = {};
		filteredStories.forEach((story) => {
			if (!sections[story.section]) {
				sections[story.section] = [];
			}
			sections[story.section].push(story);
		});
		return sections;
	});

	// Auto-expand sections when searching
	$effect(() => {
		if (searchQuery.trim()) {
			// Expand all sections that have matching stories
			const sectionsToExpand = new Set<string>();
			filteredStories.forEach((story) => {
				sectionsToExpand.add(story.section);
			});
			expandedSections = new Set([...expandedSections, ...sectionsToExpand]);
		}
	});

	let selectedStory = $derived(registeredStories.find((s) => s.id === selectedId));

	// Get all story IDs for navigation
	let allStoryIds = $derived(registeredStories.map((s) => s.id));
	let currentStoryIndex = $derived(
		selectedId ? allStoryIds.indexOf(selectedId) : -1
	);

	function navigateStories(direction: 'up' | 'down') {
		if (currentStoryIndex === -1 || allStoryIds.length === 0) return;
		
		let newIndex: number;
		if (direction === 'down') {
			newIndex = (currentStoryIndex + 1) % allStoryIds.length;
		} else {
			newIndex = currentStoryIndex === 0 ? allStoryIds.length - 1 : currentStoryIndex - 1;
		}
		selectStory(allStoryIds[newIndex]);
	}

	// Keyboard shortcuts
	function handleKeyDown(event: KeyboardEvent) {
		// Don't handle shortcuts when user is typing in an input
		if (
			event.target instanceof HTMLInputElement ||
			event.target instanceof HTMLTextAreaElement
		) {
			// Allow Esc to clear search
			if (event.key === 'Escape' && event.target === searchInputRef) {
				clearSearch();
				event.target.blur();
			}
			return;
		}

		const isMac = typeof navigator !== 'undefined' && /Mac|iPhone|iPod|iPad/i.test(navigator.userAgent);
		const modKey = isMac ? event.metaKey : event.ctrlKey;

		// K or Cmd/Ctrl+K: Focus search
		if (event.key === 'k' && !modKey) {
			event.preventDefault();
			focusSearch();
			return;
		}
		if ((event.key === 'k' || event.key === 'K') && modKey) {
			event.preventDefault();
			focusSearch();
			return;
		}

		// D or Cmd/Ctrl+D: Toggle dark mode
		if (event.key === 'd' && !modKey) {
			event.preventDefault();
			toggleTheme();
			return;
		}
		if ((event.key === 'd' || event.key === 'D') && modKey) {
			event.preventDefault();
			toggleTheme();
			return;
		}

		// 1-4: Switch viewport
		if (event.key >= '1' && event.key <= '4') {
			event.preventDefault();
			const viewports: Viewport[] = ['small-mobile', 'large-mobile', 'tablet', 'full'];
			setViewport(viewports[parseInt(event.key) - 1]);
			return;
		}

		// Esc: Clear search or close sidebar on mobile
		if (event.key === 'Escape') {
			if (searchQuery) {
				clearSearch();
			} else if (typeof window !== 'undefined' && window.innerWidth <= 768 && sidebarOpen) {
				toggleSidebar();
			}
			return;
		}

		// Arrow keys: Navigate stories
		if (event.key === 'ArrowDown') {
			event.preventDefault();
			navigateStories('down');
			return;
		}
		if (event.key === 'ArrowUp') {
			event.preventDefault();
			navigateStories('up');
			return;
		}
	}

	// Get viewport dimensions
	let viewportWidthStyle = $derived.by(() => {
		if (viewport === 'full') {
			return '100%';
		}
		return `${VIEWPORT_SIZES[viewport]}px`;
	});

	let viewportHeightStyle = $derived.by(() => {
		if (viewport === 'full') {
			return '100%';
		}
		return customHeight !== null ? `${customHeight}px` : `${DEFAULT_HEIGHT}px`;
	});
</script>

<svelte:window onkeydown={handleKeyDown} onmousemove={handleResize} onmouseup={stopResize} />

<div class="storylog">
	<aside class="sidebar" class:collapsed={!sidebarOpen}>
		<div class="sidebar-header">
			<h1 class="sidebar-title">{title}</h1>
			<button class="sidebar-toggle" onclick={toggleSidebar} aria-label="Toggle sidebar">
				{#if sidebarOpen}
					<ChevronLeftIcon />
				{:else}
					<ChevronRightIcon />
				{/if}
			</button>
		</div>
		<div class="search-container">
			<div class="search-wrapper">
				<SearchIcon class="search-icon" />
				<input
					bind:this={searchInputRef}
					type="text"
					class="search-input"
					placeholder="Find components..."
					bind:value={searchQuery}
					aria-label="Search components"
				/>
				{#if searchQuery}
					<button class="search-clear" onclick={clearSearch} aria-label="Clear search">
						<CloseIcon />
					</button>
				{/if}
				<div class="search-shortcut">
					<kbd>K</kbd>
				</div>
			</div>
		</div>
		<nav class="sidebar-nav">
			{#if loading}
				<div class="loading">Loading stories...</div>
			{:else if filteredStories.length === 0 && searchQuery}
				<div class="no-results">
					<p>No stories found matching "{searchQuery}"</p>
				</div>
			{:else}
				{#each Object.entries(groupedStories) as [section, stories] (section)}
					{@const isExpanded = expandedSections.has(section)}
					{@const hasActiveStory = stories.some((s) => s.id === selectedId)}
					<div class="nav-section">
						<button
							class="nav-section-header"
							class:has-active={hasActiveStory}
							onclick={() => {
								if (isExpanded) {
									expandedSections.delete(section);
								} else {
									expandedSections.add(section);
								}
								expandedSections = new Set(expandedSections);
							}}
							aria-expanded={isExpanded}
							aria-label={`${isExpanded ? 'Collapse' : 'Expand'} ${section}`}
						>
							<div class="nav-section-icon-wrapper">
								{#if isExpanded}
									<ChevronDownIcon width={14} height={14} />
								{:else}
									<ChevronRightIcon width={14} height={14} />
								{/if}
								<GridIcon width={16} height={16} />
							</div>
							<span class="nav-section-title">{section}</span>
						</button>
						{#if isExpanded}
							<div class="nav-section-stories">
								{#each stories as story (story.id)}
									<button
										class="nav-variant-link"
										class:active={selectedId === story.id}
										onclick={() => selectStory(story.id)}
									>
										<BookmarkIcon width={14} height={14} />
										<span>{story.title}</span>
									</button>
								{/each}
							</div>
						{/if}
					</div>
				{/each}
			{/if}
		</nav>
	</aside>

	<main class="main-content">
		<div class="canvas-header">
			<div class="header-left">
				<button class="header-menu-toggle" onclick={toggleSidebar} aria-label="Toggle sidebar">
					<MenuIcon />
				</button>
				<div class="canvas-breadcrumb">
					{#if selectedStory}
						<span class="breadcrumb-section">{selectedStory.section}</span>
						<span class="breadcrumb-separator">/</span>
						<span class="breadcrumb-variant">{selectedStory.title}</span>
					{:else}
						<span class="breadcrumb-section">{title}</span>
					{/if}
				</div>
			</div>
			{#if selectedStory}
				<div class="header-controls">
					<div class="viewport-controls">
						<button
							class="viewport-btn"
							class:active={viewport === 'small-mobile'}
							onclick={() => setViewport('small-mobile')}
							aria-label="Small mobile viewport (320px)"
							title="Small mobile (1)"
						>
							<MobileIcon />
							<span class="viewport-width">{VIEWPORT_SIZES['small-mobile']}</span>
						</button>
						<button
							class="viewport-btn"
							class:active={viewport === 'large-mobile'}
							onclick={() => setViewport('large-mobile')}
							aria-label="Large mobile viewport (375px)"
							title="Large mobile (2)"
						>
							<LargeMobileIcon />
							<span class="viewport-width">{VIEWPORT_SIZES['large-mobile']}</span>
						</button>
						<button
							class="viewport-btn"
							class:active={viewport === 'tablet'}
							onclick={() => setViewport('tablet')}
							aria-label="Tablet viewport (834px)"
							title="Tablet (3)"
						>
							<TabletIcon />
							<span class="viewport-width">{VIEWPORT_SIZES.tablet}</span>
						</button>
						<button
							class="viewport-btn"
							class:active={viewport === 'full'}
							onclick={() => setViewport('full')}
							aria-label="Full viewport (no constraints)"
							title="Full view (4)"
						>
							<DesktopIcon />
							<span class="viewport-width">Full</span>
						</button>
					</div>
					<button
						class="theme-toggle"
						onclick={toggleTheme}
						aria-label="Toggle theme"
						title="Toggle theme (D)"
					>
						{#if theme === 'dark'}
							<SunIcon />
						{:else}
							<MoonIcon />
						{/if}
					</button>
				</div>
			{/if}
		</div>

		<div class="canvas">
			<div
				class="viewport-container"
				class:small-mobile={viewport === 'small-mobile'}
				class:large-mobile={viewport === 'large-mobile'}
				class:tablet={viewport === 'tablet'}
				class:full={viewport === 'full'}
				class:resizing={isResizing}
				style="width: {viewportWidthStyle}; height: {viewportHeightStyle};"
			>
				<div class="canvas-content">
					{#if loading}
						<div class="loading-state">
							<div class="spinner"></div>
							<p>Loading stories...</p>
						</div>
					{:else}
						<!-- Render all story files (they contain Story components) -->
						{#each loadedModules as module (module.path)}
							{@const Component = module.component}
							<div style="display: contents;">
								<Component section={module.section} fileName={module.fileName} />
							</div>
						{/each}

						{#if registeredStories.length === 0}
							<div class="empty-state">
								<EmptyStateIcon />
								<h2>No stories found</h2>
								<p>Add .story.svelte or .stories.svelte files to see them here</p>
							</div>
						{/if}
					{/if}
				</div>
				{#if viewport !== 'full'}
					<button
						type="button"
						class="resize-handle"
						onmousedown={startResize}
						aria-label="Resize viewport height"
						tabindex="0"
					>
						<ResizeHandleIcon />
					</button>
				{/if}
			</div>
		</div>
	</main>
</div>

<style>
	:global(html[data-theme='light']),
	:global([data-theme='light']) {
		--storylog-bg: #ffffff;
		--storylog-sidebar-bg: #f6f9fc;
		--storylog-sidebar-header-bg: #ffffff;
		--storylog-border: #e3e8ef;
		--storylog-text: #1a1a1a;
		--storylog-text-secondary: #6b7280;
		--storylog-text-muted: #9ca3af;
		--storylog-hover: #f3f4f6;
		--storylog-active: #dbeafe;
		--storylog-active-text: #1e40af;
		--storylog-canvas-header-bg: #fafbfc;
		--storylog-input-bg: #ffffff;
		--storylog-input-border: #d1d5db;
	}

	:global(html[data-theme='dark']),
	:global([data-theme='dark']) {
		--storylog-bg: #1a1a1a;
		--storylog-sidebar-bg: #0f0f0f;
		--storylog-sidebar-header-bg: #1a1a1a;
		--storylog-border: #2d2d2d;
		--storylog-text: #e5e5e5;
		--storylog-text-secondary: #a3a3a3;
		--storylog-text-muted: #737373;
		--storylog-hover: #262626;
		--storylog-active: #1e3a5f;
		--storylog-active-text: #93c5fd;
		--storylog-canvas-header-bg: #0f0f0f;
		--storylog-input-bg: #262626;
		--storylog-input-border: #404040;
	}

	.storylog {
		display: flex;
		min-height: 100vh;
		background: var(--storylog-bg);
		color: var(--storylog-text);
		font-family: system-ui, -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
		position: relative;
		transition: background-color 0.2s ease, color 0.2s ease;
	}

	/* Sidebar */
	.sidebar {
		width: 280px;
		background: var(--storylog-sidebar-bg);
		border-right: 1px solid var(--storylog-border);
		display: flex;
		flex-direction: column;
		position: fixed;
		left: 0;
		top: 0;
		height: 100vh;
		transition: transform 0.3s ease, background-color 0.2s ease, border-color 0.2s ease;
		z-index: 100;
	}

	.sidebar.collapsed {
		transform: translateX(-280px);
	}

	.sidebar-header {
		padding: 1.25rem 1rem;
		border-bottom: 1px solid var(--storylog-border);
		display: flex;
		align-items: center;
		justify-content: space-between;
		background: var(--storylog-sidebar-header-bg);
		flex-shrink: 0;
		transition: background-color 0.2s ease, border-color 0.2s ease;
	}

	.sidebar-title {
		margin: 0;
		font-size: 1.125rem;
		font-weight: 600;
		color: var(--storylog-text);
		white-space: nowrap;
		overflow: hidden;
		text-overflow: ellipsis;
		transition: color 0.2s ease;
	}

	.sidebar-toggle {
		background: none;
		border: none;
		padding: 0.25rem;
		cursor: pointer;
		color: var(--storylog-text-secondary);
		display: flex;
		align-items: center;
		justify-content: center;
		border-radius: 4px;
		transition: all 0.2s;
	}

	.sidebar-toggle:hover {
		background: var(--storylog-hover);
		color: var(--storylog-text);
	}

	/* Search */
	.search-container {
		padding: 1rem;
		border-bottom: 1px solid var(--storylog-border);
		background: var(--storylog-sidebar-header-bg);
		transition: background-color 0.2s ease, border-color 0.2s ease;
	}

	.search-wrapper {
		position: relative;
		display: flex;
		align-items: center;
	}

	.search-wrapper :global(.search-icon) {
		position: absolute;
		left: 0.75rem;
		color: var(--storylog-text-secondary);
		pointer-events: none;
		z-index: 1;
		flex-shrink: 0;
	}

	.search-input {
		width: 100%;
		padding: 0.5rem 2.5rem 0.5rem 2.5rem;
		background: var(--storylog-input-bg);
		border: 1px solid var(--storylog-input-border);
		border-radius: 6px;
		font-size: 0.875rem;
		color: var(--storylog-text);
		transition: all 0.2s ease;
	}

	.search-input:focus {
		outline: none;
		border-color: var(--storylog-active-text);
		box-shadow: 0 0 0 3px rgba(59, 130, 246, 0.1);
	}

	.search-input::placeholder {
		color: var(--storylog-text-muted);
	}

	.search-clear {
		position: absolute;
		right: 2.5rem;
		background: none;
		border: none;
		padding: 0.25rem;
		cursor: pointer;
		color: var(--storylog-text-secondary);
		display: flex;
		align-items: center;
		justify-content: center;
		border-radius: 4px;
		transition: all 0.2s;
	}

	.search-clear:hover {
		background: var(--storylog-hover);
		color: var(--storylog-text);
	}

	.search-shortcut {
		position: absolute;
		right: 0.75rem;
		pointer-events: none;
	}

	.search-shortcut kbd {
		display: inline-flex;
		align-items: center;
		justify-content: center;
		padding: 0.125rem 0.375rem;
		font-size: 0.75rem;
		font-family: monospace;
		background: var(--storylog-hover);
		border: 1px solid var(--storylog-border);
		border-radius: 4px;
		color: var(--storylog-text-secondary);
	}

	.sidebar-nav {
		flex: 1;
		overflow-y: auto;
		padding: 1rem 0;
	}

	.loading,
	.no-results {
		padding: 1rem;
		text-align: center;
		color: var(--storylog-text-secondary);
		font-size: 0.875rem;
	}

	.nav-section {
		margin-bottom: 0.25rem;
	}

	.nav-section-header {
		display: flex;
		align-items: center;
		width: 100%;
		padding: 0.5rem 0.75rem;
		margin: 0 0.5rem;
		border: none;
		background: none;
		cursor: pointer;
		border-radius: 6px;
		transition: all 0.2s;
		color: var(--storylog-text);
		font-size: 0.875rem;
		font-weight: 500;
		text-align: left;
		gap: 0.5rem;
	}

	.nav-section-header:hover {
		background: var(--storylog-hover);
	}

	.nav-section-header.has-active {
		color: var(--storylog-text);
	}

	.nav-section-icon-wrapper {
		display: flex;
		align-items: center;
		gap: 0.375rem;
		flex-shrink: 0;
		color: var(--storylog-text-secondary);
		min-width: 2rem;
	}

	.nav-section-header :global(svg) {
		flex-shrink: 0;
	}

	.nav-section-title {
		flex: 1;
		font-size: 0.875rem;
		font-weight: 500;
		color: inherit;
		text-transform: none;
		letter-spacing: normal;
	}

	.nav-section-stories {
		margin-top: 0.125rem;
		padding-left: 1.75rem;
	}

	.nav-variant-link {
		display: flex;
		align-items: center;
		gap: 0.5rem;
		width: calc(100% - 1rem);
		padding: 0.5rem 0.75rem;
		margin: 0.125rem 0.5rem;
		color: var(--storylog-text);
		text-decoration: none;
		font-size: 0.875rem;
		font-weight: 400;
		border-radius: 6px;
		transition: all 0.2s;
		border: none;
		background: none;
		cursor: pointer;
		text-align: left;
	}

	.nav-variant-link :global(svg) {
		flex-shrink: 0;
		color: var(--storylog-text-secondary);
		opacity: 0.7;
	}

	.nav-variant-link:hover {
		background: var(--storylog-hover);
		color: var(--storylog-text);
	}

	.nav-variant-link:hover :global(svg) {
		opacity: 1;
	}

	.nav-variant-link.active {
		background: var(--storylog-active);
		color: var(--storylog-active-text);
	}

	.nav-variant-link.active :global(svg) {
		color: var(--storylog-active-text);
		opacity: 1;
	}

	/* Main Content */
	.main-content {
		flex: 1;
		margin-left: 280px;
		transition: margin-left 0.3s ease, background-color 0.2s ease;
		background: var(--storylog-bg);
		min-height: 100vh;
		display: flex;
		flex-direction: column;
	}

	.sidebar.collapsed + .main-content {
		margin-left: 0;
	}

	.canvas-header {
		padding: 1rem 1.5rem;
		border-bottom: 1px solid var(--storylog-border);
		background: var(--storylog-canvas-header-bg);
		display: flex;
		align-items: center;
		justify-content: space-between;
		transition: background-color 0.2s ease, border-color 0.2s ease;
	}

	.header-left {
		display: flex;
		align-items: center;
		gap: 1rem;
		flex: 1;
		min-width: 0;
	}

	.header-menu-toggle {
		display: flex;
		align-items: center;
		justify-content: center;
		padding: 0.5rem;
		background: var(--storylog-sidebar-bg);
		border: 1px solid var(--storylog-border);
		border-radius: 6px;
		cursor: pointer;
		color: var(--storylog-text-secondary);
		transition: all 0.2s;
		flex-shrink: 0;
	}

	/* Hide hamburger menu on desktop when sidebar is open (use sidebar toggle instead) */
	@media (min-width: 769px) {
		.sidebar:not(.collapsed) ~ .main-content .header-menu-toggle {
			display: none;
		}
	}

	.header-menu-toggle:hover {
		background: var(--storylog-hover);
		color: var(--storylog-text);
		border-color: var(--storylog-text-secondary);
	}

	.canvas-breadcrumb {
		display: flex;
		align-items: center;
		gap: 0.5rem;
		font-size: 0.875rem;
		min-width: 0;
	}

	.breadcrumb-section {
		color: var(--storylog-text-secondary);
		font-weight: 500;
		transition: color 0.2s ease;
	}

	.breadcrumb-separator {
		color: var(--storylog-text-muted);
		transition: color 0.2s ease;
	}

	.breadcrumb-variant {
		color: var(--storylog-text);
		font-weight: 600;
		font-size: 1.125rem;
		transition: color 0.2s ease;
	}

	.breadcrumb-section,
	.breadcrumb-variant {
		white-space: nowrap;
		overflow: hidden;
		text-overflow: ellipsis;
	}

	.header-controls {
		display: flex;
		align-items: center;
		gap: 0.75rem;
	}

	.viewport-controls {
		display: flex;
		gap: 0.5rem;
		background: var(--storylog-sidebar-bg);
		padding: 0.25rem;
		border-radius: 6px;
		border: 1px solid var(--storylog-border);
		transition: background-color 0.2s ease, border-color 0.2s ease;
	}

	.viewport-btn {
		display: flex;
		align-items: center;
		gap: 0.375rem;
		padding: 0.375rem 0.75rem;
		background: transparent;
		border: none;
		border-radius: 4px;
		cursor: pointer;
		color: var(--storylog-text-secondary);
		font-size: 0.75rem;
		font-weight: 500;
		transition: all 0.2s;
	}

	.viewport-btn:hover {
		background: var(--storylog-hover);
		color: var(--storylog-text);
	}

	.viewport-btn.active {
		background: var(--storylog-active);
		color: var(--storylog-active-text);
	}

	.viewport-btn :global(svg) {
		flex-shrink: 0;
	}

	.viewport-width {
		font-variant-numeric: tabular-nums;
		font-size: 0.75rem;
		font-weight: 500;
	}

	.theme-toggle {
		display: flex;
		align-items: center;
		justify-content: center;
		padding: 0.5rem;
		background: var(--storylog-sidebar-bg);
		border: 1px solid var(--storylog-border);
		border-radius: 6px;
		cursor: pointer;
		color: var(--storylog-text-secondary);
		transition: all 0.2s;
	}

	.theme-toggle:hover {
		background: var(--storylog-hover);
		color: var(--storylog-text);
		border-color: var(--storylog-text-secondary);
	}

	.canvas {
		flex: 1;
		padding: 3rem 2rem;
		display: flex;
		align-items: flex-start;
		justify-content: center;
		min-height: 400px;
		overflow-y: auto;
		overflow-x: auto;
	}

	.canvas:has(.viewport-container.full) {
		padding: 0;
		align-items: stretch;
		min-height: 0;
	}

	.viewport-container {
		transition: width 0.3s ease;
		margin: 0 auto;
		min-width: 0;
		border: 1px solid var(--storylog-border);
		border-radius: 8px;
		box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
		background: var(--storylog-bg);
		display: flex;
		flex-direction: column;
		position: relative;
		overflow: hidden;
	}

	.viewport-container.full {
		margin: 0;
		border: none;
		border-radius: 0;
		box-shadow: none;
		width: 100% !important;
		height: 100% !important;
		max-width: none;
		max-height: none;
	}

	.viewport-container:not(.resizing) {
		transition: width 0.3s ease, height 0.2s ease;
	}

	.viewport-container.full:not(.resizing) {
		transition: none;
	}

	:global([data-theme='dark']) .viewport-container:not(.full) {
		box-shadow: 0 4px 12px rgba(0, 0, 0, 0.3);
	}

	.canvas-content {
		width: 100%;
		padding: 2rem;
		padding-bottom: calc(2rem + 20px);
		flex: 1;
		overflow-y: auto;
		overflow-x: hidden;
		min-height: 0;
	}

	.viewport-container.full .canvas-content {
		padding-bottom: 2rem;
		height: 100%;
	}

	.resize-handle {
		position: absolute;
		bottom: 0;
		left: 0;
		right: 0;
		height: 20px;
		cursor: ns-resize;
		display: flex;
		align-items: center;
		justify-content: center;
		background: var(--storylog-sidebar-bg);
		border-top: 1px solid var(--storylog-border);
		border-bottom-left-radius: 8px;
		border-bottom-right-radius: 8px;
		color: var(--storylog-text-muted);
		transition: background-color 0.2s ease, color 0.2s ease, opacity 0.2s ease;
		z-index: 10;
		user-select: none;
		opacity: 0.7;
	}

	.resize-handle:hover {
		background: var(--storylog-hover);
		color: var(--storylog-text);
		opacity: 1;
	}

	.resize-handle:active {
		background: var(--storylog-active);
		color: var(--storylog-active-text);
		opacity: 1;
	}

	.viewport-container:hover .resize-handle {
		opacity: 1;
	}

	.loading-state {
		display: flex;
		flex-direction: column;
		align-items: center;
		justify-content: center;
		color: var(--storylog-text-secondary);
		gap: 1rem;
	}

	.spinner {
		width: 40px;
		height: 40px;
		border: 3px solid var(--storylog-border);
		border-top-color: var(--storylog-active-text);
		border-radius: 50%;
		animation: spin 0.8s linear infinite;
	}

	@keyframes spin {
		to {
			transform: rotate(360deg);
		}
	}

	.empty-state {
		flex: 1;
		display: flex;
		flex-direction: column;
		align-items: center;
		justify-content: center;
		color: var(--storylog-text-muted);
		padding: 3rem;
	}

	.empty-state :global(svg) {
		margin-bottom: 1rem;
	}

	.empty-state h2 {
		margin: 0 0 0.5rem 0;
		color: var(--storylog-text-secondary);
		font-size: 1.25rem;
	}

	.empty-state p {
		margin: 0;
		color: var(--storylog-text-muted);
	}

	/* Responsive */
	@media (max-width: 768px) {
		.sidebar {
			transform: translateX(-280px);
		}

		.sidebar:not(.collapsed) {
			transform: translateX(0);
		}

		.main-content {
			margin-left: 0;
		}

		.sidebar-toggle {
			display: none;
		}

		.canvas {
			padding: 2rem 1rem;
		}

		.canvas-header {
			padding: 1rem;
		}

		.header-controls {
			width: 100%;
			justify-content: space-between;
		}

		.viewport-controls {
			flex-wrap: wrap;
		}

	}

	/* Scrollbar styling */
	.sidebar-nav::-webkit-scrollbar {
		width: 8px;
	}

	.sidebar-nav::-webkit-scrollbar-track {
		background: transparent;
	}

	.sidebar-nav::-webkit-scrollbar-thumb {
		background: var(--storylog-border);
		border-radius: 4px;
	}

	.sidebar-nav::-webkit-scrollbar-thumb:hover {
		background: var(--storylog-text-muted);
	}
</style>

