<script lang="ts">
	import pako from 'pako';

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
		for (const media of parsed.querySelectorAll('Media')) {
			if (!media.querySelector('RelativePath')) {
				continue;
			}
			console.log(media.querySelector('ActualMediaFilePath')?.textContent);
		}
	};
</script>

<input type="file" onchange={handleUpload} />
