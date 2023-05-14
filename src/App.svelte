<script>
	import { onMount } from "svelte";

	export let data;

	let l = data[""].label[""];
	let q = data[""].q;

	const price = number => {
	  number = Number(number);
	  if (number === 0 || isNaN(number)) {
	    return "";
	  }
	  return `${q.currency} ${number.toLocaleString(undefined, {
	    minimumFractionDigits: 2,
	  })}`;
	};
	const qty = number => {
	  number = Number(number);
	  if (number === 0 || isNaN(number)) {
	    return "";
	  }
	  return number.toLocaleString(undefined, {
	    minimumFractionDigits: 0,
	  });
	};
	const rate = rate => {
	  rate = Number(rate) * 100;
	  if (!Number.isInteger(rate)) {
	    rate = rate.toFixed(2);
	  }
	  return `${rate} %`;
	};
	const addItem = () => {
	  q.itemDesc.push("");
	  q.itemPrice.push("");
	  q.itemQty.push("");
	  q = q;
	};
	const removeItem = () => {
	  q.itemDesc.pop();
	  q.itemPrice.pop();
	  q.itemQty.pop();
	  q = q;
	};

	onMount(() => {
	  const s = new URLSearchParams(location.search);
	  let obj = q;
	  Object.keys(q).forEach(key => {
	    const values = s.getAll(key);
	    if (values.length > 0) {
	      if (Array.isArray(q[key])) {
	        obj[key] = values;
	        return;
	      }
	      obj[key] = values[0];
	    }
	  });
	  q = { ...data[q.lang].q, ...obj };
	});

	$: l = {
	  ...data[q.lang].label[""],
	  ...data[q.lang].label[q.doc]
	};
	$: q.itemAmount = q.itemPrice.map((pr, i) => {
	  const num = Number(pr) * Number(q.itemQty[i]);
	  return num ? num : "";
	});
	$: q.totalAmount = q.itemAmount.reduce((a, b) => {
	  const num = Number(a) + Number(b);
	  return num ? num : "";
	}, 0);
	$: q.totalVat = Number(q.totalAmount) * Number(q.vatRate);
	$: q.totalWht = Number(q.totalAmount) * Number(q.whtRate);
	$: q.totalFinal =
	  Number(q.totalAmount) +
	  Number(q.totalVat) +
	  Number(q.totalWht) +
	  Number(q.totalAdjust);
</script>

