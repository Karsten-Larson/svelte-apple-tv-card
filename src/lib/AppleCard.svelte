<script lang="ts">
	import { onMount } from 'svelte';

	export let backgroundImage;

	let container;
	let card;
	let content;

	onMount(() => {
		content.setAttribute('tabindex', '0');
	});

	// Handling Tab Focus
	const handleFocus = (event: MouseEvent) => {
		const width = card.clientWidth;
		const height = card.clientHeight;

		const perspective = Math.max(width, height);

		container.style.perspective = perspective * 2.5 + 'px';
	};

	// Handling Blur
	const handleBlur = (event: MouseEvent) => {
		cleanup(card);
	};

	// Handling Mouse Movement
	const handleStart = (event: MouseEvent) => {
		content.focus();
		card.classList.add('hover');
		handleMove(event);
	};

	const handleMove = (event: MouseEvent) => {
		event.preventDefault(); // TODO: Need workaround for touch devices because this prevents clicking on links

		let element = card;

		if (!element.classList.contains('hover')) {
			return;
		}

		const mediaQuery = window.matchMedia('(prefers-reduced-motion: reduce)');

		if (
			element.classList.contains('with-shadow') &&
			!element.querySelector('.shadow') &&
			mediaQuery &&
			!mediaQuery.matches
		) {
			const shadow = document.createElement('span');
			shadow.classList.add('shadow');
			element.prepend(shadow);
		}

		if (!element.querySelector('.reflection') && mediaQuery && !mediaQuery.matches) {
			const reflection = document.createElement('span');
			reflection.classList.add('reflection');
			element.prepend(reflection);
		}

		let posX = event.offsetX;
		let posY = event.offsetY;

		const width = element.clientWidth;
		const height = element.clientHeight;
		const angleY = ((width / 2 - posX) / width) * 10;
		const angleX = (((height / 2 - posY) * -1) / height) * 10;
		const translateX = (((width / 2 - posX) * -1) / width) * 10;
		const translateY = (((height / 2 - posY) * -1) / height) * 10;

		const perspective = Math.max(width, height);

		container.style.perspective = perspective * 2.5 + 'px';

		if (mediaQuery && !mediaQuery.matches) {
			element.style.transform =
				'translateZ(4rem) rotateY(' +
				angleY +
				'deg) rotateX(' +
				angleX +
				'deg) translateX(' +
				translateX +
				'px) translateY(' +
				translateY +
				'px)';
		}

		element.querySelectorAll('.parallax-content').forEach((parallaxContent, layer) => {
			if (mediaQuery && !mediaQuery.matches) {
				layer++;
				const modifier = !parallaxContent.classList.contains('reverse') ? -0.65 : 0.2;
				parallaxContent.style.transform =
					'scale(1.075) translateX(' +
					translateX * modifier * layer +
					'px) translateY(' +
					translateY * modifier * layer +
					'px)';
			}
		});

		const reflection = element.querySelector('.reflection');

		if (reflection && mediaQuery && !mediaQuery.matches) {
			reflection.style.width = perspective * 1.5 + 'px';
			reflection.style.height = perspective * 1.5 + 'px';
			reflection.style.margin = perspective * -0.75 + 'px 0 0 ' + perspective * -0.75 + 'px';
			reflection.style.transform =
				'translateY(' +
				(posY - height / 2) +
				'px) translateX(' +
				(width * 0.1 + posX * 0.8) +
				'px)';
		}

		const shadow = element.querySelector('.shadow');

		if (shadow) {
			if (mediaQuery && !mediaQuery.matches && posY < height / 3) {
				const opacity = (1 / (height / 3)) * (height / 3 - posY);
				shadow.style.opacity = opacity.toString();
				shadow.style.boxShadow =
					'inset 0 ' + opacity * -1 + 'em .4em -.5em rgba(0,0,0,' + Math.min(opacity, 0.35) + ')';
			} else {
				shadow.style.opacity = null;
				shadow.style.boxShadow = null;
			}
		}
	};

	function handleEnd(event) {
		content.blur();
		cleanup(card);
	}

	// Cleanup the added classes
	const cleanup = (element) => {
		element.classList.remove('hover');

		element.style.transform = null;

		element.querySelectorAll('.parallax-content').forEach((parallaxContent) => {
			parallaxContent.style.transform = null;
		});

		const reflection = element.querySelector('.reflection');

		if (reflection) {
			reflection.style.transform = null;
		}

		const shadow = element.querySelector('.shadow');

		if (shadow) {
			shadow.style.boxShadow = null;
			shadow.style.opacity = null;
		}
	};
</script>

<!-- <svelte:window
	on:resize={() => {
		if (!card) return;

		const size = Math.max(card.clientWidth, card.clientHeight);
		card.style.fontSize = size / 3.5 + 'px';
	}}
/> -->

<div class="apple-tv-card-container" bind:this={container}>
	<div
		role="group"
		class="apple-tv-card"
		class:no-title={!$$slots.title}
		bind:this={card}
		on:mouseenter={handleStart}
		on:mousemove={handleMove}
		on:mouseleave={handleEnd}
	>
		<div
			class="content"
			style="background-image:url({backgroundImage});"
			bind:this={content}
			on:focus={handleFocus}
		>
			<!-- Any non-parallax content -->
			<slot name="nonparallax" />
		</div>
		<div class="parallax-content">
			<!-- Example -->
			<!-- End: Example -->
			<slot name="parallax" />
		</div>
	</div>

	<div class="apple-tv-card-title"><slot name="title" /></div>
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
				background-image: linear-gradient(to bottom, #555, #000);
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
