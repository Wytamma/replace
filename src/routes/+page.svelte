<script lang="ts">
	import { FileDropzone } from '@skeletonlabs/skeleton';
	
	import Papa from 'papaparse';
	import * as streamSaver from 'streamsaver';



	import ReplaceBar from "$lib/ReplaceBar.svelte";
	import IndexSelect from "$lib/IndexSelect.svelte";
	
	let mapFile: FileList | undefined;
	let replaceFiles: FileList | undefined;
	
	let index: string;
	let replaceDiv: HTMLElement | null = null;
	let columns: string[] | undefined;
	let mapData: any[] = [];

	async function handleFileChange() {
		const file: File | undefined = mapFile?.[0];
		if (!file) return;

		// Read the file using FileReader
		const reader = new FileReader();

		reader.onload = (event) => {
			if (!event.target?.result) return;

			// Use PapaParse to parse the CSV data
			Papa.parse<{}>(event.target.result as string, {
			header: true, // Automatically use the first row as column headers
			skipEmptyLines: true, // Skip empty lines in the CSV
			complete: (results) => {
				console.log('Parsed data:', results.data);

				// Extract column headers from the first row
				if (results.data.length > 0) {
					columns = Object.keys(results.data[0])
				}

				console.log('Columns:', columns);
				mapData = results.data;
			},
			error: (error) => {
					console.error('Error parsing CSV:', error);
					columns = [];
				},
			});
	};

	reader.onerror = (error) => {
		console.error('Error reading file:', error);
	};

	// Read the file as text
	reader.readAsText(file);
	}

	function createReplaceText(nodes: NodeListOf<ChildNode>, row: { [key: string]: string }) {
		let replaceText = '';
		for (let i = 0; i< nodes.length; i++) {
			const node = nodes[i];
			if (node instanceof HTMLElement) {
				replaceText += row[node.innerHTML];
			} else if (node instanceof Text && node.textContent) {
				replaceText += node.textContent;
			}
		}
		return replaceText;
	}
	// Escaping regex special characters
	function escapeRegex(text: string): string {
		return text.replace(/[.*+?^${}()|[\]\\]/g, '\\$&');
	}

	async function handleReplaceFileChange() {
	console.log(replaceFiles);
	if (!replaceFiles) return;
	if (!replaceDiv || replaceDiv === null) return;

	// Create replacements mapping
	const replacements: Record<string, string> = Object.fromEntries(
		mapData.map(row => {
			if (!replaceDiv) throw new Error('replaceDiv is not defined');
			const replace = createReplaceText(replaceDiv.childNodes, row);
			const find = row[index];
			if (!find) throw new Error('Index column not found');
			return [find, replace];
		})
	);

	// Compile the regex for find-replace
	const pattern = new RegExp(Object.keys(replacements).map(find => escapeRegex(find)).join('|'), 'g');

	console.log(replacements);
	console.log(pattern);

	// Process each file
	for (let i = 0; i < replaceFiles.length; i++) {
		const file = replaceFiles[i];
		const writableStream = streamSaver.createWriteStream(file.name, { size: file.size });
		const writer = writableStream.getWriter();
		const reader = file.stream().getReader();
		const decoder = new TextDecoder(); // Decode binary chunks to string
		const encoder = new TextEncoder(); // Encode processed text back to binary

		let leftover = ""; // Buffer for accumulating chunks until newline

		try {
			while (true) {
				const { value, done } = await reader.read();
				if (done) break;

				// Decode current chunk and append leftover
				const chunkWithLeftover = leftover + decoder.decode(value, { stream: true });

				// Split into lines, preserving the last incomplete line in `leftover`
				const lines = chunkWithLeftover.split("\n");
				leftover = lines.pop() || ""; // Last line becomes leftover if incomplete

				const chunk = lines.join("\n"); // Join lines back together
				
				const processedLine = chunk.replace(pattern, match => replacements[match]);
				const encodedLine = encoder.encode(processedLine + "\n"); // Add newline back
				await writer.write(encodedLine);
			}

			// Process any remaining leftover text as a final line
			if (leftover) {
				const processedLeftover = leftover.replace(pattern, match => replacements[match]);
				const finalEncodedBytes = encoder.encode(processedLeftover);
				await writer.write(finalEncodedBytes);
			}
		} finally {
			reader.releaseLock();
			writer.close();
		}
	}
}


	
</script>
<div class="h-full flex justify-center items-center mx-2">
	<div class="max-w-2xl w-full">
	{#if columns === undefined}
		<div class="flex items-center w-full ">
			<FileDropzone name="files" accept=".tsv,.csv" on:change={handleFileChange} bind:files={mapFile}>
				<svelte:fragment slot="lead">
					<div class="flex justify-center">
						<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" fill="currentColor" class="size-16">
							<path fill-rule="evenodd" d="M5.625 1.5H9a3.75 3.75 0 0 1 3.75 3.75v1.875c0 1.036.84 1.875 1.875 1.875H16.5a3.75 3.75 0 0 1 3.75 3.75v7.875c0 1.035-.84 1.875-1.875 1.875H5.625a1.875 1.875 0 0 1-1.875-1.875V3.375c0-1.036.84-1.875 1.875-1.875Zm6.905 9.97a.75.75 0 0 0-1.06 0l-3 3a.75.75 0 1 0 1.06 1.06l1.72-1.72V18a.75.75 0 0 0 1.5 0v-4.19l1.72 1.72a.75.75 0 1 0 1.06-1.06l-3-3Z" clip-rule="evenodd" />
							<path d="M14.25 5.25a5.23 5.23 0 0 0-1.279-3.434 9.768 9.768 0 0 1 6.963 6.963A5.23 5.23 0 0 0 16.5 7.5h-1.875a.375.375 0 0 1-.375-.375V5.25Z" />
						  </svg>	
					</div>
				</svelte:fragment>
				<svelte:fragment slot="message">
					<p class="text-lg">
						<span class="font-semibold">Load a file</span> or drag and drop
					</p>
				</svelte:fragment>
				<svelte:fragment slot="meta">TSV & CSV allowed.</svelte:fragment>
			</FileDropzone>
		</div>
	{:else}	
		<div class="space-y-5">
			<div class="flex justify-end items-end">
				<button class="text-gray-500 hover:text-primary-500" on:click={() => columns = undefined}>
					<svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="2" stroke="currentColor" class="size-6">
						<path stroke-linecap="round" stroke-linejoin="round" d="M6 18 18 6M6 6l12 12" />
					</svg>				  
				</button>
			</div>
				<FileDropzone name="files" on:change={handleReplaceFileChange} bind:files={replaceFiles}>
					<svelte:fragment slot="lead">
						<div class="flex justify-center">
							<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" fill="currentColor" class="size-16">
								<path fill-rule="evenodd" d="M5.625 1.5c-1.036 0-1.875.84-1.875 1.875v17.25c0 1.035.84 1.875 1.875 1.875h12.75c1.035 0 1.875-.84 1.875-1.875V12.75A3.75 3.75 0 0 0 16.5 9h-1.875a1.875 1.875 0 0 1-1.875-1.875V5.25A3.75 3.75 0 0 0 9 1.5H5.625ZM7.5 15a.75.75 0 0 1 .75-.75h7.5a.75.75 0 0 1 0 1.5h-7.5A.75.75 0 0 1 7.5 15Zm.75 2.25a.75.75 0 0 0 0 1.5H12a.75.75 0 0 0 0-1.5H8.25Z" clip-rule="evenodd" />
								<path d="M12.971 1.816A5.23 5.23 0 0 1 14.25 5.25v1.875c0 .207.168.375.375.375H16.5a5.23 5.23 0 0 1 3.434 1.279 9.768 9.768 0 0 0-6.963-6.963Z" />
							</svg>
						</div>
					</svelte:fragment> 
					<svelte:fragment slot="message">
						<p class="text-lg">
							<span class="font-semibold">Add files</span> to replace
						</p>
					</svelte:fragment>
				</FileDropzone>
			<IndexSelect columns={columns} bind:value={index}/>
			<div class="flex space-x-5">
				<ReplaceBar columns={columns} bind:contenteditableDiv={replaceDiv}/>
			</div>
		</div>
	{/if}
</div>
<div class="absolute bottom-0 p-2 text-xs text-gray-500">
	Your files never leave your computer
</div>
</div>

