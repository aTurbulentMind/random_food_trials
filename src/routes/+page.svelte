<script lang="ts">
	import type { PageData } from './$types';

	export let data: PageData;

	const { posts } = data;

	$: console.log(posts);

	function formatDate(date: Date) {
		return new Intl.DateTimeFormat('en', { dateStyle: 'long' }).format(date);
	}

	function getInstructions() {
		if (!posts.results || posts.results.length === 0) {
			return [];
		}

		const instructions = [];

		posts.results.forEach((result) => {
			if (result.recipes && result.recipes.length > 0) {
				result.recipes.forEach((recipe) => {
					const recipeTitle = recipe.name;
					const recipeInstructions = recipe.instructions || [];
					instructions.push({ title: recipeTitle, instructions: recipeInstructions });
				});
			} else if (result.sections && result.sections.length > 0) {
				const recipeTitle = result.name;
				const recipeInstructions = result.sections
					.flatMap((section) => section.components)
					.filter((component) => component.display_text);
				instructions.push({ title: recipeTitle, instructions: recipeInstructions });
			}
		});

		return instructions;
	}

	function getComponents() {
		if (!posts.results || posts.results.length === 0) {
			return [];
		}

		// flatten the components arrays for single recipes
		const singleComponents = posts.results.flatMap((post) => {
			if (post.components && post.components.length > 0) {
				return post.components;
			} else if (post.recipes && post.recipes.length > 0) {
				return post.recipes.flatMap((recipe) => recipe.components || []);
			} else {
				return [];
			}
		});

		// flatten the components arrays for compilation groups
		const compilationComponents = posts.results.flatMap((post) => {
			if (post.compilations && post.compilations.length > 0) {
				return post.compilations.flatMap((compilation) => {
					return compilation.components || [];
				});
			} else {
				return [];
			}
		});

		// combine the flattened arrays
		return [...singleComponents, ...compilationComponents].filter(
			(component) => component.raw_text
		);
	}
</script>

<body>
	<hgroup>
		<!--Entry phrase: contains recipe info, description and amount of servings-->

		<h1>Today is {formatDate(posts.results[0].createdAt)}</h1>
		<h2>Here is a random recipe to try:</h2>
		<h3>{posts.results[0].name}</h3>
		<h4>{posts.results[0].description}</h4>
	</hgroup>

	<h2>Ingredients</h2>

	{#if posts.results && posts.results.length > 0}
		<ul>
			{#each posts.results as result}
				{#if result.recipes && result.recipes.length > 0}
					{#each result.recipes as recipe}
						<li>
							<details>
								<summary>{recipe.name}</summary>
								{#each recipe.sections as section}
									<ul class="highlight">
										{#each section.components as component}
											<li>{component.raw_text}</li>
										{/each}
									</ul>
								{/each}
							</details>
						</li>
					{/each}
				{:else}
					{#each result.sections as section}
						<ul>
							{#each section.components as component}
								<li>{component.raw_text}</li>
							{/each}
						</ul>
					{/each}
				{/if}
			{/each}
		</ul>
	{:else}
		<p>No data available</p>
	{/if}

	<h2>Instructions</h2>

	{#if posts.results && posts.results.length > 0}
		<ul>
			{#each getInstructions() as { title, instructions }}
				<li>
					<details>
						<summary><strong>{title}</strong></summary>
						<ul>
							{#each instructions as instruction}
								<li>{instruction.display_text}</li>
							{/each}
						</ul>
					</details>
				</li>
			{/each}
		</ul>
	{:else}
		<p>No data available</p>
	{/if}
</body>

<style lang="scss">
	* {
		margin: 0;
		padding: 0;
		overflow-x: hidden;
	}

	body {
		background-size: 100%;
		background-color: #0a0214;
		color: #fdfdf8;
		letter-spacing: 2px;
		line-height: 1.5;
		margin: -8px;
		font-family: Arial, Helvetica, sans-serif;
	}

	h1,
	h2,
	h3,
	h4,
	p,
	li {
		margin: 1% 20%;
	}

	ul {
		&:hover {
			cursor: pointer;
		}
	}
	.highlight {
		background-color: #201131;
	}

	p {
		font-size: 1.1rem;
	}
</style>