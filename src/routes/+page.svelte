<script>
	import { collection, getDocs } from 'firebase/firestore';
	import { onMount } from 'svelte';
	import { db } from '../firebase';

	import Tester from '../components/tester/tester.svelte';

	let connected = true;

	onMount(async () => {
		try {
			// Try to fetch documents from a test collection (could be any collection you know exists)
			const testCollection = collection(db, 'testCollection');
			await getDocs(testCollection);
			console.log('Database connected.');
		} catch (error) {
			console.error('Failed to connect to Firestore:', error);
			connected = false;
		}
	});
</script>

{#if connected}
	<Tester />
{:else}
	<p>
		Failed to connect to the database. Please check your internet connection or try again later.
	</p>
{/if}
