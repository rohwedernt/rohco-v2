<script lang="ts">
	import { onMount } from 'svelte';
	import '../app.css';
	import Icon from '@iconify/svelte';

	const icons = [
		{ default: 'grommet-icons:youtube', hover: 'logos:youtube-icon' },
		{ default: 'grommet-icons:spotify', hover: 'logos:spotify-icon' },
		{ default: 'simple-icons:instagram', hover: 'skill-icons:instagram' },
		{ default: 'simple-icons:tiktok', hover: 'logos:tiktok-icon' }
	];

	let hoveredIndex: number | null = null;
	let bgVideo: HTMLVideoElement;
	let videoReady = false;

	onMount(() => {
		if (bgVideo.readyState >= 4) {
			videoReady = true;
		} else {
			bgVideo.addEventListener('canplaythrough', () => (videoReady = true), { once: true });
		}
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

<video bind:this={bgVideo} autoplay muted playsinline preload="auto" class="background-video" class:video-ready={videoReady}>
	<source src="/rohco-sm.mp4" type="video/mp4" media="(max-width: 799px)" />
	<source src="/rohco.mp4" type="video/mp4" media="(min-width: 800px)" />
	Your browser does not support the video tag.
</video>

<main>
	<div class="flex h-screen w-screen items-center justify-center">
		<h1 class="coming-soon mt-48 tracking-widest text-white font-stretch-ultra-expanded">
			— coming soon —
		</h1>
	</div>
	<div class="content-wrapper flex flex-col gap-12">
		{#each icons as icon, i}
			<div
				role="img"
				aria-label={i === 0
					? 'YouTube icon'
					: i === 1
						? 'Spotify icon'
						: i === 2
							? 'Instagram icon'
							: i === 3
								? 'TikTok icon'
								: ''}
				on:mouseenter={() => (hoveredIndex = i)}
				on:mouseleave={() => (hoveredIndex = null)}
				class=" icon-wrapper hidden cursor-pointer transition-all duration-200"
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
		background: #0a0a0a;
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
		opacity: 0;
		transition: opacity 0.8s ease-in;

		@media (min-width: 940px) {
			object-position: right top;
		}

		@media (max-width: 800px) {
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
		background: #0a0a0a;
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
			filter: drop-shadow(0 0 16px rgba(220, 38, 38, 0.5));
		}
		50% {
			transform: scale(1.18);
			filter: drop-shadow(0 0 36px rgba(220, 38, 38, 1));
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
		background: #dc2626;
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

	.coming-soon {
		opacity: 0;
		animation: fadeIn 1s ease-in forwards;
		animation-delay: 2s;
		font-size: 2rem;

		@media (max-width: 800px) {
			font-size: 1.5rem;
		}
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
