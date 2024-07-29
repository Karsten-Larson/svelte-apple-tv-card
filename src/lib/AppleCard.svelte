<script lang="ts">
	import { onMount } from 'svelte';

	// export let backgroundImage;
	export let rounded: boolean = true;
	export let titleAlwaysVisible = false;

	let container: HTMLDivElement;
	let card: HTMLDivElement;
	let content: HTMLDivElement;
	let parallaxContents: HTMLDivElement;

	let reflection: HTMLSpanElement;
	let shadow: HTMLSpanElement;

	let mediaQuery: undefined | MediaQueryList;
	onMount(() => {
		// Gets user's browser preferences
		mediaQuery = window.matchMedia('(prefers-reduced-motion: reduce)');

		// Allow content to be tabbed to
		content.setAttribute('tabindex', '0');
	});

	let hover: boolean = false;

	const handleStart = (event: MouseEvent) => {
		// content.focus();
		hover = true;
		handleMove(event);
	};

	// Handling mouse movement over element
	const handleMove = (event: MouseEvent) => {
		// Abort if mediaQuery does not exist or does not match
		// if (!mediaQuery || !mediaQuery.matches) return;

		// Mouse position on the card
		const posX = event.offsetX;
		const posY = event.offsetY;

		// Calculate needed perspecitve and angle changes
		const width = card.clientWidth;
		const height = card.clientHeight;
		const angleY = ((width / 2 - posX) / width) * 10;
		const angleX = (((height / 2 - posY) * -1) / height) * 10;
		const translateX = (((width / 2 - posX) * -1) / width) * 10;
		const translateY = (((height / 2 - posY) * -1) / height) * 10;

		// Change container perspective
		const perspective = Math.max(width, height);
		container.style.perspective = perspective * 2.5 + 'px';

		// Change apply transformations to the card
		card.style.transform =
			'translateZ(4rem) rotateY(' +
			angleY +
			'deg) rotateX(' +
			angleX +
			'deg) translateX(' +
			translateX +
			'px) translateY(' +
			translateY +
			'px)';

		// Apply transformation to all parallax elements
		Array.prototype.slice
			.call(parallaxContents.children)
			.forEach((parallaxContent, layer: number) => {
				layer++;
				const modifier = !parallaxContent.classList.contains('reverse') ? -0.65 : 0.2;
				parallaxContent.style.transform =
					'scale(1.075) translateX(' +
					translateX * modifier * layer +
					'px) translateY(' +
					translateY * modifier * layer +
					'px)';
			});

		// Apply transformation to reflection
		reflection.style.width = perspective * 1.5 + 'px';
		reflection.style.height = perspective * 1.5 + 'px';
		reflection.style.margin = perspective * -0.75 + 'px 0 0 ' + perspective * -0.75 + 'px';
		reflection.style.transform =
			'translateY(' + (posY - height / 2) + 'px) translateX(' + (width * 0.1 + posX * 0.8) + 'px)';

		// Apply transformation to shadow
		if (posY < height / 3) {
			const opacity = (1 / (height / 3)) * (height / 3 - posY);
			shadow.style.opacity = opacity.toString();
			shadow.style.boxShadow =
				'inset 0 ' + opacity * -1 + 'em .4em -.5em rgba(0,0,0,' + Math.min(opacity, 0.35) + ')';
		}
	};

	const handleEnd = (event: MouseEvent) => {
		content.blur();
		hover = false;

		card.style.transform = '';

		// Reset everything to the defaults
		Array.prototype.slice.call(parallaxContents.children).forEach((parallaxContent) => {
			parallaxContent.style.transform = null;
		});

		reflection.style.transform = '';
		shadow.style.boxShadow = '';
		shadow.style.opacity = '';
	};
</script>

