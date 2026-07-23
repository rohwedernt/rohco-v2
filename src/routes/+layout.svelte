<script lang="ts">
	import { onMount, onDestroy, tick } from 'svelte';
	import { scale } from 'svelte/transition';
	import { quintOut } from 'svelte/easing';
	import '../app.css';
	import Icon from '@iconify/svelte';

	// Shared social link data. `icon` is the flat, single-color simple-icons
	// mark for each brand: used tinted-white-on-brand-color for the mobile
	// menu tiles, and reused as-is tinted black for the desktop row (a busy,
	// colorful video background made the full-color brand marks hard to read).
	const socialLinks = [
		{
			label: 'Instagram',
			icon: 'simple-icons:instagram',
			background: 'linear-gradient(135deg, #FEDA75 0%, #FA7E1E 30%, #D62976 60%, #962FBF 85%, #4F5BD5 100%)'
		},
		{ label: 'YouTube', icon: 'simple-icons:youtube', background: '#FF0000' },
		{ label: 'Spotify', icon: 'simple-icons:spotify', background: '#1DB954' },
		{ label: 'Apple Music', icon: 'simple-icons:applemusic', background: '#FA243C' },
		{ label: 'SoundCloud', icon: 'simple-icons:soundcloud', background: '#FF5500' },
		{ label: 'TikTok', icon: 'simple-icons:tiktok', background: '#000000' }
	];

	// <source media="..."> inside <video> is only evaluated once, during the
	// browser's initial resource-selection pass — unlike <picture><source>,
	// it never re-checks on resize. So the src is driven from JS via
	// matchMedia instead, kept in sync with the same 874px breakpoint the
	// CSS media queries below use.
	const DESKTOP_BREAKPOINT = '(min-width: 874px)';
	const MOBILE_VIDEO_SRC = '/rohco-sm-9.mp4';
	const DESKTOP_VIDEO_SRC = '/rohco-8.mp4';

	let videoSrc =
		typeof window !== 'undefined' && window.matchMedia(DESKTOP_BREAKPOINT).matches
			? DESKTOP_VIDEO_SRC
			: MOBILE_VIDEO_SRC;

	let bgVideo: HTMLVideoElement;
	let videoReady = false;
	let hasPlayedThrough = false;
	let menuOpen = false;
	let breakpointMql: MediaQueryList;

	function toggleMenu() {
		menuOpen = !menuOpen;
	}

	function handleVideoEnded() {
		hasPlayedThrough = true;
	}

	async function handleBreakpointChange(event: MediaQueryListEvent) {
		const nextSrc = event.matches ? DESKTOP_VIDEO_SRC : MOBILE_VIDEO_SRC;
		if (nextSrc === videoSrc) return;

		videoSrc = nextSrc;
		videoReady = false;
		hasPlayedThrough = false;

		// Wait for Svelte to flush the new src to the DOM before calling
		// play(), so we don't race and (re)play the outgoing resource.
		await tick();
		bgVideo?.play().catch(() => {});
	}

	// Backgrounded mobile tabs can get their decoded video buffer evicted to
	// save memory; on return the element is reset to frame 0 and autoplay
	// doesn't retrigger. Once the intro has played through, snap it back to
	// the final frame instead of replaying the whole animation.
	function restoreFinalFrame() {
		if (!bgVideo || !hasPlayedThrough) return;

		const seekToEnd = () => {
			if (bgVideo.duration && bgVideo.currentTime < bgVideo.duration - 0.1) {
				bgVideo.currentTime = bgVideo.duration;
			}
		};

		if (bgVideo.readyState >= 1) {
			seekToEnd();
		} else {
			bgVideo.addEventListener('loadedmetadata', seekToEnd, { once: true });
			bgVideo.load();
		}
	}

	function handleVisibilityChange() {
		if (document.visibilityState === 'visible') restoreFinalFrame();
	}

	function handlePageShow(event: PageTransitionEvent) {
		if (event.persisted) restoreFinalFrame();
	}

	function handleCanPlayThrough() {
		videoReady = true;
	}

	onMount(() => {
		if (bgVideo.readyState >= 4) {
			videoReady = true;
		}

		// Not { once: true } — this needs to keep firing so the video fades
		// back in each time handleBreakpointChange swaps the src.
		bgVideo.addEventListener('canplaythrough', handleCanPlayThrough);
		bgVideo.addEventListener('ended', handleVideoEnded);
		document.addEventListener('visibilitychange', handleVisibilityChange);
		window.addEventListener('pageshow', handlePageShow);

		breakpointMql = window.matchMedia(DESKTOP_BREAKPOINT);
		breakpointMql.addEventListener('change', handleBreakpointChange);
	});

	onDestroy(() => {
		if (typeof document !== 'undefined') {
			document.removeEventListener('visibilitychange', handleVisibilityChange);
		}
		if (typeof window !== 'undefined') {
			window.removeEventListener('pageshow', handlePageShow);
		}
		breakpointMql?.removeEventListener('change', handleBreakpointChange);
	});
