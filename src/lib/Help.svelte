<script lang="ts">
	import { getModalStore } from '@skeletonlabs/skeleton';

	const modalStore = getModalStore();
</script>

<div class="card w-modal shadow-xl space-y-4">
<div class="max-h-dvh overflow-y-auto p-4 pb-8">
	<div class="flex justify-between">
		<h2 class="h2">What Does This App Do?</h2>
		<button class="text-surface-500 hover:text-surface-400" on:click={() => modalStore.close()}>
			<svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="2" stroke="currentColor" class="size-6">
				<path stroke-linecap="round" stroke-linejoin="round" d="M6 18 18 6M6 6l12 12" />
			</svg>				  
		</button>
	</div>
<p class="mt-4">Replace is a client-side webapp that allows you to perform multiple find and replace operations on text files. Using a TSV or CSV file as a map, you can define search keys and replacement values to update text in one or more files.</p>

<hr class="my-6 border-gray-300" />
<h2 class="text-2xl font-semibold ">How to Use the Replace App</h2>
<ol class="list-decimal list-inside mt-4 space-y-4">
  <li>
    <strong>Load Your Map File:</strong>
    <ul class="list-disc list-inside ml-5">
      <li>Drag and drop a CSV or TSV file into the designated area or use the file picker.</li>
      <li>The app will parse your file and display the columns available for selection.</li>
    </ul>
  </li>
  <li>
    <strong>Select an Index Column:</strong>
    <p>Choose the column that contains the terms you want to find (the search keys).</p>
  </li>
  <li>
    <strong>Define Replacement Logic:</strong>
    <p>Use the Replace Bar to specify how text should be replaced. Dynamic replacements can be created using placeholders from your map file columns.</p>
  </li>
  <li>
    <strong>Upload Files to Process:</strong>
    <p>Add the files you want to modify by dropping them into the app or selecting them manually.</p>
  </li>
  <li>
    <strong>Apply Replacements:</strong>
    <p>For each row in your map file a replacement will be performed. The app will process each file line by line, applying your replacement logic efficiently.</p>
  </li>
  <li>
    <strong>Download Updated Files:</strong>
    <p>Once processed, the app saves the updated files directly to your system.</p>
  </li>
</ol>
<hr class="my-6 border-gray-300" />
<h2 class="text-2xl font-semibold ">Found a bug?</h2>
<p class="mt-4 ">Please report any issues or suggestions on the <a href="https://github.com/Wytamma/replace" target="_blank" class="text-primary-500 hover:underline">GitHub repository</a>.</p>
</div>
</div>