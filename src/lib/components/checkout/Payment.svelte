<script lang="ts">
	import StripeComponent from './StripeComponent.svelte';
	import { getContextClient } from '@urql/svelte';
	import { DummyPayment, TransitionOrderToState } from '$lib/vendure';
	import { goto } from '$app/navigation';
	import { cartStore } from '$src/lib/stores';

	const client = getContextClient();
	export let currencyCode
	const setOrderState = async () => {
		let result = await client.mutation(TransitionOrderToState, { state:"ArrangingPayment" }).toPromise();
	};

	async function makePayment(){
		await setOrderState()
		const res = await client.mutation(DummyPayment, {})
		console.log(res)
		setTimeout(() => {
			//@ts-ignore
			goto(`/checkout/success/${$cartStore.code}`)
		}, 500);
	}

</script>

<div>
	<div class="grid columns-2 gap-2">
		<div>
			<!-- {JSON.stringify($cartStore)} -->
		<button class="button" onclick={makePayment}>
			DUMMY PAYMENT
		</button>
		</div>

		<div>
			<StripeComponent {currencyCode}></StripeComponent>
		</div>
	</div>
</div>
