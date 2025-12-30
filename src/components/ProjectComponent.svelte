<script lang="ts">
	import type { Project } from '$lib/types';

	export let project: Project;
</script>

<div class="max-h-full overflow-scroll">
	<ul>
		{#each project.media as media}
			<li>{media.path}</li>
		{/each}
	</ul>
</div>
