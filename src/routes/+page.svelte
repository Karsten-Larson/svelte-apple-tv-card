<script lang="ts">
	import AppleCard from '$lib/AppleCard.svelte';
</script>

<div class="title">
	<h1>Svelte Apple TV Card</h1>
	<h3>Hover above to try it!</h3>
	<p>
		Visit the <a href="https://github.com/Karsten-Larson/svelte-apple-tv-card">Github Repo</a> to view
		source code
	</p>
</div>

<div class="container">
	<AppleCard
		rounded={false}
		--background-image="url('https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcSeqXt58PjzX23twDkHZHBbpH72I98IbEK12w&s')"
	>
		<svelte:fragment slot="nonparallax"></svelte:fragment>
		<svelte:fragment slot="parallax">
			<div style="display: block;">
				<div style="width:5em;height:5em;border:1em dashed white;"></div>
				<p style="color: white; text-align: center;">Title page</p>
			</div>
		</svelte:fragment>
		<svelte:fragment slot="title">Title</svelte:fragment>
	</AppleCard>

	<AppleCard --background-image="linear-gradient(to bottom, #555, #000)">
		<svelte:fragment slot="nonparallax"></svelte:fragment>
		<svelte:fragment slot="parallax">
			<img
				src="https://cdn-icons-png.flaticon.com/512/25/25231.png"
				alt="Github"
				style="width: 200px; height:200px;"
			/>
		</svelte:fragment>
		<svelte:fragment slot="title">Fancy Title banner</svelte:fragment>
	</AppleCard>
</div>

<style>
	.title {
		margin-top: 80px;
		margin-bottom: 30px;
		text-align: center;
	}

	.container {
		margin: auto;
		width: 800px;
		display: grid;
		place-items: center;
	}
</style>
