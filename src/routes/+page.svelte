<script lang="ts">
	import { FileDropzone } from '@skeletonlabs/skeleton';
	import * as streamSaver from 'streamsaver'

	let files: FileList | undefined = undefined;

	async function concatenateFiles() {
		if (!files || files.length === 0) {
			alert("No files selected.");
			return;
		}

		const totalFileSize = Array.from(files).reduce((acc, file) => acc + file.size, 0);

		// Create a writable stream for the concatenated file
		const fileName = 'concatenated'; // Change to desired output file name
		// get the extension of the first file
		const extension = files[0].name.split('.').pop() || 'txt';

		const writableStream = streamSaver.createWriteStream(`${fileName}.${extension}`, {
          size: totalFileSize // Makes the percentage visible in the download
        });
		const writer = writableStream.getWriter();

		// Loop through each file and write its contents to the writable stream
		for (let i = 0; i < files.length; i++) {
			const file = files[i];
			const reader = file.stream().getReader();

			// Read the file content in chunks
			while (true) {
				const { value, done } = await reader.read();
				if (done) break;
				await writer.write(value); // Write the chunk to the writable stream
			}

			// Add a newline or separator between files if needed
			if (i < files.length - 1) {
				const separator = new TextEncoder().encode('\n');
				await writer.write(separator);
			}
		}

		// Close the writer to signal that the stream is complete
		writer.close();
	}
</script>

<div class="container h-full w-full mx-auto flex justify-center items-center">
	<div class="space-y-5">
		<div class="flex justify-center items-center space-x-2 align-middle">
			<span class="text-4xl sm:text-8xl">📄📄📄</span> 
			<span class="text-2xl sm:text-5xl">➡️</span> 
			<span class="text-4xl sm:text-8xl">📄</span>
		</div>
		<div class="text-center">
			<p>Concatenate multiple files into one</p>
		</div>
		<FileDropzone name="files" multiple bind:files={files} on:change={concatenateFiles}>
			<svelte:fragment slot="message">
				<p class="text-lg">
					<span class="font-semibold">Load files</span> or drag and drop
				</p>
			</svelte:fragment>
		</FileDropzone>
	</div>
	<div class="absolute bottom-0 p-2 text-xs text-gray-500">
		Your files never leave your computer
	</div>
</div>
