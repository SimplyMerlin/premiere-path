<script lang="ts">
	import pako from 'pako';
	import ProjectComponent from '../components/ProjectComponent.svelte';
	import type { Project } from '$lib/types';

	let project = $state<Project | null>(null);

	const handleUpload = (e: Event) => {
		const file = (e.target as HTMLInputElement).files?.[0];
		if (file) {
			const reader = new FileReader();
			reader.onload = (ev) => handleFile(ev.target?.result as ArrayBuffer);
			reader.readAsArrayBuffer(file);
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

<input type="file" onchange={handleUpload} />
{#if project}
	<ProjectComponent {project} />
{/if}
