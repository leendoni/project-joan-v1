<script>
	// #region imports //

	// for firebase
	import { addDoc, collection, doc, getDocs, query, updateDoc, where } from 'firebase/firestore';
	import { onMount } from 'svelte';
	import { db } from '../../../firebase';

	// for passwords
	import bcrypt from 'bcryptjs';

	// carbon components
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
		ProgressBar,
		TextArea,
		TextInput
	} from 'carbon-components-svelte';

	// carbon icons
	import { Carbon, Debug, Login, TemperatureFahrenheit } from 'carbon-icons-svelte';
	import { goto } from '$app/navigation';
	// #endregion //

	// #region database variables, fields
	let pageID = '/login/student';

	// system variables
	let systID = '',
		systDT = '',
		systST = '',
		systLD = '';

	// modules
	let modlID = '',
		modlNM = '',
		modlRQ = '',
		modlON = '';

	// academics
	let acadYR = '',
		acadST = '',
		acadND = '',
		acadSM = '',
		acadGD = '';

	// users
	let userID = '',
		userCL = '',
		userUN = '',
		userPW = '',
		userAC = '',
		userLR = '',
		userLN = '',
		userFN = '',
		userMN = '',
		userSF = '',
		userGD = '',
		userCN = '',
		userAD = '',
		userFA = '',
		userMA = '',
		userGA = '',
		userCP = '',
		userCR = '',
		userEC = '',
		userYR = '',
		userSC = '',
		userUS = '',
		userSD = '',
		userAV = '';

	let userON = '',
		userHS = '',
		userRP = '',
		userUC = '',
		userR1 = '',
		userR2 = '',
		userR3 = '',
		userR4 = '',
		userR5 = '',
		userR6 = '';

	// population
	let totlSP = '',
		totlRG = '';

	let totY01 = '',
		totY02 = '',
		totY03 = '',
		totY04 = '',
		totY05 = '',
		totY06 = '',
		totY07 = '',
		totY08 = '',
		totY09 = '',
		totY10 = '',
		totY11 = '',
		totY12 = '';

	let totM01 = '',
		totM02 = '',
		totM03 = '',
		totM04 = '',
		totM05 = '',
		totM06 = '',
		totM07 = '',
		totM08 = '',
		totM09 = '',
		totM10 = '',
		totM11 = '',
		totM12 = '';

	let totF01 = '',
		totF02 = '',
		totF03 = '',
		totF04 = '',
		totF05 = '',
		totF06 = '',
		totF07 = '',
		totF08 = '',
		totF09 = '',
		totF10 = '',
		totF11 = '',
		totF12 = '';

	// subjects
	let subjID = '',
		subjNM = '',
		subjTC = '';

	// financials
	let tranID = '',
		tranNM = '',
		tranDC = '',
		tranAM = '',
		tranPY = '';

	let tranCD = '',
		tranPB = '',
		tranAT = '',
		tranAC = '',
		tranVD = '';

	// schedules
	let schdID = '';

	// campus bulletin
	let postID = '',
		postTT = '',
		postDC = '',
		postBY = '',
		postVW = '',
		postSD = '',
		postED = '';

	// gradebook
	let gradOF = '',
		gradON = '',
		gradAY = '',
		gradQT = '';

	// problems
	let probID = '',
		probTT = '',
		probDC = '',
		probST = '';

	// locals
	let logdAC = '',
		logdCL = '',
		logdFN = '';
	// #endregion

	// #region functional variables
	// randoms
	let dtbsCN = false; // database connection
	let progTX = ''; // progressbar text

	// modals
	let mdalPW1 = false,
		mdalPW2 = false;
	let mdalAB1 = false,
		mdalBG1 = false;
	let confirm = false;

	// login
	let nputUN = '',
		nputPW = '',
		nputLR = '';
	let invaUN = false,
		invaPW = false;

	// database values
	const getSchoolID = doc(db, 'schools', '0303001');
	const getUsers = collection(getSchoolID, 'users');
	// #endregion

	// #region login functions
	// main function
	async function login() {
		try {
			// Fetch the user's data from Firestore using the entered username
			const q = query(getUsers, where('userUN', '==', nputUN));

			const snapshot = await getDocs(q);

			if (snapshot.empty) {
				invaUN = true;
				log(nputUN, false);
				return;
			}

			const userData = snapshot.docs[0].data();

			userAC = userData.userAC;
			userCL = userData.userCL;
			userFN = userData.userFN;

			console.log(userAC);
			console.log(userCL);
			console.log(userFN);

			localStorage.setItem('userAC', userAC);
			localStorage.setItem('userCL', userCL);
			localStorage.setItem('userFN', userFN);

			// Check the password using the function from before
			const isPasswordValid = await bcrypt.compare(nputPW, userData.userPW);

			if (isPasswordValid) {
				log(nputUN, true);
				goto('/godmode/');
			} else {
				invaPW = true;
			}
		} catch (error) {
			log(nputUN, false);
			console.error('Login error:', error);
			alert('An error occurred during login. Please try again.');
		}
	}
	// access logging
	async function log(userUN, aclgST) {
		try {
			const formattedDate = date(new Date());
			const data = {
				userUN,
				aclgST,
				aclgDT: formattedDate
			};

			await addDoc(collection(db, 'schools', '0303001', 'access'), data);
		} catch (error) {
			console.error('Failed to log access:', error);
		}
	}
	// date formatting for access logs
	function date(date) {
		const YYYY = date.getFullYear();
		const MM = String(date.getMonth() + 1).padStart(2, '0'); // Months are 0-indexed in JavaScript
		const DD = String(date.getDate()).padStart(2, '0');
		const HH = String(date.getHours()).padStart(2, '0');
		const MIN = String(date.getMinutes()).padStart(2, '0');
		const SS = String(date.getSeconds()).padStart(2, '0');

		return `${YYYY}${MM}${DD}-${HH}:${MIN}:${SS}`;
	}
	// #endregion

	// #region password reset functions
	// load user data
	async function loadUserData() {
		// Fetch the user's data from Firestore using the entered username
		const q = query(getUsers, where('userLR', '==', nputLR));

		const snapshot = await getDocs(q);

		if (snapshot.empty) {
			return;
		}

		const userData = snapshot.docs[0].data();

		console.log(userLN);

		userLN = userData.userLN;
		userFN = userData.userFN;
		userMN = userData.userMN;
	}
	// update user data
	async function updateUserData() {
		const updatedData = {
			userRP: 'true'
		};

		const q = query(getUsers, where('userLR', '==', nputLR));
		const snapshot = await getDocs(q);

		if (snapshot.empty) {
			console.error('No records found for the selected reference code.');
			return;
		}

		const docId = snapshot.docs[0].id;
		const docRef = doc(db, 'schools', '0303001', 'users', docId);

		try {
			await updateDoc(docRef, updatedData);
			console.log('Successfully updated');
		} catch (error) {
			console.error('Error updating document:', error);
		}
	}
	// #endregion
	onMount(async () => {
		try {
			// Try to fetch documents from a test collection (could be any collection you know exists)
			const getSchoolID = doc(db, 'schools', '0303001');
			progTX = 'Retrieving...';
			const getUsers = collection(getSchoolID, 'users');
			progTX = 'Connecting...';
			await getDocs(getUsers);
			console.log('Connected.');
			dtbsCN = true;
		} catch (error) {
			console.error('Failed. :', error);
			dtbsCN = false;
		}
	});
	// #endregion
