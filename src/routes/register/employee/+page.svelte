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
		RadioButton,
		RadioButtonGroup,
		RadioTile,
		TextArea,
		TextInput,
		TileGroup,
		Tooltip
	} from 'carbon-components-svelte';

	import { Debug, GenderFemale, GenderMale, Pen, Return } from 'carbon-icons-svelte';
	// #endregion

	// #region database variables, fields
	let pageID = '/register/employee';

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
		goto('/login');
	}

	function clear() {
		userID = '';
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
		userCP = '';
		userCR = '';
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
		userID = result;
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
			userCP == '' ||
			userEC == '' ||
			userUN == '' ||
			userPW == '' ||
			userID == ''
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
				userID,
				userCL: 'employee',
				userUN,
				userPW: hashedPassword,
				userLR,
				userLN,
				userFN,
				userMN,
				userSF,
				userGD,
				userCN,
				userAD,
				userFA,
				userMA,
				userGA,
				userCP,
				userCR,
				userEC,
				userAY: '2023-2024',
				userSM: 'Second',
				userYR,
				userSC,
				userSD,
				userAV,
				userON: 'ACTIVE',
				userHS,
				userRP,
				userUC,
				userR1,
				userR2,
				userR3,
				userR4,
				userR5,
				userR6,
				userR7,
				userR8,
				userR9
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
						tooltipPosition="left"
						iconDescription="Report a problem"
						kind="ghost"
						icon={Debug}
						on:click={() => (mdalBG1 = true)}
					/>
					<Button
						href="/login"
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
	<div class="flex flex-col">
		<!-- displayed on mobile -->
		<Content id="#" class="flex flex-col bg-neutral-900 h-72">
			<h1 class="pt-28 lg:pt-44 text-white">Employee Registration</h1>
		</Content>
		<Content class="h-auto">
			<Form on:submit>
				<FormGroup
					legendText="This page is only meant to be used by employees of Colegio de San Juan de Dios, Incorporated."
				>
					<br />
					<RadioButtonGroup selected="">
						<div slot="legendText">Pick your provided Government ID</div>
						<div class="flex flex-col lg:flex-row lg:gap-4 place-items-start">
							<RadioButton value="prc" labelText="Professional License (PRC ID)" />
							<RadioButton value="lto" labelText="Driver's License (LTO)" />
							<RadioButton value="nat" labelText="National ID" />
							<RadioButton value="phl" labelText="PhilHealth ID" />
							<RadioButton value="ppt" labelText="Passport" />
						</div>
					</RadioButtonGroup>
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
							labelText="Government ID Number"
							placeholder="Enter your ID number"
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
							labelText="Employee Address"
							placeholder="Your current address"
							required
						/>
					</FluidForm>
					<FluidForm class="flex flex-col lg:flex-row">
						<TextInput
							bind:value={userCP}
							labelText="Contact Person"
							placeholder="Your contact person's name"
							required
						/>
						<TextInput
							bind:value={userCR}
							labelText="Relation to Contact Person (optional)"
							placeholder="Your relation to your contact person"
							required
						/>
						<TextInput
							bind:value={userEC}
							labelText="Emergency Contact Number"
							placeholder="Your emergency contact number"
						/>
					</FluidForm>
					<br />
					<FluidForm class="flex flex-col lg:flex-row">
						<TextInput
							bind:value={userID}
							labelText="Account ID"
							placeholder="Your system-generated account ID"
							readonly
						/>
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
							tooltipPosition="left"
							required
						/>
					</FluidForm>
					<br />
					<ButtonSet class="flex flex-col gap-2 lg:flex-row">
						<Checkbox
							bind:checked={confirm}
							on:click={generateUN}
							on:click={generateAC}
							labelText="I certify that the information above is correct."
						/>
						<br />
						<Button disabled={!confirm} on:click={register} icon={Pen}>Register</Button>
					</ButtonSet>
				</FormGroup>
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
			<h3>{userID}</h3>
			<br />
		</div>
		<div>
			<p class="italic">
				Important: <strong>Jot down what is writted in this window or take a screenshot</strong> to take
				note of your account details. This will be used upon logging in to the system.
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
			<h4>Your Government ID Number {userLR} already exists on the database.</h4>
			<br />
			<p class="italic">
				Double check the ID number you input. If the problem still persists, contact the system
				administrator directly with regards to this issue.
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
