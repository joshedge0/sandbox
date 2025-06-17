<script lang="ts">
	import { Plus } from '@lucide/svelte';

	//type expenseCategory = 'Weekly' | 'Monthly';

	interface expense {
		expenseCategory: string;
		note: String;
		amount: number;
	}

    interface errors {
        category?: string;
        amount?: string;
    }

	let expenses: expense[] = [];

	let monthlyBudget: number = 1200;
	let monthlyCurrent: number = 1200;
	let monthlyPercent: number = 100;

	let weeklyBudget: number = 300;
	let weeklyCurrent: number = 300;
	let weeklyPercent: number = 100;

	let category: string = '';
	let amount: number;
	let note: string = '';
    let errors: errors = {};

	const handleSubmitExpense = () => {
        errors = {};
        if (!category) errors.category = 'Category is required';
        if (!amount) errors.amount = 'Amount is required';

        if (Object.keys(errors).length === 0) {
            monthlyCurrent -= amount;
            monthlyPercent = (monthlyCurrent / monthlyBudget) * 100;
            if (category == 'Weekly') {
                weeklyCurrent -= amount;
                weeklyPercent = (weeklyCurrent / weeklyBudget) * 100;
            }

            const newExpense: expense = {
                expenseCategory: category,
                note: note,
                amount: amount
            };

            expenses = [...expenses, newExpense];
        }
	};
</script>

<div class="flex flex-col">
	<h1 class="mx-auto text-3xl font-bold mt-[40px]">budget</h1>

	<div class="flex flex-row m-auto">
		<div class="p-4">
			<p>monthly: {monthlyCurrent}</p>
			<div class="pie" style="--percent: {monthlyPercent};"></div>
		</div>
		<div class="p-4">
			<p>weekly: {weeklyCurrent}</p>
			<div class="pie" style="--percent: {weeklyPercent};"></div>
		</div>
	</div>

    <form class="flex flex-col m-auto" on:submit|preventDefault={handleSubmitExpense}>
		<div class="flex flex-row m-auto">
			<div>
				<label for="category">Category</label>
				<select id="category" bind:value={category} class="border px-2 py-1 rounded">
					<option value="" disabled selected>Select category</option>
					<option value="Weekly">Weekly</option>
					<option value="Monthly">Monthly</option>
				</select>
                {#if errors.category}<p class="text-red-500">{errors.category}</p>{/if}
			</div>
			<div>
				<label for="note">Note</label>
				<input id="note" bind:value={note} placeholder="Note" />
			</div>
			<div>
				<label for="amount">Amount</label>
				<input id="amount" bind:value={amount} placeholder="Amount" />
			</div>
            {#if errors.amount}<p class="text-red-500">{errors.amount}</p>{/if}
		</div>

		<button class="btn" type="submit"><Plus />Add</button>
	</form>

	<table class="table-auto border-collapse border border-gray-300 m-auto">
		<thead>
			<tr class="bg-gray-100">
				<th class="border border-gray-300 px-4 py-2 text-left">Category</th>
				<th class="border border-gray-300 px-4 py-2 text-left">Note</th>
				<th class="border border-gray-300 px-4 py-2 text-left">Amount</th>
			</tr>
		</thead>
		<tbody>
			{#each expenses as expense}
				<tr>
					<td class="border border-gray-300 px-4 py-2">{expense.expenseCategory}</td>
					<td class="border border-gray-300 px-4 py-2">{expense.note}</td>
					<td class="border border-gray-300 px-4 py-2">{expense.amount}</td>
				</tr>
			{/each}
		</tbody>
	</table>
</div>

<style>
	.pie {
		width: 150px;
		height: 150px;
		border-radius: 50%;
		background: conic-gradient(#4caf50 calc(var(--percent) * 1%), #e0e0e0 0);
	}
</style>