</script>

{#if dtbsCN}
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
			<h1 class="pt-32 text-white">Student Log-in</h1>
		</Content>
		<Content id="#" class="bg-neutral-900 w-1/3 h-screen hidden lg:flex" />
		<Content class="self-center w-11/12 lg:w-2/5">
			<h1 class="pb-12 hidden lg:flex">Student Log-in</h1>
			<Form on:submit>
				<div class="flex flex-col gap-6">
					<div>
						<TextInput
							bind:value={nputUN}
							invalid={invaUN}
							placeholder="Username"
							size="xl"
							required
						/>
						<PasswordInput
							bind:value={nputPW}
							invalid={invaPW}
							type="password"
							placeholder="Password"
							size="xl"
							required
						/>
					</div>
					<ButtonSet stacked>
						<Button on:click={login} icon={Login}>Log in</Button>
						<Button on:click={() => (mdalPW1 = true)} kind="ghost">Forgot password?</Button>
					</ButtonSet>
					<br />
				</div>
			</Form>
			<br /><br />
			<div>
				New student?
				<Link href="/register/student" icon={Carbon}>Register here.</Link>.
			</div>
		</Content>
	</div>
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

<!-- modals -->
<Modal
	bind:open={mdalPW1}
	modalHeading="Reset your password"
	primaryButtonText="Confirm"
	primaryButtonDisabled={!confirm}
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
		<TextInput
			bind:value={nputLR}
			labelText="Learner's Reference Number"
			placeholder="Enter your LRN"
			required
		/>
		<Button size="field" on:click={loadUserData}>Search</Button>
	</FluidForm>
	<br />
	<FluidForm class="flex flex-col lg:flex-row">
		<TextInput
			bind:value={userLN}
			readonly
			labelText="Last Name"
			placeholder="Your last name"
			required
		/>
		<TextInput
			bind:value={userFN}
			readonly
			labelText="First Name"
			placeholder="Your first name"
			required
		/>
		<TextInput
			bind:value={userMN}
			readonly
			labelText="Middle Name"
			placeholder="Your middle name"
			required
		/>
	</FluidForm>
	<br />
	<Checkbox bind:checked={confirm} labelText="The information above is correct." />
</Modal>
<Modal
	bind:open={mdalPW2}
	on:open={updateUserData}
	modalHeading="Reset your password"
	primaryButtonText="Return to Login"
	primaryButtonDisabled={!confirm}
	on:click:button--primary={() => (mdalPW2 = false)}
>
	<br />
	<h5>A notification to reset your account has been sent to your System Administrator.</h5>
	<br />
	<h5 class="italic">Temporary Password:</h5>
	<h3>csjd.reset123</h3>
	<br />
	<p class="italic">
		Using the temporary password, try logging in to your account within 1-2 days from resetting your
		password. If the temporary password does not work within that period, contact your system
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
		and secure backbone of Firebase, Project JOAN is a web-based platform dedicated to streamlining
		academic, financial, and administrative processes. Designed with precision, Project JOAN is a
		testament to efficiency, user-friendliness, and unwavering data security.
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
	<FluidForm class="flex flex-col lg:flex-row">
		<TextInput bind:value={pageID} labelText="Page Link" placeholder={pageID} readonly />
		<TextInput
			bind:value={probID}
			labelText="Problem ID"
			placeholder="Provide a short description of the problem"
			readonly
		/>
	</FluidForm>
	<FluidForm class="flex flex-col lg:flex-row">
		<TextInput
			bind:value={probTT}
			labelText="Problem Title"
			placeholder="Provide a short description of the problem you encountered"
			required
		/>
	</FluidForm>
	<FluidForm class="flex flex-col lg:flex-row">
		<TextArea
			placeholder="Provide an in-depth description of the problem you encountered"
			bind:value={probDC}
			required
		/>
	</FluidForm>
</Modal>
