<script lang="ts">
	import '../app.css';
	import { onMount } from 'svelte';
	import Icon from '@iconify/svelte';

	const icons = [
		{ default: 'grommet-icons:youtube', hover: 'logos:youtube-icon' },
		{ default: 'grommet-icons:spotify', hover: 'logos:spotify-icon' },
		{ default: 'simple-icons:instagram', hover: 'skill-icons:instagram' },
		{ default: 'simple-icons:tiktok', hover: 'logos:tiktok-icon' }
	];

	let hoveredIndex: number | null = null;

	let bgVideo: HTMLVideoElement;
	let videoEnded = false;

	onMount(() => {
		bgVideo.play();
	});
</script>

<video
	bind:this={bgVideo}
	class="background-video"
	src="/rohco.mp4"
	autoplay
	muted
	playsinline
	on:ended={() => (videoEnded = true)}
></video>

<!-- Main routed content -->
<main>
	<div class="fixed top-8 right-8 z-10 flex flex-col gap-12">
		{#each icons as icon, i}
			<div
				role="img"
				on:mouseenter={() => (hoveredIndex = i)}
				on:mouseleave={() => (hoveredIndex = null)}
				class="cursor-pointer transition-all duration-200"
			>
				<Icon
					icon={hoveredIndex === i ? icon.hover : icon.default}
					width="80"
					height="80"
					class="text-gray-800"
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
	}
</style>
