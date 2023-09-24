<script>
	import { collection, getDocs } from 'firebase/firestore';
	import { onMount } from 'svelte';
	import { db } from '../firebase';

	import {
		Button,
		Header,
		HeaderUtilities
	} from 'carbon-components-svelte';

	import {
		Debug,
		Login
	} from 'carbon-icons-svelte';

	// pictograms

	// variables
	let modal1 = false;
	let modal2 = false;
	let modal3 = false;
	let confirmEnabled = false;

	// for modal checkbox
	function toggleButtonState() {
		confirmEnabled = !confirmEnabled;
	}

	function handleLogin() {}

	let connected = true;

	onMount(async () => {
		try {
			// Try to fetch documents from a test collection (could be any collection you know exists)
			const testCollection = collection(db, 'testCollection');
			await getDocs(testCollection);
			console.log('Database connected.');
		} catch (error) {
			console.error('Failed to connect to database. :', error);
			connected = false;
		}
	});
</script>

<!-- {#if connected}
 -->
<Header company="Project" platformName="JOAN" href="/">
	<div class="text-white hidden lg:flex">Colegio de San Juan de Dios, Incorporated</div>
	<div class="text-white flex lg:hidden">for&nbsp<strong>CSJD</strong></div>
	<HeaderUtilities>
		<div class="flex">
			<div class="flex">
				<Button
					href="/bugs"
					tooltipAlignment="end"
					tooltipPosition="bottom"
					iconDescription="Report a problem"
					kind="secondary"
					icon={Debug}
				/>
				<Button
					href="/login"
					tooltipAlignment="end"
					tooltipPosition="bottom"
					iconDescription="Login"
					kind="primary"
					icon={Login}
				/>
			</div>
		</div>
	</HeaderUtilities>
</Header>

<!-- adapts depending on the device -->
<div class="flex flex-col items-stretch lg:flex-row" />
<!-- {:else}
	
{/if} -->
