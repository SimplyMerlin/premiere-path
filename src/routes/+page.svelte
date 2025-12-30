<script lang="ts">
	import pako from 'pako';
	import ProjectComponent from '../components/ProjectComponent.svelte';
	import type { Project } from '$lib/types';

	let project = $state<Project | null>(null);
	let fileName = $state<string | null>(null);

	let status = $state('Drag a premiere file into your browser!');

	const processFile = (file: File) => {
		fileName = file.name;
		const reader = new FileReader();
		reader.onload = (ev) => handleFile(ev.target?.result as ArrayBuffer);
		reader.readAsArrayBuffer(file);
	};

	const handleDrop = (e: DragEvent) => {
		e.preventDefault();
		const file = e.dataTransfer?.files[0];
		if (file) {
			processFile(file);
		}
	};

	const handleFile = (arrayBuffer: ArrayBuffer) => {
		const decompressed = pako.ungzip(arrayBuffer, { to: 'string' });
		console.log(decompressed);
		const parsed = new DOMParser().parseFromString(decompressed, 'application/xml');
		createProject(parsed);
		for (const media of parsed.querySelectorAll('Media')) {
			if (!media.querySelector('RelativePath')) {
				continue;
			}
			console.log(media.querySelector('ActualMediaFilePath')?.textContent);
		}
		status = 'Yippie!';
	};

	const createProject = (xmlDocument: Document) => {
		const name = '?';
		const path = '?';

		let media: Project['media'] = [];

		for (const mediaElement of xmlDocument.querySelectorAll('Media')) {
			const relativePath = mediaElement.querySelector('RelativePath');
			if (!relativePath) {
				continue;
			}
			const path = mediaElement.querySelector('ActualMediaFilePath')!.textContent;
			media.push({
				path: path,
				relativePath: relativePath.textContent
			});
		}

		project = {
			name: name,
			path: path,
			media: media
		};
	};
</script>

<!-- <div class="m-4 p-4">
	<input type="file" onchange={handleUpload} />
</div>
{#if project}
	<div class="m-4 h-full overflow-scroll p-4">
		<ProjectComponent {project} />
	</div>
{/if} -->

<div
	class="flex h-screen flex-col gap-px bg-neutral-700"
	role="application"
	onwheel={(e) => e.stopPropagation()}
	ondragover={(e) => e.preventDefault()}
	ondrop={handleDrop}
>
	<div class="flex justify-center bg-neutral-950">
		<div class="py-4">{status}</div>
	</div>
	<div class="grid min-h-0 grow grid-cols-2 gap-px">
		<div class="flex min-h-0 flex-col bg-neutral-950 p-2">
			{#if project}
				<ProjectComponent {project} />
			{/if}
		</div>
		<div class="min-h-0 bg-neutral-950"></div>
	</div>
</div>
