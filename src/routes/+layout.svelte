<script lang="ts">
	import '../app.css';
	import { onMount, onDestroy } from 'svelte';
	import Icon from '@iconify/svelte';

	const icons = [
		{ default: 'grommet-icons:youtube', hover: 'logos:youtube-icon' },
		{ default: 'grommet-icons:spotify', hover: 'logos:spotify-icon' },
		{ default: 'simple-icons:instagram', hover: 'skill-icons:instagram' },
		{ default: 'simple-icons:tiktok', hover: 'logos:tiktok-icon' }
	];

	let hoveredIndex: number | null = null;
	let bgVideo: HTMLVideoElement;
	let isMobile: boolean;

	function updateViewport() {
		const nowMobile = window.matchMedia('(max-width: 799px)').matches;
		if (nowMobile !== isMobile) {
			isMobile = nowMobile;
			if (bgVideo) {
				bgVideo.load();
				bgVideo.play();
			}
		}
	}

	onMount(() => {
	requestAnimationFrame(() => {
		isMobile = window.matchMedia('(max-width: 799px)').matches;
		if (bgVideo) {
			bgVideo.play();
		}
		window.addEventListener('resize', updateViewport);
	});
});

	onDestroy(() => {
		if (typeof window !== 'undefined') {
			window.removeEventListener('resize', updateViewport);
		}
	});
</script>

<video bind:this={bgVideo} autoplay muted playsinline class="background-video">
	<source src={isMobile ? '/rohco-sm.mp4' : '/rohco.mp4'} type="video/mp4" />
	Your browser does not support the video tag.
</video>

<main>
	<div class="content-wrapper flex flex-col gap-12">
		{#each icons as icon, i}
			<div
				role="img"
				on:mouseenter={() => (hoveredIndex = i)}
				on:mouseleave={() => (hoveredIndex = null)}
				class="icon-wrapper cursor-pointer transition-all duration-200"
			>
				<Icon
					icon={hoveredIndex === i ? icon.hover : icon.default}
					class="h-full w-full text-gray-800"
				/>
			</div>
		{/each}
	</div>

	<!-- Preload all hover variants offscreen -->
	<div class="hidden">
		<Icon icon="logos:youtube-icon" />
		<Icon icon="logos:spotify-icon" />
		<Icon icon="skill-icons:instagram" />
		<Icon icon="logos:tiktok-icon" />
	</div>

	<slot />
</main>

<style>
	:global(#app),
	:global(body) {
		height: 100%;
	}

	.content-wrapper {
		position: fixed;
		top: 8px;
		right: 8px;
		z-index: 10;

		@media (max-width: 800px) {
			right: unset;
			left: 8px;
		}
	}

	.background-video {
		position: fixed;
		top: 0;
		right: 0;
		width: 100vw;
		height: 100vh;
		object-fit: cover;
		object-position: center top;
		z-index: -1;
		pointer-events: none;

		@media (min-width: 940px) {
			object-position: right top;
		}

		@media (max-width: 800px) {
			object-position: center center;
		}
	}

	.icon-wrapper {
		width: 80px;
		height: 80px;
	}

	@media (max-width: 800px) {
		.icon-wrapper {
			width: 60px;
			height: 60px;
		}
	}
</style>
