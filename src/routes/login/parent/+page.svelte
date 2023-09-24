<script>
	import { collection, doc, getDocs } from 'firebase/firestore';
	import { onMount } from 'svelte';
	import { db } from '../../../firebase';

	import {
		Button,
		ButtonSet,
		Checkbox,
		CodeSnippet,
		Content,
		FluidForm,
		Form,
		FormGroup,
		Header,
		HeaderUtilities,
		Modal,
		ProgressBar,
		RadioButton,
		RadioButtonGroup,
		TextArea,
		TextInput
	} from 'carbon-components-svelte';

	import { Debug, Login } from 'carbon-icons-svelte';

	// pictograms

	// variables
	let mdalBG1 = false;
	let mdalAB1 = false;

	let pageID = '/login/parent';
	let progTX = '';

	function handleLogin() {}

	let connected = false;

	onMount(async () => {
		try {
			// Try to fetch documents from a test collection (could be any collection you know exists)
			const getSchoolID = doc(db, 'schools', '0303001');
			progTX = 'Retrieving School ID';
			const getUsers = collection(getSchoolID, 'users');
			progTX = 'Connecting to User Database';
			await getDocs(getUsers);
			console.log('Database connected.');
			connected = true;
		} catch (error) {
			console.error('Failed to connect to database. :', error);
			connected = false;
		}
	});
</script>

{#if connected}
	<Header company="Project" platformName="JOAN" href="/">
		<div class="text-white hidden lg:flex">Colegio de San Juan de Dios, Incorporated</div>
		<div class="text-white flex lg:hidden">for&nbsp<strong>CSJD</strong></div>
		<HeaderUtilities>
			<div class="flex">
				<div class="flex">
					<Button
						tooltipAlignment="end"
						tooltipPosition="bottom"
						iconDescription="Report a problem"
						kind="secondary"
						icon={Debug}
						on:click={() => (mdalBG1 = true)}
					/>
				</div>
			</div>
		</HeaderUtilities>
	</Header>

	<!-- adapts depending on the device -->
	<div class="flex flex-col items-stretch lg:flex-row">
		<!-- displayed on mobile -->
		<Content id="#" class="flex flex-col bg-neutral-900 h-60 lg:hidden">
			<h1 class="pt-32 text-white">Parent Log-in</h1>
		</Content>
		<Content id="#" class="bg-neutral-900 w-1/3 h-screen hidden lg:flex" />
		<Content class="self-center w-11/12 lg:w-2/5">
			<h1 class="pb-12 hidden lg:flex">Parent Log-in</h1>
			<Form on:submit>
				<FormGroup legendText="Enter your progeny's information.">
					<FluidForm>
						<div class="flex" />
						<TextInput labelText="Student LRN" placeholder="Enter your progeny's LRN" required />
						<TextInput
							labelText="Account Code"
							placeholder="Enter your progeny's Account Code"
							required
						/>
					</FluidForm>
				</FormGroup>
				<ButtonSet stacked>
					<Button on:click={handleLogin} icon={Login}>Log in</Button>
				</ButtonSet>
				<br />
			</Form>
		</Content>
	</div>

	<Modal
		bind:open={mdalAB1}
		modalHeading=""
		primaryButtonText="Close"
		on:click:button--primary={() => (mdalAB1 = false)}
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
			and secure backbone of Firebase, Project JOAN is a web-based platform dedicated to
			streamlining academic, financial, and administrative processes. Designed with precision,
			Project JOAN is a testament to efficiency, user-friendliness, and unwavering data security.
		</p>
	</Modal>
	<Modal
		bind:open={mdalBG1}
		modalHeading="Report a problem"
		primaryButtonText="Submit"
		on:click:button--primary={() => (mdalBG1 = false)}
	>
		<br />
		<p>
			We intend to make Project JOAN a cohesive and effective application. We value your feedback
			regarding problems you face while using the application.
		</p>
		<br />
		<h6 class="flex flex-col">Tell us more about your problem.</h6>
		<br />
		<div>
			<CodeSnippet type="inline" code={pageID} />
		</div>
		<br />
		<FluidForm class="flex flex-col lg:flex-row">
			<TextInput
				labelText="Problem Title"
				placeholder="Provide a short description of the problem you encountered"
				required
			/>
		</FluidForm>
		<FluidForm class="flex flex-col lg:flex-row">
			<TextArea
				placeholder="Provide an in-depth description of the problem you encountered"
				required
			/>
		</FluidForm>
	</Modal>
{:else}
	<Header company="Project" platformName="JOAN" href="/">
		<div class="text-white hidden lg:flex">Colegio de San Juan de Dios, Incorporated</div>
		<div class="text-white flex lg:hidden"><strong>CSJDD</strong></div>
		<HeaderUtilities>
			<div class="flex">
				<div class="flex">
					<Button
						tooltipAlignment="end"
						tooltipPosition="bottom"
						iconDescription="Report a problem"
						kind="secondary"
						icon={Debug}
						on:click={() => (mdalBG1 = true)}
					/>
				</div>
			</div>
		</HeaderUtilities>
	</Header>
	<div class="flex flex-col items-stretch lg:flex-row">
		<!-- displayed on mobile -->
		<Content id="#" class="flex flex-col bg-neutral-900 h-60 lg:hidden">
			<ProgressBar size="md" helperText={progTX} />
		</Content>
		<Content id="#" class="bg-neutral-900 w-1/3 h-screen hidden lg:flex" />
		<Content class="self-center w-11/12 lg:w-2/5">
			<ProgressBar size="md" helperText={progTX} />
		</Content>
		<Content />
	</div>
{/if}
