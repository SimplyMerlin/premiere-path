<script lang="ts">
	import type { Project } from '$lib/types';

	let { project }: { project: Project } = $props();

	let startsWith = $state('');
	let hideFiltered = $state(false);

	const matchingMedia = $derived(
		project.media.filter((media) => !media.path.startsWith(startsWith) || startsWith === '')
	);

	const filteredOutMedia = $derived(
		project.media.filter((media) => media.path.startsWith(startsWith) && startsWith !== '')
	);

	const matchCount = $derived(matchingMedia.length);
	const totalCount = $derived(project.media.length);

	const displayList = $derived(
		hideFiltered ? matchingMedia : [...matchingMedia, ...filteredOutMedia]
	);
</script>

<div class="flex max-h-full min-h-0 flex-col gap-px bg-neutral-700">
	<div class="flex items-center gap-2 bg-neutral-950 px-2 py-2">
		<input
			type="text"
			bind:value={startsWith}
			class="min-w-0 flex-1 px-1 outline-1 outline-neutral-700"
			placeholder="Filter..."
		/>
		<label class="flex cursor-pointer items-center gap-1 text-sm whitespace-nowrap">
			<input type="checkbox" bind:checked={hideFiltered} />
			Hide
		</label>
		<span class="text-sm whitespace-nowrap text-neutral-400">({matchCount}/{totalCount})</span>
	</div>
	<div class="min-h-0 flex-1 overflow-y-auto bg-neutral-950 px-2">
		<ul>
			{#each displayList as media}
				{@const isFilteredOut = startsWith !== '' && media.path.startsWith(startsWith)}
				<li class={isFilteredOut ? 'text-neutral-600' : ''}>
					<span>{media.path}</span>
					{#if media.usageAmount > 1}
						<span class={isFilteredOut ? 'text-neutral-600' : 'text-neutral-500'}
							>({media.usageAmount})</span
						>
					{/if}
				</li>
			{/each}
		</ul>
	</div>
</div>
