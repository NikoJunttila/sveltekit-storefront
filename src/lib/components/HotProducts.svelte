<script lang="ts">
	import * as m from '$lib/paraglide/messages.js';
	import { formatCurrency } from '$lib/utils';
	import { getContextClient } from '@urql/svelte';
	import { type ProductListOptions, type ProductDetailFragment } from '$lib/gql/graphql';
	import { GetProducts } from '../vendure';

	const client = getContextClient();
	const options: ProductListOptions = {
		take: 8,
		skip: 0
		//enable this to use actual popularity
		//https://vendure.io/hub/popularity-score
		// sort : {
		//     popularityScore : SortOrder.Desc
		// }
	};

	let products: ProductDetailFragment[] | any[] = $state([]);

	let loading = $state(false);
	let error = $state<string | null>(null);

	async function getProducts() {
		loading = true;
		error = null;

		try {
			const result = await client.query(GetProducts, { options });
			// console.log('Query result:', result);
			// Check for GraphQL errors
			if (result.error) {
				throw new Error(`GraphQL Error: ${result.error.message}`);
			}

			// Check if we have data
			if (!result.data?.products.items) {
				throw new Error('No search data returned from server');
			}
			products = result.data?.products.items || [];
		} catch (err) {
			console.error('Error fetching products:', err);
			error = err instanceof Error ? err.message : 'Failed to fetch products';
		} finally {
			loading = false;
		}
	}

	$effect(() => {
		getProducts();
	});
</script>

<!-- Hot products section - Mosaic grid -->
<section class="mx-auto my-8 max-w-screen-2xl p-4">
	<h2 class="mb-8 text-center text-3xl font-bold">
		<span class="flame-gradient">🔥 {m.hot_products()} 🔥</span>
	</h2>

	{#if loading}
		<div class="flex items-center justify-center py-12">
			<div class="text-lg text-gray-600">Loading products...</div>
		</div>
	{:else if error}
		<div class="rounded-lg border border-red-200 bg-red-50 p-4 text-center">
			<p class="font-medium text-red-800">Error loading products</p>
			<p class="mt-1 text-sm text-red-600">{error}</p>
			<button
				onclick={() => getProducts()}
				class="mt-3 rounded bg-red-600 px-4 py-2 text-white transition-colors hover:bg-red-700"
			>
				Try Again
			</button>
		</div>
	{:else if products.length === 0}
		<div class="py-12 text-center">
			<p class="text-gray-600">No products found</p>
		</div>
	{:else}
		<div class="grid grid-cols-2 overflow-hidden rounded-md md:grid-cols-3 lg:grid-cols-4">
			{#each products as product, i}
				<a
					aria-label={`Browse ${product.name} product`}
					href={`/product/${product.slug}`}
					class="group relative overflow-hidden shadow-md
	                       {i === 0 ? 'md:col-span-2 md:row-span-2' : ''}
	                       {i === 3 ? 'lg:col-span-2' : ''}"
				>
					<div class="aspect-square h-full w-full">
						{#if product.featuredAsset}
							<img
								loading="lazy"
								src={product.featuredAsset.preview}
								alt={product.featuredAsset.name}
								class="h-full w-full object-cover transition-transform duration-300 group-hover:scale-105"
								width="600"
								height="600"
							/>
						{/if}
					</div>
					<!-- Overlay content -->
					<div
						class="absolute inset-0 flex flex-col justify-end bg-gradient-to-t
	                            from-black/40 to-transparent p-4"
					>
						<h3 class="text-center text-lg font-semibold text-white drop-shadow-lg md:text-xl">
							{product.name}
						</h3>
						<div
							class="absolute inset-0 flex items-center justify-center bg-black/60
	                                p-4 opacity-0 transition-opacity duration-300 group-hover:opacity-100"
						>
							<div class="text-center text-xl text-white">
								{formatCurrency(product.variants[0].priceWithTax, product.variants[0].currencyCode)}
							</div>
						</div>
					</div></a
				>
			{/each}
		</div>
	{/if}
</section>

<style>
	/* Custom grid sizing for mosaic effect */
	@media (min-width: 768px) {
		.grid {
			grid-auto-rows: minmax(200px, auto);
		}
		.md\:col-span-2 {
			grid-column: span 2;
		}
		.md\:row-span-2 {
			grid-row: span 2;
		}
	}
	.flame-gradient {
		background: linear-gradient(to right, #dc6a23, #ff8c00, #ff4500, #ff0000);
		-webkit-background-clip: text;
		background-clip: text;
		-webkit-text-fill-color: transparent;
		color: transparent;
	}
</style>
