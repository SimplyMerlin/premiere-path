<script lang="ts">
	import pako from 'pako';
	import ProjectComponent from '../components/ProjectComponent.svelte';
	import type { Project } from '$lib/types';

	let project = $state<Project | null>(null);
	let fileName = $state<string | null>(null);

	let status = $state('Waiting for your premiere pro project...');

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
		status = 'Loaded! (you can drag another one in btw if you want)';
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
			const alreadyAdded = media.find((m) => m.path === path);
			if (alreadyAdded) {
				alreadyAdded.usageAmount++;
				continue;
			}
			media.push({
				path: path,
				relativePath: relativePath.textContent,
				usageAmount: 1
			});
		}

		project = {
			name: name,
			path: path,
			media: media
		};
	};
</script>

<div
	class="flex h-screen flex-col gap-px bg-neutral-700"
	role="application"
	onwheel={(e) => e.stopPropagation()}
	ondragover={(e) => e.preventDefault()}
	ondrop={handleDrop}
>
	<div class="flex items-center justify-between bg-neutral-950 px-8">
		<div class="py-4">{status}</div>
		<a
			class="text-neutral-400 underline decoration-dashed hover:text-neutral-200"
			href="https://github.com/simplymerlin/premiere-path"
			target="_blank"
			rel="noopener noreferrer">github</a
		>
	</div>
	<div class="flex min-h-0 flex-1 bg-neutral-950">
		<div class="mx-auto flex min-h-0 w-7xl flex-col bg-neutral-950 outline-1 outline-neutral-700">
			{#if project}
				<ProjectComponent {project} />
			{:else}
				<div class="flex h-full flex-col items-center justify-center gap-1">
					<div>Drag a .prproj file into your browser to see where all the media is located!</div>
					<div class="text-neutral-400">(fully local in your browser btw)</div>
				</div>
			{/if}
		</div>
	</div>
</div>
