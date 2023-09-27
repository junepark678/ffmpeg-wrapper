<script lang="ts">
	import "@/app.css";
	import Output from "./lib/Output.svelte";
	import { FFmpeg } from "@ffmpeg/ffmpeg";
	import { toBlobURL, fetchFile } from "@ffmpeg/util";
	import { onDestroy } from "svelte";
	// import mime from "mime-types";

	const baseURL = "https://unpkg.com/@ffmpeg/core-mt@0.12.2/dist/esm";
	const ffmpeg = new FFmpeg();

	$: output_blob = "";
	$: output = false;
	$: userMessage = "";
	// @ts-ignore
	let output_div;
	let output_component: Output;

	let input: HTMLInputElement;
	let cancelButton: HTMLButtonElement;
	let convertBtn: HTMLButtonElement;
	function reset() {
		output_blob = "";
		output = false;
		userMessage = "";
		input.disabled = false;
		cancelButton.disabled = true;
		convertBtn.disabled = false;
	}

	$: output_type = "gif";
	$: quality = "medium";
	$: color_filter = "none";
	$: output_blob_type = "application/octet-stream";

	$: output_blob_type, recalculateType();
	$: output_blob, console.log(output_blob);

	function recalculateType() {
		if (output_type === "gif") {
			output_blob_type = "image/gif";
		} else if (output_type === "mp4") {
			output_blob_type = "video/mp4";
		} else if (output_type === "webm") {
			output_blob_type = "video/webm";
		}
	}

	ffmpeg.on("log", ({ message }) => {
		userMessage = message;
		console.log(message);
	});

	async function cancel() {
		ffmpeg.terminate();
		input.disabled = false;
		convertBtn.disabled = false;
		cancelButton.disabled = false;
		reset();
	}

	async function convert() {
		try {
			cancelButton.disabled = false;
			output_blob = "";
			let tmpvideo = document.getElementById("video-upload");
			if (tmpvideo === null) {
				input.disabled = false;
				convertBtn.disabled = false;
				cancelButton.disabled = true;
				reset();
				return;
			}
			const video = tmpvideo as HTMLInputElement;
			if (video.files === null) {
				convertBtn.disabled = false;
				input.disabled = false;
				cancelButton.disabled = true;
				reset();
				return;
			}
			if (video.files.length === 0) {
				input.disabled = false;
				convertBtn.disabled = false;
				cancelButton.disabled = true;
				reset();
				return;
			}
			video.disabled = true;
			const name = video.files[0].name.split(".")[0];
			const type = video.files[0].type.split("/")[1];
			const output2 = `${name}.gif`;
			const progress = document.getElementById(
				"progress"
			) as HTMLDivElement;
			const convert = document.getElementById(
				"convert"
			) as HTMLButtonElement;
			if (convert === null) {
				convertBtn.disabled = false;
				input.disabled = false;
				cancelButton.disabled = true;
				reset();
				return;
			}
			convert.disabled = true;
			if (progress === null) {
				convertBtn.disabled = false;
				input.disabled = false;
				cancelButton.disabled = true;
				reset();

				return;
			}
			progress.innerHTML = "Loading ffmpeg-core.js";
			await ffmpeg.load({
				coreURL: await toBlobURL(
					`${baseURL}/ffmpeg-core.js`,
					"text/javascript"
				),
				wasmURL: await toBlobURL(
					`${baseURL}/ffmpeg-core.wasm`,
					"applicaiton/wasm"
				),
				workerURL: await toBlobURL(
					`${baseURL}/ffmpeg-core.worker.js`,
					"text/javascript"
				),
			});

			let videoURL = new Blob([video.files[0]], {
				type: video.files[0].type,
			});

			progress.innerHTML = "Start converting";
			const fileName = video.files[0].name;
			await ffmpeg.writeFile(fileName, await fetchFile(videoURL));

			let params = ["-t", "00:00:03", "-i", fileName, "-preset", quality];

			if (color_filter === "vivid") {
				params = params.concat(["-vf", "hue=h=19:s=3:b=0.8"]);
			}
			if (color_filter === "monochrome") {
				params = params.concat(["-vf", "format=gray"]);
			}
			if (color_filter === "vintage") {
				params = params.concat(["-vf", "hue=h=-10:s=-2.3:b=-0.5"]);
			}

			params = params.concat([`test.${output_type}`]);

			await ffmpeg.exec(params);

			/*vivid: 			"-vf",
			"hue=h=19:s=3:b=0.8",*/

			progress.innerHTML = "Complete";
			convert.disabled = false;
			const data = await ffmpeg.readFile(`test.${output_type}`);

			if (output_type === "gif") {
				output_blob_type = "image/gif";
			} else if (output_type === "mp4") {
				output_blob_type = "video/mp4";
			} else if (output_type === "webm") {
				output_blob_type = "video/webm";
			}

			output_blob = URL.createObjectURL(
				new Blob([(data as Uint8Array).buffer], {
					type: /*mime.lookup(output_type ||*/ output_blob_type,
				})
			);

			console.log(output_blob);

			input.disabled = false;
			convertBtn.disabled = false;
			cancelButton.disabled = true;

			// output_image.src = output_blob;
			// output = true;
		} catch {
			input.disabled = false;
			convertBtn.disabled = false;
			cancelButton.disabled = true;
			reset();
		}
		// output_component.$destroy();
		// new Output({
		// 	// @ts-ignore
		// 	target: output_div,
		// 	props: {
		// 		output_blob: output_blob,
		// 		output_blob_type: output_blob_type,
		// 	},
		// });
	}
</script>

<div
	class="flex flex-col align-middle h-screen w-screen items-center justify-center"
>
	<div class="">
		<h1 class="text-5xl block pb-4">Video Converter</h1>
		<input
			type="file"
			name="video-upload"
			id="video-upload"
			on:change={reset}
			bind:this={input}
		/>
		<select
			name="quality"
			id="quality"
			bind:value={quality}
			on:change={reset}
		>
			<option value="low">Low</option>
			<option value="medium">Medium</option>
			<option value="high">High</option>
		</select>
		<select
			name="output_type"
			id="output_type"
			bind:value={output_type}
			on:change={reset}
		>
			<option value="gif">GIF</option>
			<option value="mp4">MP4</option>
			<option value="webm">WEBM</option>
		</select>
		<select
			name="colorFilter"
			id="colorFilter"
			bind:value={color_filter}
			on:change={reset}
		>
			<option value="none">None</option>
			<option value="vintage">Vintage</option>
			<option value="vivid">Vivid</option>
			<option value="monochrome">Monochrome</option>
		</select>
		<button
			id="convert"
			on:click={convert}
			class="border border-solid border-black border-3 p-2 ml-2 mr-0"
			bind:this={convertBtn}>Convert</button
		>
		{#if !output_blob}
			<button
				on:click={cancel}
				disabled={true}
				bind:this={cancelButton}
				class="border border-solid border-black border-3 p-2 m-0.5"
				>Cancel</button
			>
		{/if}
		<div id="progress" />
		{#key output_blob}
		<div bind:this={output_div}>
			<Output
				{output_blob}
				{output_blob_type}
				bind:this={output_component}
			/>
		</div>
		{/key}
	</div>
	{#if !output_blob}
		<div>{userMessage}</div>
	{/if}
</div>
