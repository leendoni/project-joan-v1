<script>
	import { collection, getDocs } from 'firebase/firestore';
	import { onMount } from 'svelte';
	import { db } from '../../../firebase';

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
		Modal,
		TextInput
	} from 'carbon-components-svelte';
	// icons
	import { HelpFilled, Login, NotebookReference, Pen, TestTool } from 'carbon-icons-svelte';
	import { goto } from '$app/navigation';

	// pictograms

	// variables
	let open = false;
	let confirmEnabled = false;

	// for confirmation checkbox
	function toggleButtonState() {
		confirmEnabled = !confirmEnabled;
	}

	//for closing the modal / redirecting to home
	function goHome() {
		open = false;
		goto('/');
	}
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
					href="/test/"
					tooltipPosition="left"
					iconDescription="Testing Tools"
					kind="secondary"
					icon={TestTool}
				/>
				<Button
					tooltipPosition="left"
					iconDescription="About JOAN"
					kind="secondary"
					icon={HelpFilled}
				/>
				<Button
					tooltipPosition="left"
					iconDescription="System Guide"
					kind="primary"
					icon={NotebookReference}
				/>
			</div>
		</div>
	</HeaderUtilities>
</Header>

<!-- adapts depending on the device -->
<div class="flex flex-col items-stretch">
	<!-- displayed on mobile -->
	<Content id="#" class="flex flex-col bg-neutral-900 h-72">
		<h1 class="pt-28 lg:pt-44 text-white">Account Registration</h1>
	</Content>
	<Content class="h-full w-full">
		<Form on:submit>
			<FormGroup
				legendText="For aspiring students, please provide us with your personal information."
			>
				<br />
				<FluidForm class="flex flex-col lg:flex-row">
					<TextInput readonly labelText="Academic Year" placeholder="2023-2024" required />
					<TextInput readonly labelText="Semester" placeholder="Second" required />
					<TextInput labelText="Grade Level" placeholder="1, 2, 3, etc." required />
					<TextInput labelText="Learner's Reference Number" placeholder="Enter your LRN" required />
				</FluidForm>
				<br />
				<FluidForm class="flex flex-col lg:flex-row">
					<TextInput labelText="Last Name" placeholder="Your last name" required />
					<TextInput labelText="First Name" placeholder="Your first name" required />
					<TextInput labelText="Middle Name" placeholder="Your middle name" required />
					<TextInput labelText="Suffix (if any)" placeholder="Sr., Jr., III., etc." required />
				</FluidForm>
				<FluidForm class="flex flex-col lg:flex-row">
					<TextInput labelText="Contact Number" placeholder="Your contact number" required />
					<TextInput labelText="Student Address" placeholder="Your current address" required />
				</FluidForm>
				<FluidForm class="flex flex-col lg:flex-row">
					<TextInput labelText="Mother's Name" placeholder="Your mother's name" required />
					<TextInput labelText="Father's Name" placeholder="Your father's name" required />
					<TextInput labelText="Guardian's Name (optional)" placeholder="Your guardian's name" />
				</FluidForm>
			</FormGroup>
			<ButtonSet class="flex flex-col gap-2 lg:flex-row">
				<Checkbox bind:checked={confirmEnabled} labelText="The information above is correct." />
				<Button disabled={!confirmEnabled} on:click={() => (open = true)} icon={Pen}
					>Register</Button
				>
			</ButtonSet>
		</Form>
	</Content>
</div>

<Modal
	bind:open
	modalHeading="JOAN Account Registered"
	primaryButtonText="Return to login"
	on:click:button--primary={goHome}
>
	<div class="flex flex-col content-center">
		<div>
			<br />
			<p>Below are your account details:</p>
			<br />
			<p class="italic">Username:</p>
			<h5>Lobramonte.031859@csjd.project-joan.cloud</h5>
			<br />
			<p class="italic">Password:</p>
			<h5>Lobramonte031859</h5>
			<br />
			<p class="italic">Account Code:</p>
			<h3>HHD98H</h3>
			<br />
		</div>
		<div>
			<p class="italic">
				Important Note:<br /><strong>Take note of your account code</strong> as this will be used
				during the enrollment process. Also, <strong>immediately change your password</strong> upon gaining
				access to your account.
			</p>
		</div>
	</div>
</Modal>

<!-- {:else}
	
{/if} -->