</script>

<svelte:head>
	<title>Rohco</title>
</svelte:head>

<div class="loader" class:hidden={videoReady}>
	<div class="horns">🤘</div>
	<div class="equalizer">
		<span class="bar"></span>
		<span class="bar"></span>
		<span class="bar"></span>
		<span class="bar"></span>
		<span class="bar"></span>
	</div>
</div>

<video
	bind:this={bgVideo}
	src={videoSrc}
	autoplay
	muted
	playsinline
	preload="auto"
	class="background-video"
	class:video-ready={videoReady}
>
	Your browser does not support the video tag.
</video>

<button
	type="button"
	class="menu-bar"
	class:open={menuOpen}
	class:visible={videoReady}
	aria-expanded={menuOpen}
	aria-controls="mobile-menu-panel"
	on:click={toggleMenu}
>
	<span class="menu-bar-glyph" aria-hidden="true">
		<span class="menu-bar-glyph-line"></span>
		<span class="menu-bar-glyph-line"></span>
	</span>
	<span class="menu-bar-label">{menuOpen ? 'CLOSE' : 'MENU'}</span>
</button>

<div id="mobile-menu-panel" class="menu-panel" class:open={menuOpen} aria-hidden={!menuOpen}>
	<div class="menu-panel-handle" aria-hidden="true"></div>
	<nav class="menu-items">
		{#if menuOpen}
			{#each socialLinks as item, i (item.icon)}
				<a
					href="#"
					class="menu-item"
					aria-label={item.label}
					style="--card-bg: {item.background}"
					in:scale={{ duration: 340, delay: i * 45, start: 0.55, easing: quintOut }}
				>
					<Icon icon={item.icon} class="menu-item-icon" />
				</a>
			{/each}
		{/if}
	</nav>
</div>

<div class="desktop-social-row">
	{#each socialLinks as item (item.icon)}
		<a href="#" class="desktop-social-icon" aria-label={item.label}>
			<Icon icon={item.icon} class="desktop-social-icon-glyph" />
		</a>
	{/each}
</div>

<main>


	<slot />
</main>

<style>
	:global(#app),
	:global(body) {
		height: 100%;
		background: #ffffff;
	}

	.background-video {
		position: fixed;
		top: 0;
		right: 0;
		width: 100vw;
		height: 100vh;
		object-fit: cover;
		object-position: right top;
		z-index: -1;
		pointer-events: none;
		opacity: 0;
		transition: opacity 0.8s ease-in;

		@media (max-width: 874px) {
			object-position: center center;
		}
	}

	.background-video.video-ready {
		opacity: 1;
	}

	.loader {
		position: fixed;
		inset: 0;
		z-index: 100;
		background: linear-gradient(150deg, #f0f4ff 0%, #ffffff 50%, #f5f0ff 100%);
		display: flex;
		flex-direction: column;
		align-items: center;
		justify-content: center;
		gap: 2rem;
		opacity: 1;
		transition: opacity 0.8s ease-out;
		pointer-events: none;
	}

	.loader.hidden {
		opacity: 0;
	}

	.horns {
		font-size: 5rem;
		animation: pulse-horns 1.4s ease-in-out infinite;
		user-select: none;
	}

	@keyframes pulse-horns {
		0%, 100% {
			transform: scale(1);
			filter: drop-shadow(0 0 16px rgba(99, 102, 241, 0.4));
		}
		50% {
			transform: scale(1.18);
			filter: drop-shadow(0 0 32px rgba(99, 102, 241, 0.8));
		}
	}

	.equalizer {
		display: flex;
		align-items: flex-end;
		gap: 5px;
		height: 36px;
	}

	.bar {
		width: 7px;
		background: #6366f1;
		border-radius: 3px 3px 0 0;
		animation: equalize 0.7s ease-in-out infinite alternate;
	}

	.bar:nth-child(1) { animation-duration: 0.55s; }
	.bar:nth-child(2) { animation-duration: 0.85s; animation-delay: 0.1s; }
	.bar:nth-child(3) { animation-duration: 0.5s;  animation-delay: 0.2s; }
	.bar:nth-child(4) { animation-duration: 0.75s; animation-delay: 0.12s; }
	.bar:nth-child(5) { animation-duration: 0.65s; animation-delay: 0.05s; }

	@keyframes equalize {
		from { height: 6px; }
		to   { height: 36px; }
	}

	.desktop-social-row {
		display: none;
		position: fixed;
		right: 7.2rem;
		bottom: 2rem;
		z-index: 10;
		align-items: center;
		gap: 4.6rem;

		@media (min-width: 874px) {
			display: flex;
		}
	}

	.desktop-social-icon {
		display: flex;
		line-height: 0;
		color: #404040;
		filter:
			drop-shadow(0 0 1.5px #fff)
			drop-shadow(0 0 1.5px #fff)
			drop-shadow(0 0 1.5px #fff)
			drop-shadow(0 6px 14px rgba(0, 0, 0, 0.25));
		transition: transform 0.25s ease, filter 0.25s ease;
		opacity: 0;
		animation: desktopIconFadeIn 8s ease forwards;
	}

	@keyframes desktopIconFadeIn {
		from { opacity: 0; }
		to   { opacity: 1; }
	}

	.desktop-social-icon:hover {
		transform: translateY(-6px) scale(1.1);
		filter:
			drop-shadow(0 0 1.5px #fff)
			drop-shadow(0 0 1.5px #fff)
			drop-shadow(0 0 1.5px #fff)
			drop-shadow(0 10px 20px rgba(0, 0, 0, 0.3));
	}

	.desktop-social-icon:active {
		transform: translateY(-2px) scale(1.03);
	}

	.desktop-social-icon :global(.desktop-social-icon-glyph) {
		width: 4rem;
		height: 4rem;
	}

	.menu-bar {
		--menu-bar-top: 57vh;
		--menu-bar-height: 42px;

		display: none;
		position: fixed;
		top: var(--menu-bar-top);
		left: 0;
		right: 0;
		z-index: 20;
		height: var(--menu-bar-height);
		width: 100%;
		align-items: center;
		justify-content: center;
		gap: 0.6rem;
		background: rgba(255, 255, 255, 0.72);
		backdrop-filter: blur(14px) saturate(160%);
		-webkit-backdrop-filter: blur(14px) saturate(160%);
		border: none;
		border-top: 1px solid rgba(255, 255, 255, 0.6);
		border-bottom: 1px solid rgba(0, 0, 0, 0.06);
		box-shadow: 0 6px 20px rgba(0, 0, 0, 0.05);
		font: inherit;
		font-weight: 600;
		letter-spacing: 0.15em;
		font-size: 0.7rem;
		color: #111;
		cursor: pointer;
		opacity: 0;
		pointer-events: none;
		transition: opacity 4s ease-in, background 0.3s ease, box-shadow 0.3s ease;

		@media (max-width: 873px) {
			display: flex;
		}
	}

	.menu-bar:active {
		background: rgba(255, 255, 255, 0.9);
	}

	.menu-bar.open {
		box-shadow: 0 2px 10px rgba(0, 0, 0, 0.04);
	}

	.menu-bar.visible {
		opacity: 1;
		pointer-events: auto;
	}

	.menu-bar-glyph {
		display: flex;
		flex-direction: column;
		align-items: center;
		justify-content: center;
		gap: 5px;
		width: 16px;
	}

	.menu-bar-glyph-line {
		display: block;
		width: 100%;
		height: 2px;
		border-radius: 2px;
		background: #111;
		transition: transform 0.3s ease, opacity 0.3s ease;
	}

	.menu-bar.open .menu-bar-glyph-line:first-child {
		transform: translateY(3.5px) rotate(45deg);
	}

	.menu-bar.open .menu-bar-glyph-line:last-child {
		transform: translateY(-3.5px) rotate(-45deg);
	}

	.menu-bar-label {
		transform: translateY(1px);
	}

	.menu-panel {
		--menu-bar-top: 57vh;
		--menu-bar-height: 42px;

		display: none;
		position: fixed;
		top: calc(var(--menu-bar-top) + var(--menu-bar-height));
		left: 0;
		right: 0;
		bottom: 0;
		z-index: 19;
		background:
			radial-gradient(120% 90% at 15% 0%, rgba(99, 102, 241, 0.28), transparent 55%),
			radial-gradient(120% 90% at 85% 15%, rgba(232, 121, 249, 0.2), transparent 55%),
			rgba(18, 18, 26, 0.92);
		backdrop-filter: blur(22px) saturate(160%);
		-webkit-backdrop-filter: blur(22px) saturate(160%);
		box-shadow: 0 -8px 30px rgba(0, 0, 0, 0.35);
		clip-path: inset(0 0 100% 0);
		transition: clip-path 0.45s cubic-bezier(0.65, 0, 0.35, 1);
		pointer-events: none;

		@media (max-width: 873px) {
			display: block;
		}
	}

	.menu-panel.open {
		clip-path: inset(0 0 0 0);
		pointer-events: auto;
	}

	.menu-panel-handle {
		width: 36px;
		height: 4px;
		border-radius: 3px;
		background: rgba(255, 255, 255, 0.2);
		margin: 10px auto 0;
	}

	.menu-items {
		display: grid;
		grid-template-columns: repeat(3, 1fr);
		grid-template-rows: repeat(2, 1fr);
		gap: 16px;
		height: calc(100% - 30px);
		padding: 20px;
	}

	.menu-item {
		display: flex;
		align-items: center;
		justify-content: center;
		border-radius: 22px;
		background:
			linear-gradient(160deg, rgba(255, 255, 255, 0.32), rgba(255, 255, 255, 0) 45%),
			var(--card-bg);
		box-shadow:
			inset 0 1px 1px rgba(255, 255, 255, 0.25),
			0 10px 20px -6px rgba(0, 0, 0, 0.28);
		text-decoration: none;
		-webkit-tap-highlight-color: transparent;
		transition: transform 0.15s ease, filter 0.15s ease;
	}

	.menu-item:active {
		transform: scale(0.93);
		filter: brightness(0.94);
	}

	.menu-item :global(.menu-item-icon) {
		width: 2.5rem;
		height: 2.5rem;
		color: #ffffff;
	}

	@keyframes fadeIn {
		from {
			opacity: 0;
		}
		to {
			opacity: 1;
		}
	}
</style>
