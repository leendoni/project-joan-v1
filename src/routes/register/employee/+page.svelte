<script>
	import { goto } from '$app/navigation';
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
		RadioButton,
		RadioButtonGroup,
		RadioTile,
		TextInput,
		TileGroup
	} from 'carbon-components-svelte';

	import { Debug, GenderFemale, GenderMale, Pen, Return } from 'carbon-icons-svelte';

	// variables

	// for gender
	let male = true;
	let female = true;

	function isMale() {
		male = true;
		female = false;
	}

	function isFemale() {
		male = false;
		female = true;
	}

	// for confirmation checkbox
	let confirmEnabled = false;

	//for closing the modal / redirecting to home
	let open = false;

	function goHome() {
		open = false;
		goto('/login/employee');
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
					href="/bugs/report"
					tooltipAlignment="end"
					tooltipPosition="bottom"
					iconDescription="Report a problem"
					kind="secondary"
					icon={Debug}
				/>
				<Button
					href="/login/employee"
					tooltipPosition="left"
					iconDescription="Return to login"
					kind="primary"
					icon={Return}
				/>
			</div>
		</div>
	</HeaderUtilities>
</Header>

<!-- adapts depending on the device -->
<div class="flex flex-col items-stretch">
	<!-- displayed on mobile -->
	<Content id="#" class="flex flex-col bg-neutral-900 h-72">
		<h1 class="pt-28 lg:pt-44 text-white">Employee Registration</h1>
	</Content>
	<Content class="h-full w-full">
		<Form on:submit>
			<FormGroup
				legendText="For aspiring students, please provide us with your personal information."
			>
				<br />
				<RadioButtonGroup legendText="Pick your provided government ID" selected="">
                    <RadioButton labelText="PRC ID" value="PRC" />
                    <RadioButton labelText="Driver's License" value="LTO" />
                    <RadioButton labelText="National ID" value="NAT" />
                    <RadioButton labelText="PhilHealth ID" value="PHL" />
                    <RadioButton labelText="Passport" value="PPT" />
                </RadioButtonGroup>
                <br>
				<FluidForm class="flex flex-col lg:flex-row">
					<TextInput labelText="Government ID Number" placeholder="Enter your ID number" required />
					<TextInput labelText="Expiration Date" placeholder="Expiration Date" required />
				</FluidForm>
				<br />
				<FluidForm class="flex flex-col lg:flex-row">
					<TextInput labelText="Last Name" placeholder="Your last name" required />
					<TextInput labelText="First Name" placeholder="Your first name" required />
					<TextInput labelText="Middle Name" placeholder="Your middle name" required />
					<TextInput labelText="Suffix (if any)" placeholder="Sr., Jr., III., etc." required />
					<ButtonSet class="flex flex-row lg:flex-col">
						<div class="w-1/2 lg:w-full">
							<Button disabled={!male} size="small" kind="secondary" on:click={isMale}>Male</Button>
						</div>
						<div class="w-1/2 lg:w-full">
							<Button disabled={!female} size="small" kind="secondary" on:click={isFemale}
								>Female</Button
							>
						</div>
					</ButtonSet>
				</FluidForm>
				<FluidForm class="flex flex-col lg:flex-row">
					<TextInput labelText="Contact Number" placeholder="Your contact number" required />
					<TextInput labelText="Email Address" placeholder="Your email address" required />
					<TextInput labelText="Employee Address" placeholder="Your current address" required />
				</FluidForm>
				<FluidForm class="flex flex-col lg:flex-row">
					<TextInput labelText="Contact Person" placeholder="Your contact person's name" required />
					<TextInput labelText="Relation (optional)" placeholder="Your relation to your contact person" required />
                    <TextInput
						labelText="Emergency Contact Number"
						placeholder="Your emergency contact number"
						required
					/>
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
	modalHeading="Account Registered!"
	primaryButtonText="Return to login"
	on:click:button--primary={goHome}
>
	<div class="flex flex-col content-center">
		<div>
			<br />
			<p>Below are your account details:</p>
			<br />
			<p class="italic">Username:</p>
			<h5>lobramonte.31859</h5>
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
