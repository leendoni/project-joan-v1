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
		Dropdown,
		FluidForm,
		Form,
		FormGroup,
		Header,
		HeaderUtilities,
		Link,
		Modal,
		PasswordInput,
		ProgressBar,
		RadioButton,
		RadioButtonGroup,
		TextArea,
		TextInput
	} from 'carbon-components-svelte';

	import {
		Carbon,
		Debug,
		HelpFilled,
		Login,
		WatsonHealthCrossReference
	} from 'carbon-icons-svelte';

	// pictograms

	// variables
	let mdalPW1 = false;
	let mdalPW2 = false;
	let mdalAB1 = false;
	let mdalBG1 = false;
	let confirmEnabled = false;

	let pageID = '/login/employee';
	let progTX = '';

	function handleLogin() {}

	let connected = false;

	onMount(async () => {
		try {
			// Try to fetch documents from a test collection (could be any collection you know exists)
			const getSchoolID = doc(db, 'schools', '0303001');
			progTX='Retrieving School ID'
			const getUsers = collection(getSchoolID, 'users');
			progTX='Connecting to User Database'
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

	<!-- adapts depending on the device -->
	<div class="flex flex-col items-stretch lg:flex-row">
		<!-- displayed on mobile -->
		<Content id="#" class="flex flex-col bg-neutral-900 h-60 lg:hidden">
			<h1 class="pt-32 text-white">Employee Log-in</h1>
		</Content>
		<Content id="#" class="bg-neutral-900 w-1/3 h-screen hidden lg:flex" />
		<Content class="self-center w-11/12 lg:w-2/5">
			<h1 class="pb-12 hidden lg:flex">Employee Log-in</h1>
			<Form on:submit>
				<FormGroup legendText="Enter your account information.">
					<FluidForm>
						<div class="flex" />
						<TextInput labelText="Username" placeholder="Enter your username" required />
						<PasswordInput
							required
							type="password"
							labelText="Password"
							placeholder="Enter your password"
						/>
					</FluidForm>
				</FormGroup>
				<ButtonSet stacked>
					<Button on:click={handleLogin} icon={Login}>Log in</Button>
					<Button on:click={() => (mdalPW1 = true)} kind="ghost">Forgot password?</Button>
				</ButtonSet>
				<br />
			</Form>
			<br /><br />
			<div>
				New employee?
				<Link href="/registration/employee" icon={Carbon}>Register here.</Link>.
			</div>
		</Content>
	</div>

	<Modal
		bind:open={mdalPW1}
		modalHeading="Reset your password"
		primaryButtonText="Confirm"
		primaryButtonDisabled={!confirmEnabled}
		secondaryButtonText="Cancel"
		selectorPrimaryFocus="#db-name"
		on:click:button--secondary={() => (mdalPW1 = false)}
		on:click:button--primary={() => (mdalPW2 = true)}
		on:click:button--primary={() => (mdalPW1 = false)}
	>
		<br />
		<p>To reset your account, provide the following information:</p>
		<br />
		<FluidForm class="flex flex-col lg:flex-row">
			<TextInput labelText="Account Code" placeholder="Enter your account code" required />
			<Button size="field">Search</Button>
		</FluidForm>
		<br />
		<FluidForm class="flex flex-col lg:flex-row">
			<TextInput readonly labelText="Last Name" placeholder="Your last name" required />
			<TextInput readonly labelText="First Name" placeholder="Your first name" required />
			<TextInput readonly labelText="Middle Name" placeholder="Your middle name" required />
		</FluidForm>
		<br />
		<Checkbox bind:checked={confirmEnabled} labelText="The information above is correct." />
	</Modal>
	<Modal
		bind:open={mdalPW2}
		modalHeading="Reset your password"
		primaryButtonText="Return to Login"
		primaryButtonDisabled={!confirmEnabled}
		on:click:button--primary={() => (mdalPW2 = false)}
	>
		<br />
		<h5>A notification to reset your account has been sent to your System Administrator.</h5>
		<br />
		<h5 class="italic">Temporary Password:</h5>
		<h3>csjd.reset123</h3>
		<br />
		<p class="italic">
			Using the temporary password, try logging in to your account within 1-2 days from resetting
			your password. If the temporary password does not work within that period, contact your system
			administrator for a follow-up reset.
		</p>
	</Modal>
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
