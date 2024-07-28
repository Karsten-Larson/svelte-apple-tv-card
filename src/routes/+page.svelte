<script lang="ts">
	import AppleCard from '$lib/AppleCard.svelte';
</script>

<h1>Welcome to your library project</h1>
<p>Create your package using @sveltejs/package and preview/showcase your work with SvelteKit</p>
<p>Visit <a href="https://kit.svelte.dev">kit.svelte.dev</a> to read the documentation</p>

<div class="container">
	<AppleCard
		backgroundImage={'https://media.istockphoto.com/id/514047994/photo/dark-wall.webp?b=1&s=170667a&w=0&k=20&c=6UN1w6v2YGEkXrFaaxf7BRAJZWmi-Elql0punDoWgtQ='}
	>
		<svelte:fragment slot="nonparallax">
			<p>Hello</p>
		</svelte:fragment>
		<svelte:fragment slot="parallax">
			<div style="width:5em;height:5em;border:1em dashed black;"></div>
		</svelte:fragment>
		<svelte:fragment slot="title">Title</svelte:fragment>
	</AppleCard>
</div>

<style>
	.container {
		width: 800px;
		display: grid;
		place-items: center;
	}
</style>
