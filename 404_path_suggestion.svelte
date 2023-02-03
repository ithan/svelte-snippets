<script lang="ts">
	import { page } from '$app/stores';
	const modules = import.meta.glob('./../**/*page.svelte');
  
  function levenshtein_distance(
	a: string,
	b: string
): number {
	const matrix = [];
	// Create a matrix with the length of both strings
	for (let i = 0; i <= a.length; i++) {
		matrix[i] = [i];
	}
	for (let j = 0; j <= b.length; j++) {
		matrix[0][j] = j;
	}
	// Calculate the minimum distance between the two strings
	for (let i = 1; i <= a.length; i++) {
		for (let j = 1; j <= b.length; j++) {
			if (a.charAt(i - 1) === b.charAt(j - 1)) {
				matrix[i][j] = matrix[i - 1][j - 1];
			} else {
				matrix[i][j] = Math.min(
					matrix[i - 1][j - 1] + 1,
					matrix[i][j - 1] + 1,
					matrix[i - 1][j] + 1
				);
			}
		}
	}
	// Return the minimum distance between the two strings
	return matrix[a.length][b.length];
}

  
  

	const pages = Object.keys(modules).map((path) => {
		path = path.split('/').slice(1, -1).join('/');
		const name = path.replace(/^\//, '').replace(/\/$/, '');
		let name_mapped = name === '..' ? 'home' : name;
		return {
			name: `${name_mapped}`,
			path: `/${path}${path ? '/' : ''}`
		};
	});

	const current_page = $page.url.pathname;

	const generate_suggestions = () => {
		const suggestions = [];
		const suggestions_limit = 3;
		for (let i = 0; i < pages.length; i++) {
			const distance = levenshtein_distance(
				pages[i].path,
				current_page
			);
			console.log({
				name: pages[i].name,
				path: pages[i].path,
				distance: distance
			});
			if (distance <= 5) {
				suggestions.push({
					name: pages[i].name,
					path: pages[i].path,
					distance: distance
				});
				if (suggestions.length === suggestions_limit) {
					break;
				}
			}
		}
		suggestions.sort((a, b) => a.distance - b.distance);
		return suggestions.map((suggestion) => {
			return {
				name: suggestion.name,
				path: suggestion.path
			};
		});
	};

	const suggestions = generate_suggestions();
	console.log(suggestions);
</script>

<div
	class="flex h-screen w-full flex-col items-center justify-center bg-primary-800"
>
	<div
		class="h-32 w-32 animate-pulse fill-current text-gray-400 md:h-48 md:w-48 lg:h-32 lg:w-32"
	>
		<!-- logo here -->
	</div>

	<div class="flex flex-col items-center justify-center">
		<p
			class="lg:text-7xl mt-8 text-5xl font-bold tracking-wider text-gray-600 md:text-6xl"
		>
			404
		</p>
		<p
			class="mt-2 text-2xl font-bold text-gray-600 md:text-3xl lg:text-4xl"
		>
			Page not found
		</p>
		<p
			class="mt-2 text-center text-gray-500 md:text-lg xl:text-xl"
		>
			Seems like you tried to access <span
				class="font-bold text-orange-600"
				>{current_page}</span
			>
			which doesn't exist.

		</p>
		<p
			class="mt-2 text-center text-gray-500 md:text-lg xl:text-xl"
		>
			Here are some suggestions for you <br />
			{#each suggestions as suggestion}
				<a
					href={suggestion.path}
					class="mx-4 mt-4 inline-block border-b-2 border-transparent text-green-600 hover:border-primary-500 hover:text-primary-400"
				>
					{suggestion.name}
				</a>
			{/each}
		</p>
	</div>
</div>