<div class="apple-tv-card-container" bind:this={container}>
	<div
		role="group"
		class="apple-tv-card"
		class:no-title={!$$slots.title}
		class:hover
		class:not-rounded={!rounded}
		bind:this={card}
		on:mouseenter={handleStart}
		on:mousemove={handleMove}
		on:mouseleave={handleEnd}
	>
		<!-- Added effects for on hover -->
		<span class="shadow" bind:this={shadow} />
		<span class="reflection" bind:this={reflection} />

		<div
			class="content"
			bind:this={content}
			on:focus={() => {
				const width = card.clientWidth;
				const height = card.clientHeight;

				const perspective = Math.max(width, height);

				container.style.perspective = perspective * 2.5 + 'px';
			}}
		>
			<!-- Any non-parallax content -->
			<slot name="nonparallax" />
		</div>
		<div class="parallax-content" bind:this={parallaxContents}>
			<!-- Example -->
			<!-- End: Example -->
			<slot name="parallax" />
		</div>
	</div>

	<div class="apple-tv-card-title" class:always-visible={titleAlwaysVisible}>
		<slot name="title" />
	</div>
</div>

<style lang="scss">
	.apple-tv-card-container {
		position: relative;
		width: 100%;
		padding-bottom: 3.5rem;

		&.no-title {
			padding-bottom: 0;
		}

		& > .apple-tv-card {
			width: 100%;
			border-radius: min(max(2vmax, 2rem), 3rem);
			box-shadow: 0 0.25rem 0.25rem #0002;
			transform-origin: 50%;
			transition: transform 50ms ease-in-out;
			transform-style: preserve-3d;
			overflow: hidden;
			position: relative;
			z-index: 0;

			&.not-rounded {
				border-radius: 0;
			}

			@media (prefers-reduced-motion: reduce) {
				transition: none;
			}

			&.hover,
			&:focus-within {
				box-shadow: 0 1.5rem 2rem 0.25rem #0005;
				outline: none;
				transform: translateZ(4rem);

				& + .apple-tv-card-title {
					bottom: 0.9rem;
					opacity: 1;
				}
			}

			& > .shadow {
				position: absolute;
				top: 0;
				right: 0;
				bottom: 0;
				left: 0;
				pointer-events: none;
				background: #0002;
				opacity: 0;
				z-index: 3;
			}

			& > .reflection {
				position: absolute;
				top: 0;
				left: 0;
				pointer-events: none;
				background-image: radial-gradient(#fff7, transparent 70%);
				transform: translateY(-100%);
				z-index: 4;
			}

			& > .content {
				display: block !important;
				width: 100%;
				border: none;
				outline: none;
				background-image: var(--background-image, none);
				background-repeat: var(--background-repeat, no-repeat);
				background-size: var(--background-size, cover);
				// background-image: linear-gradient(to bottom, #555, #000);
				background-position: center center;
				padding-bottom: 58%;
				z-index: 1;
				position: relative !important;

				* {
					pointer-events: none;
				}
			}

			& > .parallax-content {
				transform-style: preserve-3d;
				position: absolute;
				top: 0;
				right: 0;
				bottom: 0;
				left: 0;
				display: flex;
				justify-content: center;
				align-items: center;
				pointer-events: none;
				background-position: center center;
				background-repeat: no-repeat;
				transition: transform 50ms ease-in-out;
				z-index: 2;

				@media (prefers-reduced-motion: reduce) {
					transition: none;
				}
			}
		}

		& > .apple-tv-card-title {
			position: absolute;
			pointer-events: none;
			bottom: 1.3rem;
			left: 0;
			right: 0;
			opacity: 0;
			text-align: center;
			font-size: 1.3rem;
			white-space: nowrap;
			overflow: hidden;
			text-overflow: ellipsis;
			color: white;
			text-shadow: 0 1px 2px black;
			transition:
				opacity 0.12s ease-in-out,
				bottom 0.09s ease-in-out;
			font-family: 'Helvetica Neue', Helvetica, Arial, sans-serif;

			@media (prefers-reduced-motion: reduce) {
				transition: none;
			}

			&.always-visible {
				opacity: 1;
			}
		}
	}
</style>
