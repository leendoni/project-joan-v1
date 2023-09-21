<script>
	import { collection, getDocs } from 'firebase/firestore';
	import { onMount } from 'svelte';
	import { db } from '../firebase';

	import {
		Button,
		ButtonSet,
		Checkbox,
		Content,
		FluidForm,
		Form,
		FormGroup,
		Header,
		HeaderUtilities,
		Link,
		Modal,
		PasswordInput,
		RadioButton,
		RadioButtonGroup,
		TextInput
	} from 'carbon-components-svelte';
	
	import {
		Carbon,
		HelpFilled,
		Login,
		WatsonHealthCrossReference
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
					href="/godmode"
					tooltipAlignment="end"
					tooltipPosition="bottom"
					iconDescription="Godmode"
					kind="secondary"
					icon={WatsonHealthCrossReference}
				/>
				<Button
					tooltipAlignment="end"
					tooltipPosition="bottom"
					iconDescription="About JOAN"
					kind="secondary"
					icon={HelpFilled}
					on:click={() => (modal3 = true)}
				/>
			</div>
		</div>
	</HeaderUtilities>
</Header>

<!-- adapts depending on the device -->
<div class="flex flex-col items-stretch lg:flex-row">
	<!-- displayed on mobile -->
	<Content id="#" class="flex flex-col bg-neutral-900 h-60 lg:hidden">
		<h1 class="pt-20 text-white">Sign in to your<br />JOAN Account.</h1>
	</Content>
	<Content id="#" class="bg-neutral-900 w-1/3 h-screen hidden lg:flex" />
	<Content class="self-center w-11/12 lg:w-2/5">
		<h1 class="pb-12 hidden lg:flex">Sign in to your<br />JOAN Account.</h1>
		<Form on:submit>
			<FormGroup legendText="Enter your account information.">
				<FluidForm>
					<div class="flex" />
					<TextInput labelText="Username" placeholder="Enter username..." required />
					<PasswordInput
						required
						type="password"
						labelText="Password"
						placeholder="Enter password..."
					/>
				</FluidForm>
			</FormGroup>
			<ButtonSet stacked>
				<Button on:click={handleLogin} icon={Login}>Log in</Button>
				<Button on:click={() => (modal1 = true)} kind="ghost">Forgot password?</Button>
			</ButtonSet>
			<br />
			<div>
				New student?
				<Link href="/registration/student" icon={Carbon}>Register here.</Link>.
			</div>
		</Form>
	</Content>
</div>

<Modal
	bind:open={modal1}
	modalHeading="Reset your password"
	primaryButtonText="Confirm"
	primaryButtonDisabled={!confirmEnabled}
	secondaryButtonText="Cancel"
	selectorPrimaryFocus="#db-name"
	on:click:button--secondary={() => (modal1 = false)}
	on:click:button--primary={() => (modal2 = true)}
	on:click:button--primary={() => (modal1 = false)}
>
	<br />
	<p>To reset your account, provide the following information:</p>
	<br />
	<RadioButtonGroup selected="stud" legendText="Account Type">
		<RadioButton labelText="Student" value="student" />
		<RadioButton labelText="Employee" value="employee" />
	</RadioButtonGroup>
	<br />
	<TextInput
		labelText="Account Code"
		placeholder="Enter your Account Code"
		helperText="If unable to be found, contact your system administrator."
	/>
	<br />
	<Checkbox bind:checked={confirmEnabled} labelText="The information above is correct." />
</Modal>
<Modal
	bind:open={modal2}
	modalHeading="Reset your password"
	primaryButtonText="Return to Login"
	primaryButtonDisabled={!confirmEnabled}
	on:click:button--primary={() => (modal2 = false)}
>
	<br />
	<h4>A notification to reset your account has been sent to your System Administrator.</h4>
	<br />
	<h5>Temporary Password:</h5>
	<h3>csjd.reset123</h3>
	<br />
	<p class="italic">
		Note: Try logging in to your account within 1-2 days from resetting your password. If the
		temporary password does not work within that period, contact your system administrator for a
		follow-up reset.
	</p>
</Modal>
<Modal
	bind:open={modal3}
	modalHeading=""
	primaryButtonText="Close"
	on:click:button--primary={() => (modal3 = false)}
>
	<br />
	<h1>Project JOAN for CSJDD</h1>
	<p class="italic">a part of the Joint Online Academic Network</p>
	<br />
	<p>
		This version of Project JOAN is tailored for Colegio de San Juan de Dios Incorporated; a
		private-catholic school in San Rafael, Bulacan.
	</p>
	<br />
	<p>
		Merging state-of-the-art technologies like Svelte, SvelteKit, and TailwindCSS with the robust
		and secure backbone of Firebase, Project JOAN is a web-based platform dedicated to streamlining
		academic, financial, and administrative processes. Designed with precision, Project JOAN is a
		testament to efficiency, user-friendliness, and unwavering data security.
	</p>
</Modal>
<!-- {:else}
	
{/if} -->
