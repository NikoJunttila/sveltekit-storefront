<script lang="ts">
	import { formatCurrency } from '$lib/utils';
	import { useFragment } from '$lib/gql';
	import { ShippingMethodQuote } from '$lib/vendure';
	import * as m from '$lib/paraglide/messages';

	interface Props {
		shippingOptions: any[];
		selectedShippingOption: string;
		setShippingOption: (option: string[]) => void; // Ensure function updates state
		channel : any
	}

	let {
		shippingOptions,
		selectedShippingOption = $bindable(),
		setShippingOption,
		channel
	}: Props = $props();

</script>

<section class="overflow-hidden rounded-lg bg-white shadow-sm ring-1 ring-gray-900/5">
	<div class="border-b border-gray-200 bg-gray-50 px-6 py-4">
		<h2 class="text-lg font-semibold text-gray-900">{m.delivery_methods()}</h2>
	</div>
	<div class="p-6">
		<div class="space-y-4" role="radiogroup">
				{#each useFragment(ShippingMethodQuote, shippingOptions) as shippingOption}
					<label
						class="flex cursor-pointer items-center space-x-3 rounded-lg border border-gray-200 p-4 hover:border-lime-500"
					>
						<input
							onchange={() => setShippingOption([shippingOption.id])}
							type="radio"
							name="selectedShippingOption"
							value={shippingOption.id}
							bind:group={selectedShippingOption}
							class="h-4 w-4 border-gray-300 text-lime-600 focus:ring-lime-500"
						/>
						<div class="flex flex-1 items-center justify-between">
							<span class="text-sm font-medium text-gray-900">{shippingOption.name}</span>
							<span class="text-sm font-medium text-gray-900">
								{formatCurrency(shippingOption.price, channel.defaultCurrencyCode)}
							</span>
						</div>
					</label>
				{:else}
					<p class="text-sm text-gray-500">No shipping options available</p>
				{/each}
		</div>
	</div>
</section>
