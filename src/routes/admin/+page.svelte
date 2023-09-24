<script>
	// #region imports
	// for firebase
	import { goto } from '$app/navigation';
	import { addDoc, collection, doc, getDocs, query, where } from 'firebase/firestore';
	import { db } from '../../firebase';

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
		SideNav,
		SideNavDivider,
		SideNavItems,
		SideNavLink,
		TextArea,
		TextInput
	} from 'carbon-components-svelte';

	import {
		Book,
		Box,
		Building,
		Calendar,
		CalendarSettings,
		Currency,
		Dashboard,
		Debug,
		Education,
		Events,
		Friendship,
		GenderFemale,
		GenderMale,
		IbmCloudForEducation,
		InformationFilled,
		Logout,
		Money,
		NotebookReference,
		Notification,
		Pen,
		RepoArtifact,
		ReportData,
		Return,
		Settings,
		UserSettings,
		WatsonHealthStackedScrolling_1,
		WatsonHealthStackedScrolling_2
	} from 'carbon-icons-svelte';
	// #endregion

	// #region database variables, fields
	let pageID = '/admin/users';

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
	let sideNv = false;
	let axis01 = '';
	// randoms
	let dtbsCN = false; // database connection
	let progTX = ''; // progressbar text

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
	function generateProbID() {
		const characters = 'ABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789';
		let result = '';
		for (let i = 0; i < 6; i++) {
			result += characters.charAt(Math.floor(Math.random() * characters.length));
		}
		probID = result;
	}
	// #endregion
	// #region handlers
	function handleLogout() {
		localStorage.removeItem('userID');
		localStorage.removeItem('userCL');
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

			logdID = localStorage.getItem('userID');
			logdCL = localStorage.getItem('userCL');
			logdFN = localStorage.getItem('userFN');

			const allowedUserClasses = ['admin', 'employee']; // allowed classes
			axis01 = allowedUserClasses.includes(logdCL); // check if class is allowed
		} catch (error) {
			console.error('Failed. :', error);
			dtbsCN = false;
		}
	});
</script>

{#if dtbsCN}
	{#if axis01}
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
							tooltipPosition="left"
							iconDescription="Dashboard"
							kind="secondary"
							icon={Dashboard}
						/>
						<Button
							tooltipPosition="left"
							iconDescription="Notifications"
							kind="secondary"
							icon={Notification}
						/>
						<Button
							tooltipPosition="left"
							iconDescription="User Settings"
							kind="secondary"
							icon={UserSettings}
						/>
						<Button
							href="/login"
							tooltipPosition="left"
							iconDescription="Logout"
							kind="danger"
							icon={Logout}
							on:click={handleLogout}
						/>
					</div>
				</div>
			</HeaderUtilities>
		</Header>

		<SideNav bind:isOpen={sideNv} rail>
			<SideNavItems>
				<SideNavLink icon={Debug} href="../bugs" text="Reported Bugs" />
				<SideNavDivider />
				<SideNavLink icon={InformationFilled} href="../system/about" text="System Information" />
				<!-- <SideNavLink icon={Settings} href="../system/settings" text="System Settings" />
				<SideNavLink icon={ReportData} href="../system/reports" text="System Reports" /> -->
				<SideNavDivider />
				<SideNavLink icon={ReportData} href="../reports/management" text="Management Reports" />
				<SideNavLink icon={Events} href="../admin/users" text="User Management" />
				<SideNavLink
					icon={IbmCloudForEducation}
					href="../admin/subjects"
					text="Subject Management"
				/>
				<SideNavLink icon={CalendarSettings} href="../admin/schedules" text="Schedule Management" />
				<SideNavLink
					icon={WatsonHealthStackedScrolling_1}
					href="../admin/bulletin"
					text="Bulletin Management"
				/>
				<SideNavDivider />
				<!-- <SideNavLink icon={ReportData} href="../reports/financial" text="Financial Reports" />
				<SideNavLink icon={Money} href="../financial/transactions" text="Financial Transactions" />
				<SideNavLink icon={Currency} href="../financial/settings" text="Financial Settings" />
				<SideNavDivider /> -->
				<SideNavLink icon={ReportData} href="../reports/academic" text="Academic Reports" />
				<SideNavLink icon={Education} href="../academic/students" text="Students" />
				<SideNavLink icon={Book} href="../academic/subjects" text="Subjects" />
				<SideNavLink icon={NotebookReference} href="../academic/gradebook" text="Gradebook" />
				<SideNavLink icon={Friendship} href="../guidance/records" text="Guidance Records" />
				<SideNavDivider />
				<SideNavLink icon={Calendar} href="../info/schedules" text="Class Schedules" />
				<SideNavLink
					icon={WatsonHealthStackedScrolling_2}
					href="../info/bulletin"
					text="Campus Bulletin"
				/>
				<SideNavLink icon={Building} href="../info/school" text="The School" />
				<SideNavDivider />
				<SideNavLink icon={Box} href="../archives" text="System Archives" />
			</SideNavItems>
		</SideNav>

		<!-- adapts depending on the device -->
		<div class="flex flex-col pl-10">
			<!-- displayed on mobile -->
			<Content id="#" class="flex flex-col bg-neutral-900 h-72">
				<h1 class="pt-28 lg:pt-44 text-white">Welcome, {logdFN}!</h1>
			</Content>
			<Content class="h-auto" />
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
				<h1>403 Forbidden</h1>
				<p>You are not allowed to access the page you requested.</p>
			</Content>
			<Content id="#" class="bg-neutral-900 w-1/3 h-screen hidden lg:flex" />
			<Content class="self-center w-11/12 lg:w-2/5">
				<h1>403 Forbidden</h1>
				<p>You are not allowed to access the page you requested.</p>
			</Content>
			<Content />
		</div>
	{/if}
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
			placeholder="System-generated problem ID"
			readonly
		/>
	</FluidForm>
	<FluidForm class="flex flex-col lg:flex-row">
		<TextInput
			on:focus={generateProbID}
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
			<h4>Your Learner's Reference Number {userLR} already exists on the database.</h4>
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
