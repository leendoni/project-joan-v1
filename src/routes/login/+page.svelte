<script>
	// #region imports //

	// for firebase
	import { addDoc, collection, doc, getDocs, query, updateDoc, where } from 'firebase/firestore';
	import { onMount } from 'svelte';
	import { db } from '../../firebase';
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
	import { goto } from '$app/navigation';
	import { Carbon, Debug, Login } from 'carbon-icons-svelte';
	// #endregion //

	// #region database variables, fields
	let pageID = '/login/student';

	// system
	let systID = '', // campus code
		systDT = '', // system date + time
		systST = ''; // status

	// modules
	let modlID = '', // module id
		modlNM = '', // module name
		modlON = ''; // active inactive

	// academics
	let acadYR = '', // active acad year
		acadST = '', // start date
		acadND = '', // end date
		acadSM = '', // semester
		acadGD = ''; // period

	// users
	let userID = '', // user ID
		userCL = '', // class
		userUN = '', // username
		userPW = '', // password
		userLR = '', // LRN
		userLN = '', // last name
		userFN = '', // first name
		userMN = '', // middle name
		userSF = '', // suffix
		userGD = '', // gender
		userCN = '', // contact num
		userAD = '', // address
		userFA = '', // father
		userMA = '', // mother
		userGA = '', // guardian
		userCP = '', // contact person
		userCR = '', // contact relation
		userEC = '', // emergency contact
		userAY = '', // current acad year
		userSM = '', // current semester
		userYR = '', // current year
		userSC = '', // current section
		userSD = '', // schedule assignment (for personnel)
		userAV = ''; // user average
	let userON = '', // active, inactive
		userHS = '', // has suffix
		userRP = '', // reset password
		userUC = '', // unsaved changes
		userR1 = '', // requirements (up to 9)
		userR2 = '', // requirements (up to 9)
		userR3 = '', // requirements (up to 9)
		userR4 = '', // requirements (up to 9)
		userR5 = '', // requirements (up to 9)
		userR6 = '', // requirements (up to 9)
		userR7 = '', // requirements (up to 9)
		userR8 = '', // requirements (up to 9)
		userR9 = ''; // requirements (up to 9)

	// population
	let totlRG = ''; // total registered
	let totY01 = '', // total per year level (up to 12)
		totY02 = '', // total per year level (up to 12)
		totY03 = '', // total per year level (up to 12)
		totY04 = '', // total per year level (up to 12)
		totY05 = '', // total per year level (up to 12)
		totY06 = '', // total per year level (up to 12)
		totY07 = '', // total per year level (up to 12)
		totY08 = '', // total per year level (up to 12)
		totY09 = '', // total per year level (up to 12)
		totY10 = '', // total per year level (up to 12)
		totY11 = '', // total per year level (up to 12)
		totY12 = ''; // total per year level (up to 12)
	let totM01 = '', // total male per year
		totM02 = '', // total male per year
		totM03 = '', // total male per year
		totM04 = '', // total male per year
		totM05 = '', // total male per year
		totM06 = '', // total male per year
		totM07 = '', // total male per year
		totM08 = '', // total male per year
		totM09 = '', // total male per year
		totM10 = '', // total male per year
		totM11 = '', // total male per year
		totM12 = ''; // total male per year
	let totF01 = '', // total female per year
		totF02 = '', // total female per year
		totF03 = '', // total female per year
		totF04 = '', // total female per year
		totF05 = '', // total female per year
		totF06 = '', // total female per year
		totF07 = '', // total female per year
		totF08 = '', // total female per year
		totF09 = '', // total female per year
		totF10 = '', // total female per year
		totF11 = '', // total female per year
		totF12 = ''; // total female per year

	// subjects
	let subjID = '', // subject id
		subjNM = '', // subject name
		subjTC = '', // teacher (take id)
		subjAY = ''; // acad year of subject

	// financials
	let tranID = '', // transaction id
		tranNM = '', // transaction name
		tranDC = '', // description
		tranAY = '', // acad year
		tranAM = '', // amount needed
		tranPY = ''; // payor. put grade levels here

	// transactions
	let tranCD = '', // transaction code (after pay)
		tranPB = '', // paid by (put userID to log)
		tranAT = '', // amount tendered
		tranAC = '', // amount change
		tranVD = ''; // void (if refunded)

	// schedules
	let schdID = '', // schedule id (assign to sections and teachers)
		schdAY = ''; // acad year

	// campus bulletin
	let postID = '', // post id
		postTT = '', // title
		postDC = '', // description
		postBY = '', // take userID
		postVW = '', // only viewable by? put class
		postSD = '', // start date
		postED = '', // end date
		postAY = ''; // acad year

	// gradebook
	let gradOF = '', // load userID here
		gradON = '', // load subjID here
		gradAY = '', // academic year
		gradQ1 = '', // quarters
		gradFN = ''; // final grade

	// problems
	let probID = '', // report ID
		probTT = '', // title
		probDC = '', // description
		probST = ''; // status

	// locals
	let logdID = '', // logged in user ID
		logdCL = '', // logged in class
		logdFN = ''; // logged in first name
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
	let mdalON1 = false;
	let confirm = false;

	// login
	let nputUN = '',
		nputPW = '',
		nputID = '';
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

			userON = userData.userON;

			if (userON == 'ACTIVE') {
				// Check the password using the function from before
				const isPasswordValid = await bcrypt.compare(nputPW, userData.userPW);

				if (isPasswordValid) {
					userID = userData.userID;
					userCL = userData.userCL;
					userFN = userData.userFN;

					localStorage.setItem('userID', userID);
					localStorage.setItem('userCL', userCL);
					localStorage.setItem('userFN', userFN);

					log(nputUN, true);

					if (userCL == 'god') {
						goto('/godmode');
					} else if (userCL == 'employee') {
						goto('/admin');
					} else if (userCL == 'student') {
						goto('/home');
					}
				} else {
					invaPW = true;
				}
			} else {
				mdalON1 = true;
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
		const q = query(getUsers, where('userID', '==', nputID));

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

		const q = query(getUsers, where('userID', '==', nputID));
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
</script>

{#if dtbsCN}
	<Header company="Project" platformName="JOAN" href="/">
		<div class="text-white hidden lg:flex">Colegio de San Juan de Dios, Incorporated</div>
		<div class="text-white flex lg:hidden"><strong>CSJDD</strong></div>
		<HeaderUtilities>
			<div class="flex">
				<div class="flex">
					<Button
						tooltipPosition="left"
						iconDescription="Report a problem"
						kind="ghost"
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
			<h1 class="pt-32 text-white">Account Log-in</h1>
		</Content>
		<Content id="#" class="bg-neutral-900 w-1/3 h-screen hidden lg:flex" />
		<Content class="self-center w-11/12 lg:w-2/5">
			<h1 class="pb-12 hidden lg:flex">Account Log-in</h1>
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
							tooltipPosition="left"
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
						tooltipPosition="left"
						iconDescription="Report a problem"
						kind="ghost"
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
			bind:value={nputID}
			labelText="Account ID"
			placeholder="Enter your account ID"
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
<Modal
	bind:open={mdalON1}
	modalHeading="Account Inactive"
	primaryButtonText="Close"
	on:click:button--primary={() => (mdalON1 = false)}
>
	<br />
	<h5>Your account is not yet activated. Enroll to activate / reactivate your account.</h5>
	<br />
	<p class="italic">
		To ensure system efficiency, student accounts get reset every semester. You must first enroll to
		the current semester to regain access to your account.
	</p>
</Modal>