<div class="flex flex-wrap justify-center items-center my-4 print:hidden">
	{#each Object.keys(data) as lng, i (`lang-${i}`)}
		<button class="block duration-300 px-4 py-2 shadow-xl {q.lang === lng ? "bg-purple-600 text-white" : "text-gray-900 bg-white hover:bg-purple-600 focus:bg-purple-600 hover:text-white focus:text-white"}" on:click={() => {
			q.lang = lng
		}}>
			{data[lng]['']}
		</button>
	{/each}
	{#each Object.keys(data[q.lang].label) as dc, i (`doc-${i}`)}
		<button class="block duration-300 px-4 py-2 shadow-xl {q.doc === dc ? "bg-purple-600 text-white" : "text-gray-900 bg-white hover:bg-purple-600 focus:bg-purple-600 hover:text-white focus:text-white"}" on:click={() => {
			q.doc = dc
		}}>
			{data[q.lang].label[dc].title}
		</button>
	{/each}
</div>

<div class="font-sans bg-white text-black p-6 max-w-[60rem] mx-auto print:max-w-none print:mx-0">
	<div class="grid grid-cols-2 gap-6">
		<div class="">
			<div class="shadow-xl mb-6 rounded-lg">
				<p class="px-4 pt-2 text-lg" contenteditable="true" bind:textContent={q.vendorName}></p>
				<p class="px-4" contenteditable="true" bind:textContent={q.vendorId}></p>
				<p class="px-4 pb-2" contenteditable="true" bind:textContent={q.vendorAddress}></p>
			</div>
			<div class="shadow-xl mb-6 rounded-lg">
				<h3 class="text-lg font-semibold text-center bg-purple-600 text-white p-2 rounded-t-lg">{l.client}</h3>
				<p class="px-4 pt-2 text-lg" contenteditable="true" bind:textContent={q.clientName}></p>
				<p class="px-4" contenteditable="true" bind:textContent={q.clientId}></p>
				<p class="px-4 pb-2" contenteditable="true" bind:textContent={q.clientAddress}></p>
			</div>
		</div>
		<div class="">
			<div class="shadow-xl mb-6 rounded-lg">
				<h3 class="text-lg font-semibold text-center bg-purple-600 text-white p-2 rounded-t-lg">{l.paymethod}</h3>
				<p class="px-4 py-2 text-center" contenteditable="true" bind:textContent={q.paymethod}></p>
			</div>
			<div class="shadow-xl mb-6 rounded-lg grid grid-cols-2 p-2">
				<h1 class="text-4xl text-center text-purple-600 font-bold pb-2 col-span-full">{l.title}</h1>
				<div class="pr-1 text-right text-purple-600">{l.ref}</div>
				<p class="pl-1" contenteditable="true" bind:textContent={q.ref}></p>
				<div class="pr-1 text-right text-purple-600">{l.date}</div>
				<p class="pl-1" contenteditable="true" bind:textContent={q.date}></p>
				{#if q.doc !== 'receipt'}
					<div class="pr-1 text-right text-purple-600">{l.duedate}</div>
					<p class="pl-1" contenteditable="true" bind:textContent={q.duedate}></p>
				{/if}
			</div>
		</div>
	</div>
	<table class="w-full mb-6">
		<thead class="text-center text-purple-600">
			<tr class="">
				<th class="p-2 border">{l.itemNo}</th>
				<th class="border">
					<div class="flex">
						<p class="p-2 flex-grow">{l.itemDesc}</p>
						<button class="p-2 text-purple-600 focus:bg-purple-200 hover:bg-purple-200 rounded-full transition duration-300 ease-in-out print:hidden" on:click={addItem}>
							<!-- heroicons solid duplicate -->
							<svg class="h-6 w-6" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 20 20" fill="currentColor">
								<path d="M7 9a2 2 0 012-2h6a2 2 0 012 2v6a2 2 0 01-2 2H9a2 2 0 01-2-2V9z" />
								<path d="M5 3a2 2 0 00-2 2v6a2 2 0 002 2V5h8a2 2 0 00-2-2H5z" />
							</svg>
						</button>
						<button class="p-2 text-purple-600 focus:bg-purple-200 hover:bg-purple-200 rounded-full transition duration-300 ease-in-out print:hidden" on:click={removeItem}>
							<!-- heroicons solid trash -->
							<svg class="h-6 w-6" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 20 20" fill="currentColor">
								<path fill-rule="evenodd" d="M9 2a1 1 0 00-.894.553L7.382 4H4a1 1 0 000 2v10a2 2 0 002 2h8a2 2 0 002-2V6a1 1 0 100-2h-3.382l-.724-1.447A1 1 0 0011 2H9zM7 8a1 1 0 012 0v6a1 1 0 11-2 0V8zm5-1a1 1 0 00-1 1v6a1 1 0 102 0V8a1 1 0 00-1-1z" clip-rule="evenodd" />
							</svg>
						</button>
					</div>
				</th>
				<th class="p-2 border">{l.itemPrice}</th>
				<th class="p-2 border">{l.itemQty}</th>
				<th class="p-2 border">{l.itemAmount}</th>
			</tr>
		</thead>
		<tbody class="">
			{#each q.itemDesc as _, i (`item-${i}`)}
				<tr class="">
					<td class="p-2 border text-center" contenteditable="true">{i + 1}</td>
					<td class="p-2 border" contenteditable="true" bind:textContent={q.itemDesc[i]}></td>
					<td class="p-2 border text-center" contenteditable="true" 
						on:focus={(e) => {e.target.textContent = q.itemPrice[i]}}
						on:input={(e) => {q.itemPrice[i] = e.target.textContent}}
						on:blur={(e) => {e.target.textContent = price(q.itemPrice[i])}}
					>
						{price(q.itemPrice[i])}
					</td>
					<td class="p-2 border text-center" contenteditable="true" 
						on:focus={(e) => {e.target.textContent = q.itemQty[i]}}
						on:input={(e) => {q.itemQty[i] = e.target.textContent}}
						on:blur={(e) => {e.target.textContent = qty(q.itemQty[i])}}
					>
						{qty(q.itemQty[i])}
					</td>
					<td class="p-2 border text-right">
						{price(q.itemAmount[i])}
					</td>
				</tr>
			{/each}
		</tbody>
		<tfoot class="">
			<tr class="">
				<td class="p-2 border" colspan="2"></td>
				<td class="p-2 border text-center" colspan="2">{l.totalAmount}</td>
				<td class="p-2 border text-right">
					{price(q.totalAmount)}
				</td>
			</tr>
		</tfoot>
	</table>
	<div class="grid grid-cols-2 gap-6">
		<div>
			<div class="shadow-xl mb-6 rounded-lg">
				<h3 class="text-lg font-semibold text-center bg-purple-600 text-white p-2 rounded-t-lg">{l.note}</h3>
				<p class="px-4 py-2 text-center" contenteditable="true" bind:textContent={q.note}></p>
			</div>
			<div class="shadow-xl mb-6 rounded-lg">
				<div class="">
					<img class="mx-auto" src={q.vendorStamp} alt="" width="" height="">
				</div>
				<h2 class="text-lg p-2 text-center font-semibold text-purple-600">{l.vendorSign}</h2>
				<p class="px-4 pt-2 text-center" contenteditable="true"></p>
				<p class="px-4 pb-2 italic text-right" contenteditable="true"></p>
			</div>
		</div>
		<div>
			<div class="shadow-xl mb-6 rounded-lg grid grid-cols-2 p-2">
				<div class="pr-1 text-right text-purple-600">
					<span class="">{l.totalVat}</span>
					<span class=""> </span>
					<span class="" contenteditable="true" 
						on:focus={(e) => {e.target.textContent = q.vatRate}}
						on:input={(e) => {q.vatRate = e.target.textContent}}
						on:blur={(e) => {e.target.textContent = rate(q.vatRate)}}
					>
						{rate(q.vatRate)}
					</span>
				</div>
				<p class="pl-1">
					{price(q.totalVat)}
				</p>
				{#if q.doc === 'receipt'}
					<div class="pr-1 text-right text-purple-600">
						<span class="">{l.totalWht}</span>
						<span class=""> </span>
						<span class="" contenteditable="true" 
							on:focus={(e) => {e.target.textContent = q.whtRate}}
							on:input={(e) => {q.whtRate = e.target.textContent}}
							on:blur={(e) => {e.target.textContent = rate(q.whtRate)}}
						>
							{rate(q.whtRate)}
						</span>
					</div>
					<p class="pl-1">
						{price(q.totalWht)}
					</p>
				{/if}
				<div class="pr-1 text-right text-purple-600">{l.totalAdjust}</div>
				<p class="pl-1" contenteditable="true" 
					on:focus={(e) => {e.target.textContent = q.totalAdjust}}
					on:input={(e) => {q.totalAdjust = e.target.textContent}}
					on:blur={(e) => {e.target.textContent = price(q.totalAdjust)}}
				>
					{price(q.totalAdjust)}
				</p>
				<div class="pr-1 text-right text-purple-600 font-bold">{l.totalFinal}</div>
				<p class="pl-1 text-purple-600 font-bold">
					{price(q.totalFinal)}
				</p>
			</div>
			<div class="shadow-xl mb-6 rounded-lg">
				<h3 class="text-lg font-semibold text-center bg-purple-600 text-white p-2 rounded-t-lg">{l.clientSign}</h3>
				<p class="px-4 pt-2 text-center" contenteditable="true"></p>
				<p class="px-4 pb-2 italic text-right" contenteditable="true"></p>
			</div>
			<h2 class="text-lg p-2 text-center font-semibold text-purple-600">{l.thankMessage}</h2>
		</div>
	</div>
</div>

<div class="flex flex-wrap justify-center items-center my-4 print:hidden gap-4">
	<label class="">
		<span class="">Currency</span>
		<input class="border border-purple-600 w-12" bind:value={q.currency} />
	</label>
	<button class="block duration-300 px-4 py-2 shadow-xl text-white bg-purple-600 hover:bg-white focus:bg-white hover:text-gray-900 focus:text-gray-900" on:click={() => {window.print()}}>
		Print
	</button>
</div>
