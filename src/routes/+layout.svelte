<script lang="ts">
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
</script>

<video bind:this={bgVideo} autoplay muted playsinline class="background-video">
	<source src="/rohco-sm.mp4" type="video/mp4" media="(max-width: 799px)" />
	<source src="/rohco.mp4" type="video/mp4" media="(min-width: 800px)" />
	Your browser does not support the video tag.
</video>

<main>
	<div class="flex items-center justify-center h-screen w-screen">
		<h1 class="coming-soon mt-50 text-white font-stretch-ultra-expanded tracking-widest">— coming soon —</h1>
	</div>
	<div class="content-wrapper flex flex-col gap-12">
		{#each icons as icon, i}
			<div
				role="img"
				on:mouseenter={() => (hoveredIndex = i)}
				on:mouseleave={() => (hoveredIndex = null)}
				class=" hidden icon-wrapper cursor-pointer transition-all duration-200"
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
