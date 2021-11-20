<script>
import {onMount} from 'svelte';
import {writable} from 'svelte/store';

import {db} from './firebase';
let todayDateString = new Date().toISOString().slice(0,10);
export let date= todayDateString;
let customerList = writable([]);
let name ="";
let mobileNumber ="";
let remindingTime = "";

async function loadCustomerList(){
	let collRef = db.collection('medicine-reminder-customer');
  	let allDocs= await collRef.get();
	let list = [];
	for(const doc of allDocs.docs){
		let docData = doc.data();
		let customer = {
			id:doc.id, 
			name: docData.name,
			mobileNumber: docData.mobileNumber,
			remindingTime: docData.remindingTime // daily reminder, one reminder only , given reminding time
		};
		list.push(customer);
   		console.log(doc.id, '=>', doc.data());
  	}
	customerList.set(list);
}

onMount(async () => {
	loadCustomerList();
});

function handleOnSubmit() {
	console.log("form submitted");
	db.collection("medicine-reminder-customer").add({
	    date,	
		name,
		mobileNumber,
		remindingTime
	})
	.then(() => {
		console.log("Document successfully written!");
		loadCustomerList();
	})
	.catch((error) => {
		console.error("Error writing document: ", error);
	});
}

function deleteCustomer(id){
	console.log("remove customer id:", id);
	db.collection("medicine-reminder-customer").doc(id).delete().then(() => {
		console.log("Document successfully deleted!");
		loadBathList();
	}).catch((error) => {
		console.error("Error removing document: ", error);
	});
}
</script>

<main>
	<!-- <EnvTest></EnvTest> -->
<h1>Medicine Reminder</h1>
<p>Daily Reminder given time</p>

<form on:submit|preventDefault={handleOnSubmit}>
	
	<h3>Add Customer</h3>
	<label>Date:<input type="date" bind:value={date}/></label>
	<label><input type="text" bind:value={name}/>Name</label>
	<label><input type="tel" bind:value={mobileNumber}/>Mobile Number</label>
	<label><input type="time" bind:value={remindingTime}/>Reminding time</label>
	<button type="submit">Save</button>
</form>

<hr/>
<h3>Customer list</h3>
<div class="customerList">
	{#each $customerList as customer}
		<div>
			{customer.bathDate} <br/>
			Name: {customer.name}<br/>
			Mobile: {customer.mobileNumber} <br/>
			Time: {customer.remindingTime} <br/>
			<button on:click={deleteCustomer(customer.id)}>Delete</button>
		</div>
	{/each}
</div>
</main>

<style>

</style>