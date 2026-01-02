<script lang="ts">
	import type { Project } from '$lib/types';

	export let project: Project;
</script>

<div class="max-h-full overflow-scroll">
	<ul>
		{#each project.media as media}
			<li>
				<span>{media.path}</span>
				{#if media.usageAmount > 1}
					<span class="text-neutral-500">({media.usageAmount})</span>
				{/if}
			</li>
		{/each}
	</ul>
</div>
