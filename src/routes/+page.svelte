<script lang="ts">
	import type { ProcessedPost, LinkObject } from '$lib/types';
	import { processLinks } from '$lib/wordpressUtils';
	import { parseCsv } from '$lib/wordpressUtils';

	let email = $state('');
	let password = $state('');
	let file = $state<File | null>(null);
	let fileName = $state('');
	let result = $state<ProcessedPost[] | null>(null);
	let isLoading = $state(false);

	function handleFileChange(event: Event) {
		const target = event.target as HTMLInputElement;
		if (target.files && target.files.length > 0) {
			file = target.files[0];
			fileName = file.name;
		} else {
			file = null;
			fileName = '';
		}
	}

	async function handleSubmit(event: Event) {
		event.preventDefault();

		if (!file || !email || !password) {
			alert('Please fill in all fields');
			return;
		}

		isLoading = true;
		try {
			const links = await parseCsv(file);
			result = await processLinks(links, email, password);
		} catch (error) {
			console.error('Error processing links:', error);
			alert('An error occurred while processing the links. Please check the console for details.');
		} finally {
			isLoading = false;
		}
	}
</script>

<div class="container mx-auto p-4">
	<div class="card bg-base-100 shadow-xl">
		<div class="card-body">
			<h2 class="card-title mb-4 text-2xl">WordPress Link Uploader</h2>

			<form onsubmit={handleSubmit} class="space-y-4">
				<div class="form-control">
					<label for="email" class="label">
						<span class="label-text">Email</span>
					</label>
					<input
						type="email"
						id="email"
						bind:value={email}
						placeholder="Enter your WordPress email"
						class="input input-bordered w-full"
						required
					/>
				</div>

				<div class="form-control">
					<label for="password" class="label">
						<span class="label-text">Application Password</span>
					</label>
					<input
						type="password"
						id="password"
						bind:value={password}
						placeholder="Enter your WordPress application password"
						class="input input-bordered w-full"
						required
					/>
				</div>

				<div class="form-control">
					<label for="file" class="label">
						<span class="label-text">CSV File</span>
					</label>
					<input
						type="file"
						id="file"
						accept=".csv"
						onchange={handleFileChange}
						class="file-input file-input-bordered w-full"
						required
					/>
					{#if fileName}
						<p class="mt-2 text-sm">Selected file: {fileName}</p>
					{/if}
				</div>

				<div class="form-control mt-6">
					<button type="submit" class="btn btn-primary" disabled={isLoading}>
						{isLoading ? 'Processing...' : 'Upload Links'}
					</button>
				</div>
			</form>

			{#if isLoading}
				<div class="mt-4">
					<p>Processing links... This may take a while.</p>
				</div>
			{/if}

			{#if result}
				<div class="mt-4">
					<h3 class="text-lg font-semibold">Processed Posts:</h3>
					<ul class="mt-2 list-disc pl-5">
						{#each result as post}
							<li>{post.title} - Converted to Markdown</li>
						{/each}
					</ul>
				</div>
			{/if}
		</div>
	</div>
</div>
