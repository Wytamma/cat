<script lang="ts">
	import { FileDropzone } from '@skeletonlabs/skeleton';
	import { Accordion, AccordionItem } from '@skeletonlabs/skeleton';

	import * as streamSaver from 'streamsaver';

	let files: FileList | undefined = undefined;
	let regexInput: string = ''; // Holds the user-provided regex string
	let defaultFilename: string = 'concatenated'; // Default filename for the concatenated file
	let useFistFileExtension: boolean = true; // Use the extension from the first file

	// if useFistFileExtension changes, reset the defaultFilename
	$: if (useFistFileExtension === false && defaultFilename === 'concatenated') {
		defaultFilename = 'concatenated.txt';
	} else if (useFistFileExtension === true && defaultFilename === 'concatenated.txt') {
		defaultFilename = 'concatenated';
	}

	async function concatenateFiles() {
		if (!files || files.length === 0) {
			alert("No files selected.");
			return;
		}

		let regex: RegExp | null = null;
		try {
			regex = regexInput ? new RegExp(regexInput) : null;
		} catch (error) {
			alert("Invalid regex. Please correct it and try again.");
			return;
		}

		// Group files based on regex
		const fileGroups: Record<string, File[]> = {};
		for (const file of Array.from(files)) {
			let key = defaultFilename; // Default group key
			if (regex) {
				const match = file.name.match(regex);
				if (match) {
					key = match[0]; // Use the matched part as the group key
				}
			}
			if (!fileGroups[key]) fileGroups[key] = [];
			fileGroups[key].push(file);
		}

		// Process each group and concatenate files
		for (const [group, groupedFiles] of Object.entries(fileGroups)) {
			const totalFileSize = groupedFiles.reduce((acc, file) => acc + file.size, 0);

			// Create a writable stream for the concatenated file
			const fileName = `${group}`; // Group-based output file name
			let extension = ''; // Default extension
			if (useFistFileExtension) {
				extension = groupedFiles[0].name.split('.').pop() || ''; // Use the extension from the first file
				if (extension) extension = `.${extension}`;
			}
			const writableStream = streamSaver.createWriteStream(`${fileName}${extension}`, {
				size: totalFileSize,
			});
			const writer = writableStream.getWriter();

			// Loop through each file and write its contents to the writable stream
			for (let i = 0; i < groupedFiles.length; i++) {
				const file = groupedFiles[i];
				const reader = file.stream().getReader();

				// Read the file content in chunks
				while (true) {
					const { value, done } = await reader.read();
					if (done) break;
					await writer.write(value); // Write the chunk to the writable stream
				}

				// Add a newline or separator between files if needed
				if (i < groupedFiles.length - 1) {
					const separator = new TextEncoder().encode('\n');
					await writer.write(separator);
				}
			}

			// Close the writer to signal that the stream is complete
			writer.close();
		}
	}
</script>

<div class="container h-full w-full mx-auto flex justify-center items-center">
	<div class="space-y-5">
		<div class="flex justify-center items-center space-x-2 align-middle">
			<span class="text-5xl sm:text-8xl">📄📄📄</span> 
			<span class="text-3xl sm:text-5xl">➡️</span> 
			<span class="text-5xl sm:text-8xl">📄</span>
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
		
		<Accordion>
			<AccordionItem>
				<svelte:fragment slot="summary">
					<div class="text-gray-400">
						Advanced
					</div>
				</svelte:fragment>
				<svelte:fragment slot="iconClosed">
					<div class="text-gray-400 fill-current w-3 transition-transform duration-[200ms]  rotate-180">
						<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 448 512">
							<path d="M201.4 374.6c12.5 12.5 32.8 12.5 45.3 0l160-160c12.5-12.5 12.5-32.8 0-45.3s-32.8-12.5-45.3 0L224 306.7 86.6 169.4c-12.5-12.5-32.8-12.5-45.3 0s-12.5 32.8 0 45.3l160 160z"></path>
						</svg>
					</div>
				</svelte:fragment>
                <svelte:fragment slot="iconOpen">
					<div class="text-gray-400 fill-current w-3 transition-transform duration-[200ms] ">
						<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 448 512">
							<path d="M201.4 374.6c12.5 12.5 32.8 12.5 45.3 0l160-160c12.5-12.5 12.5-32.8 0-45.3s-32.8-12.5-45.3 0L224 306.7 86.6 169.4c-12.5-12.5-32.8-12.5-45.3 0s-12.5 32.8 0 45.3l160 160z"></path>
						</svg>
					</div>
				</svelte:fragment>
				<svelte:fragment slot="content">
					<label class="flex items-center space-x-2">
						<input class="checkbox" type="checkbox" bind:checked={useFistFileExtension} />
						<p>Use extension from first file</p>
					</label>
					<label class="label">
						<span>Default filename</span>
						<input class="input p-2" type="text" placeholder="Enter an extension" bind:value={defaultFilename} />
					</label>
					<label class="label">
						<span>Group by Regex</span>
						<input class="input p-2" type="text" placeholder="Enter a regex" bind:value={regexInput} />
					</label>
					
					<div class="text-center text-sm text-gray-400">
						Advanced options must be set before loading files
					</div>
				</svelte:fragment>
			</AccordionItem>
		</Accordion>
	</div>
	<div class="absolute bottom-0 p-2 text-xs text-gray-500">
		Your files never leave your computer
	</div>
</div>
