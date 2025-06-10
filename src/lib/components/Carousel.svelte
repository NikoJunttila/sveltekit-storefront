<script lang="ts">
	import * as m from '$lib/paraglide/messages';
	import { getContextClient } from '@urql/svelte';
	import { formatCurrency } from '$lib/utils';
	import { type SearchInput } from '$lib/gql/graphql';
	import { SearchProducts } from '../vendure';

	let { slug, current } = $props<{ slug: string; current: string }>();
	const client = getContextClient();

	const options: SearchInput = {
		take: 4,
		skip: 0,
		collectionSlug: slug,
		groupByProduct: true
	};

	let products: any[] = $state([]);
	let loading = $state(true); // Start in loading state
	let error = $state<string | null>(null);
	let filterCurrent = $derived(products.filter((p) => p.productName != current));

	async function getProducts() {
		loading = true;
		error = null;
		try {
			const result = await client.query(SearchProducts, { input: options });

			if (result.error) {
				throw new Error(`GraphQL Error: ${result.error.message}`);
			}

			products = result.data?.search.items || [];
		} catch (err) {
			console.error('Error fetching products:', err);
			error = err instanceof Error ? err.message : 'Failed to fetch products';
		} finally {
			loading = false;
		}
	}

	// Helper functions remain the same
	function getProductImage(product: any) {
		return (
			product?.productAsset?.preview ||
			product?.featuredAsset?.preview ||
			'/placeholder-product.jpg'
		);
	}

	function getProductUrl(product: any) {
		return `/product/${product.slug}`;
	}

	// The $effect hook is now much simpler
	$effect(() => {
		getProducts();
	});
</script>

<section class="px-4 py-8">
	<h2 class="mb-8 text-center text-3xl font-bold text-gray-900">{m.related_products()}</h2>

	{#if loading}
		<div class="flex items-center justify-center py-12">
			<div class="flex items-center space-x-2">
				<div
					class="h-4 w-4 animate-spin rounded-full border-2 border-gray-300 border-t-blue-600"
				></div>
				<span class="text-lg text-gray-600">{m.loading}...</span>
			</div>
		</div>
	{:else if error}
		<div class="mx-auto max-w-md rounded-lg border border-red-200 bg-red-50 p-6 text-center">
			<div class="mb-2 text-red-600">
				<svg class="mx-auto h-8 w-8" fill="none" stroke="currentColor" viewBox="0 0 24 24">
					<path
						stroke-linecap="round"
						stroke-linejoin="round"
						stroke-width="2"
						d="M12 8v4m0 4h.01M21 12a9 9 0 11-18 0 9 9 0 0118 0z"
					></path>
				</svg>
			</div>
			<p class="font-medium text-red-800">{m.error_loading_products()}</p>
			<p class="mt-1 text-sm text-red-600">{error}</p>
			<button
				onclick={getProducts}
				class="mt-4 rounded-md bg-red-600 px-4 py-2 text-white transition-colors hover:bg-red-700 focus:outline-none focus:ring-2 focus:ring-red-500 focus:ring-offset-2"
			>
				{m.try_again()}
			</button>
		</div>
	{:else if products.length === 0}
		<div class="py-12 text-center">
			<div class="mb-4 text-gray-400">
				<svg class="mx-auto h-12 w-12" fill="none" stroke="currentColor" viewBox="0 0 24 24">
					<path
						stroke-linecap="round"
						stroke-linejoin="round"
						stroke-width="2"
						d="M20 7l-8-4-8 4m16 0l-8 4m8-4v10l-8 4m0-10L4 7m8 4v10M4 7v10l8 4"
					></path>
				</svg>
			</div>
			<p class="text-lg text-gray-600">{m.related_not_found()}</p>
		</div>
	{:else}
		<div class="grid grid-cols-1 gap-4 sm:grid-cols-2 md:grid-cols-3 lg:grid-cols-4">
			{#each filterCurrent as product}
				<div
					class="group relative flex h-full flex-col overflow-hidden rounded-lg border border-gray-200 bg-white shadow-sm transition-all hover:shadow-lg"
				>
					<a href={getProductUrl(product)} class="flex h-full flex-col">
						<div class="aspect-square overflow-hidden bg-gray-100">
							<img
								src={getProductImage(product)}
								alt={product.productName}
								class="h-full w-full object-cover transition-transform group-hover:scale-105"
								loading="lazy"
								width="300"
								height="300"
							/>
						</div>

						<div class="flex flex-grow flex-col justify-between p-4">
							<h3
								class="mb-2 line-clamp-2 text-sm font-medium text-gray-900 group-hover:text-blue-600"
							>
								{product.productName}
							</h3>

							<div class="mt-auto">
								{#if product.price}
									<div class="flex items-center gap-1">
										<span class="text-lg font-bold text-gray-900">
											{formatCurrency(product.price.min, product.currencyCode)}
										</span>

										{#if product.price.min !== product.price.max}
											<span class="text-lg font-bold text-gray-900">
												- {formatCurrency(product.price.max, product.currencyCode)}
											</span>
										{/if}
									</div>
								{/if}

								{#if !product.inStock}
									<span class="mt-1 text-xs font-medium text-red-600">{m.out_of_stock()}</span>
								{/if}
							</div>
						</div>
					</a>
				</div>
			{/each}
		</div>
	{/if}
</section>

<style>
	.line-clamp-2 {
		display: -webkit-box;
		-webkit-line-clamp: 2;
		line-clamp: 2;
		-webkit-box-orient: vertical;
		overflow: hidden;
		text-overflow: ellipsis;
	}
</style>
