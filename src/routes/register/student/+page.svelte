<script>
	// #region imports
	// for firebase
	import { goto } from '$app/navigation';
	import { addDoc, collection, doc, getDocs, query, where } from 'firebase/firestore';
	import { db } from '../../../firebase';

	import { onMount } from 'svelte';

	// for passwords
	import bcrypt from 'bcryptjs';

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
		PasswordInput,
		ProgressBar,
		TextArea,
		TextInput
	} from 'carbon-components-svelte';

	import { Debug, GenderFemale, GenderMale, Pen, Return } from 'carbon-icons-svelte';
	// #endregion

	// #region database variables, fields
	let pageID = '/register/student';

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
	let logdID = '',
		logdCL = '',
		logdFN = '';
	// #endregion

	// #region functional variables
	// randoms
	let dtbsCN = false; // database connection
	let progTX = ''; // progressbar text

	let male = true,
		female = true; // for gender

	let confirm = false,
		duplLR = false; // for confirmation checkbox

	// modals
	let mdalRG1 = false;
	let mdalRG2 = false;
	let mdalRG3 = false;
	let mdalBG1 = false;
	// #endregion

	// #region functions
	// #region general
	function isMale() {
		male = true;
		female = false;
		userGD = 'Male';
	}

	function isFemale() {
		male = false;
		female = true;
		userGD = 'Female';
	}

	function goHome() {
		mdalRG1 = false;
		goto('/login/student');
	}

	function clear() {
		userUN = '';
		userPW = '';

		male = true;
		female = true;

		userLR = '';
		userLN = '';
		userFN = '';
		userMN = '';
		userSF = '';
		userCN = '';
		userAD = '';
		userMA = '';
		userFA = '';
		userGA = '';
		userEC = '';
	}
	// #endregion
	// #region generators
	function generateAC() {
		const characters = 'ABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789';
		let result = '';
		for (let i = 0; i < 6; i++) {
			result += characters.charAt(Math.floor(Math.random() * characters.length));
		}
		userAC = result;
	}

	function generateUN() {
		const trimLR = userLR.slice(-6); // Get the last 6 characters of the LRN
		userUN = `${userLN}.${trimLR}@csjd.project-joan.cloud`;
	}

	async function generatePW(userPW) {
		const salt = await bcrypt.genSalt(10); // Generate a salt
		const hashedPassword = await bcrypt.hash(userPW, salt); // Hash the password
		return hashedPassword;
	}

	// #endregion
	// #region checkers
	async function checkDuplicateLRN(field, value) {
		const getSchoolID = doc(db, 'schools', '0303001');
		const getUsers = collection(getSchoolID, 'users');
		const q = query(getUsers, where(field, '==', value));
		const snapshot = await getDocs(q);
		return !snapshot.empty; // Returns true if the entry exists
	}
	// #endregion
	// #region handlers
	async function register(event) {
		duplLR = await checkDuplicateLRN('userLR', userLR);

		let fillST, xistST;

		if (
			userLR == '' ||
			userLN == '' ||
			userFN == '' ||
			userMN == '' ||
			userGD == '' ||
			userCN == '' ||
			userAD == '' ||
			userMA == '' ||
			userFA == '' ||
			userEC == '' ||
			userUN == '' ||
			userPW == '' ||
			userAC == ''
		) {
			confirm = false;
			fillST = false;
			mdalRG3 = true; // fill all modal
		} else {
			fillST = true;
		}

		if (duplLR) {
			confirm = false;
			xistST = true;
			mdalRG2 = true; // exists modal
		} else {
			xistST = false;
		}

		if (fillST && !xistST) {
			confirm = false;
			mdalRG1 = true; // success modal

			event.preventDefault();

			const hashedPassword = await generatePW(userPW);

			console.log(duplLR);

			// Construct the data object
			const formData = {
				acadYR: '2023-2024',
				acadSM: 'Second',
				userLR,
				userLN,
				userFN,
				userMN,
				userSF,
				userGD,
				userCN,
				userAD,
				userMA,
				userFA,
				userGA,
				userEC,
				userUN,
				userPW: hashedPassword,
				userAC,
				userON,
				userRP
			};

			// check duplicates
			if (duplLR) {
				return; // Don't proceed with saving the data if there are duplicates
			} else {
				try {
					await upload('0303001', formData);
				} catch (e) {
					console.log('Failed to save data. Please try again.');
				}
			}
		}
	}
	async function upload(schoolCode, data) {
		try {
			const docRef = await addDoc(collection(db, 'schools', schoolCode, 'users'), data);
			console.log('Document written with ID: ', docRef.id);
			return docRef.id; // you can return the ID to further use it if needed
		} catch (e) {
			console.error('Error adding document: ', e);
			throw e; // re-throwing the error for the caller to handle
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
					<Button
						href="/login/student"
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
			<h1 class="pt-28 lg:pt-44 text-white">Student Registration</h1>
		</Content>
		<Content class="h-full w-full">
			<Form on:submit>
				<FormGroup
					legendText="For aspiring students, please provide us with your personal information."
				>
					<br />
					<FluidForm class="flex flex-col lg:flex-row">
						<TextInput
							bind:value={acadYR}
							readonly
							labelText="Academic Year"
							placeholder="2023-2024"
							required
						/>
						<TextInput
							bind:value={acadSM}
							readonly
							labelText="Semester"
							placeholder="Second"
							required
						/>
						<TextInput
							bind:value={userLR}
							labelText="Learner's Reference Number"
							placeholder="Enter your LRN"
							required
						/>
					</FluidForm>
					<br />
					<FluidForm class="flex flex-col lg:flex-row">
						<TextInput
							bind:value={userLN}
							labelText="Last Name"
							placeholder="Your last name"
							required
						/>
						<TextInput
							bind:value={userFN}
							labelText="First Name"
							placeholder="Your first name"
							required
						/>
						<TextInput
							bind:value={userMN}
							labelText="Middle Name"
							placeholder="Your middle name"
							required
						/>
						<TextInput
							bind:value={userSF}
							labelText="Suffix (if any)"
							placeholder="Sr., Jr., III., etc."
							required
						/>
						<ButtonSet class="flex flex-row lg:flex-col">
							<div class="w-1/2 lg:w-full">
								<Button
									disabled={!male}
									size="small"
									kind="secondary"
									icon={GenderMale}
									on:click={isMale}>Male</Button
								>
							</div>
							<div class="w-1/2 lg:w-full">
								<Button
									disabled={!female}
									size="small"
									kind="secondary"
									icon={GenderFemale}
									on:click={isFemale}>Female</Button
								>
							</div>
						</ButtonSet>
					</FluidForm>
					<FluidForm class="flex flex-col lg:flex-row">
						<TextInput
							bind:value={userCN}
							labelText="Contact Number"
							placeholder="Your contact number"
							required
						/>
						<TextInput
							bind:value={userAD}
							labelText="Student Address"
							placeholder="Your current address"
							required
						/>
					</FluidForm>
					<FluidForm class="flex flex-col lg:flex-row">
						<TextInput
							bind:value={userMA}
							labelText="Mother's Name"
							placeholder="Your mother's name"
							required
						/>
						<TextInput
							bind:value={userFA}
							labelText="Father's Name"
							placeholder="Your father's name"
							required
						/>
						<TextInput
							bind:value={userGA}
							labelText="Guardian's Name (optional)"
							placeholder="Your guardian's name"
						/>
						<TextInput
							bind:value={userEC}
							labelText="Emergency Contact Number"
							placeholder="Your emergency contact number"
							required
						/>
					</FluidForm>
					<br />
					<FluidForm class="flex flex-col lg:flex-row">
						<TextInput
							bind:value={userUN}
							labelText="Username"
							placeholder="Your system-generated username"
							readonly
						/>
						<PasswordInput
							bind:value={userPW}
							labelText="Password"
							placeholder="Enter your password"
							required
						/>
						<TextInput
							bind:value={userAC}
							labelText="Account Code"
							placeholder="Your system-generated account code"
							readonly
						/>
					</FluidForm>
				</FormGroup>
				<ButtonSet class="flex flex-col gap-2 lg:flex-row">
					<Checkbox
						bind:checked={confirm}
						on:click={generateUN}
						on:click={generateAC}
						labelText="I certify that the information above is correct."
					/>
					<Button disabled={!confirm} on:click={register} icon={Pen}>Register</Button>
				</ButtonSet>
			</Form>
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
	bind:open={mdalBG1}
	modalHeading="Report a problem"
	primaryButtonText="Submit"
	on:click:button--primary={clear}
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
		<TextInput bind:value={pageID} labelText="Problem Title" placeholder={pageID} readonly />
		<TextInput
			bind:value={probID}
			labelText="Problem ID"
			placeholder="Provide a short description of the problem you encountered"
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
	bind:open={mdalRG1}
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
			<h5>{userUN}</h5>
			<br />
			<p class="italic">Account Code:</p>
			<h3>{userAC}</h3>
			<br />
		</div>
		<div>
			<p class="italic">
				Important: <strong>Jot down what is writted in this window or take a screenshot</strong> to take
				note of your account details. This will be used upon enrollment or upon logging in to the system.
			</p>
		</div>
	</div>
</Modal>
<Modal
	bind:open={mdalRG2}
	modalHeading="Registration Failed"
	primaryButtonText="Close"
	on:click:button--primary={() => (mdalRG2 = false)}
>
	<div class="flex flex-col content-center">
		<div>
			<br />
			<h4>Your Learner's Reference Number, {userLR} already exists on the database.</h4>
			<br />
			<p class="italic">
				Double check the LRN you have input. If the problem still persists, contact the school
				directly with regards to this issue.
			</p>
		</div>
	</div>
</Modal>
<Modal
	bind:open={mdalRG3}
	size="xs"
	modalHeading="Registration Failed"
	primaryButtonText="Close"
	on:click:button--primary={() => (mdalRG3 = false)}
>
	<div class="flex flex-col content-center">
		<div>
			<br />
			<p>Please ensure that you have filled up all required fields.</p>
			<br />
		</div>
	</div>
</Modal>
