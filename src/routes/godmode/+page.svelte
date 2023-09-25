<script>
	// #region imports
	// for firebase
	import { goto } from '$app/navigation';
	import {
		addDoc,
		collection,
		deleteDoc,
		doc,
		getDocs,
		query,
		updateDoc,
		where
	} from 'firebase/firestore';
	import { db } from '../../firebase';

	import { onMount } from 'svelte';

	// for passwords
	import bcrypt from 'bcryptjs';

	import {
		Alarm,
		GlobalCurrency,
		Group,
		OptimizeCashFlow_02,
		Warning_02
	} from 'carbon-pictograms-svelte';

	import {
		Button,
		ButtonSet,
		Checkbox,
		ComboBox,
		Content,
		DataTable,
		FluidForm,
		Form,
		FormGroup,
		Header,
		HeaderGlobalAction,
		HeaderUtilities,
		InlineNotification,
		Link,
		Modal,
		Pagination,
		PasswordInput,
		ProgressBar,
		RadioButton,
		RadioButtonGroup,
		SideNav,
		SideNavDivider,
		SideNavItems,
		SideNavLink,
		Tag,
		TextArea,
		TextInput,
		Toolbar,
		ToolbarContent,
		ToolbarMenu,
		ToolbarMenuItem,
		ToolbarSearch,
		Tooltip
	} from 'carbon-components-svelte';

	import {
		Add,
		Book,
		Box,
		Building,
		Calendar,
		CalendarSettings,
		Currency,
		Dashboard,
		Debug,
		Delete,
		Edit,
		Education,
		Events,
		Filter,
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
		Recycle,
		RepoArtifact,
		ReportData,
		Return,
		Save,
		Settings,
		SettingsAdjust,
		TrashCan,
		UserAdmin,
		UserFollow,
		UserSettings,
		UserSponsor,
		WatsonHealthStackedScrolling_1,
		WatsonHealthStackedScrolling_2
	} from 'carbon-icons-svelte';
	// #endregion

	// #region database variables, fields
	let pageID = '/';

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

	// users (from db)
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
	let godmode;

	let axis01,
		axis02,
		axis03,
		axis04,
		axis05,
		axis06,
		axis07,
		axis08,
		axis09,
		axis10,
		axis11,
		axis12,
		axis13,
		axis14,
		axis15,
		axis16,
		axis17,
		axis18,
		axis19,
		axis20,
		axis21,
		axis22,
		axis23,
		axis24,
		axis25;

	function clearAxis() {
		axis01 = false;
		axis02 = false;
		axis03 = false;
		axis04 = false;
		axis05 = false;
		axis06 = false;
		axis07 = false;
		axis08 = false;
		axis09 = false;
		axis10 = false;
		axis11 = false;
		axis12 = false;
		axis13 = false;
		axis14 = false;
		axis15 = false;
		axis16 = false;
		axis17 = false;
		axis18 = false;
		axis19 = false;
		axis20 = false;
		axis21 = false;
		axis22 = false;
		axis23 = false;
		axis24 = false;
		axis25 = false;
	}

	let open01,
		open02,
		open03,
		open04,
		open05,
		open06,
		open07,
		open08,
		open09,
		open10,
		open11,
		open12,
		open13,
		open14,
		open15,
		open16,
		open17,
		open18,
		open19,
		open20,
		open21,
		open22,
		open23,
		open24,
		open25;

	function clearOpen() {
		open01 = false;
		open02 = false;
		open03 = false;
		open04 = false;
		open05 = false;
		open06 = false;
		open07 = false;
		open08 = false;
		open09 = false;
		open10 = false;
		open11 = false;
		open12 = false;
		open13 = false;
		open14 = false;
		open15 = false;
		open16 = false;
		open17 = false;
		open18 = false;
		open19 = false;
		open20 = false;
		open21 = false;
		open22 = false;
		open23 = false;
		open24 = false;
		open25 = false;
	}

	// randoms
	let dtbsCN = false; // database connection
	let progTX = ''; // progressbar text

	let male = false;
	let female = false;

	let headTX = '',
		descTX = '';

	let confirm = false,
		duplLR = false; // for confirmation checkbox

	// modals
	let mdalRG1 = false;
	let mdalRG2 = false;
	let mdalRG3 = false;
	let mdalBG1 = false;

	// database values
	const getSchoolID = doc(db, 'schools', '0303001');
	const getUsers = collection(getSchoolID, 'users');
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
	// #region loaders

	// #endregion
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

			headTX = 'Pick a module.';
			descTX = 'On the sidebar, choose the module that you want to work on.';

			const god = ['god']; // allowed classes
			godmode = god.includes(logdCL); // check if class is allowed
		} catch (error) {
			console.error('Failed. :', error);
			dtbsCN = false;
		}
	});

	// #region user management
	let editM06 = true;

	let headers06 = [
		{ key: 'userID', value: 'Account ID' },
		{ key: 'userLR', value: 'Student LRN' },
		{ key: 'userLN', value: 'Last Name' },
		{ key: 'userFN', value: 'First Name' },
		{ key: 'userMN', value: 'Middle Name' },
		{ key: 'userON', value: 'Status' }
	];

	let rows06 = [];
	let pageSize06 = 5;
	let page06 = 1;

	let selectedRowIds06 = []; // get toggled radio

	function openM06() {
		if (!open06) {
			clearOpen();
			clearAxis();
			open06 = true;
			headTX = 'Pick a module.';
			descTX = 'On the sidebar, choose the module that you want to work on.';
			const allowedOnM06 = ['god', 'admin']; // allowed classes
			axis06 = allowedOnM06.includes(logdCL); // check if class is allowed
			loadUserData();
		} else {
			clearOpen();
			clearAxis();
			headTX = 'Pick a module.';
			descTX = 'On the sidebar, choose the module that you want to work on.';
		}
	}

	function enableEditM06() {
		if (editM06) {
			editM06 = false;

			if (userGD == 'Male') {
				male = true;
				female = false;
			} else {
				female = true;
				male = false;
			}
		} else if (!editM06) {
			editM06 = true;

			male = false;
			female = false;
		}
	}

	async function fetchDataByReferenceCode(referenceCode) {
		const getSchoolID = doc(db, 'schools', '0303001');
		const getUsers = collection(getSchoolID, 'users');

		const q = query(getUsers, where('userID', '==', referenceCode));
		const snapshot = await getDocs(q);

		if (snapshot.empty) {
			// No records found for the provided reference code
			console.log('fetchDataByReferenceCode: Empty records.');
			return null;
		}

		return snapshot.docs[0].data();
	}

	async function handleReferenceCodeInput() {
		const data = await fetchDataByReferenceCode(selectedRowIds06.toString()); // convert to string or it wont load

		if (data) {
			userID = data.userID;
			userCL = data.userCL;
			userAY = data.userAY;
			userSM = data.userSM;
			userLR = data.userLR;
			userLN = data.userLN;
			userFN = data.userFN;
			userMN = data.userMN;
			userSF = data.userSF;
			userGD = data.userGD;
			userCN = data.userCN;
			userAD = data.userAD;
			userMA = data.userMA;
			userFA = data.userFA;
			userGA = data.userGA;
			userCP = data.userCP;
			userCR = data.userCR;
			userEC = data.userEC;
			userON = data.userON;
		} else {
			// clear fields
			userLN = '';
			userFN = '';
			userMN = '';
		}
	}

	async function getUserData() {
		const q = query(getUsers, where('userCL', '!=', 'god'));
		const snapshot = await getDocs(q);
		const data = snapshot.docs.map((doc) => doc.data());
		return data;
	}

	async function loadUserData() {
		const data = await getUserData();
		rows06 = data.map((item) => ({
			id: item.userID,
			userID: item.userID,
			userCL: item.userCL,
			userAY: item.userAY,
			userSM: item.userSM,
			userLR: item.userLR,
			userLN: item.userLN,
			userFN: item.userFN,
			userMN: item.userMN,
			userSF: item.userSF,
			userGD: item.userGD,
			userCN: item.userCN,
			userAD: item.userAD,
			userMA: item.userMA,
			userFA: item.userFA,
			userGA: item.userGA,
			userCP: item.userCP,
			userCR: item.userCR,
			userEC: item.userEC,
			userON: item.userON
		}));
	}

	async function getStudentData() {
		const q = query(getUsers, where('userCL', '==', 'student'));
		const snapshot = await getDocs(q);
		const data = snapshot.docs.map((doc) => doc.data());
		return data;
	}

	async function loadStudentData() {
		const data = await getStudentData();
		rows06 = data.map((item) => ({
			id: item.userID,
			userID: item.userID,
			userCL: item.userCL,
			userAY: item.userAY,
			userSM: item.userSM,
			userLR: item.userLR,
			userLN: item.userLN,
			userFN: item.userFN,
			userMN: item.userMN,
			userSF: item.userSF,
			userGD: item.userGD,
			userCN: item.userCN,
			userAD: item.userAD,
			userMA: item.userMA,
			userFA: item.userFA,
			userGA: item.userGA,
			userCP: item.userCP,
			userCR: item.userCR,
			userEC: item.userEC,
			userON: item.userON
		}));
	}

	async function getEmployeeData() {
		const q = query(getUsers, where('userCL', '!=', 'student'));
		const snapshot = await getDocs(q);
		const data = snapshot.docs.map((doc) => doc.data());
		return data;
	}

	async function loadEmployeeData() {
		const data = await getEmployeeData();
		rows06 = data.map((item) => ({
			id: item.userID,
			userID: item.userID,
			userCL: item.userCL,
			userAY: item.userAY,
			userSM: item.userSM,
			userLR: item.userLR,
			userLN: item.userLN,
			userFN: item.userFN,
			userMN: item.userMN,
			userSF: item.userSF,
			userGD: item.userGD,
			userCN: item.userCN,
			userAD: item.userAD,
			userMA: item.userMA,
			userFA: item.userFA,
			userGA: item.userGA,
			userCP: item.userCP,
			userCR: item.userCR,
			userEC: item.userEC,
			userON: item.userON
		}));
	}

	async function updateLoadM06() {
		const updatedData = {
			userAY: userAY,
			userCL: userCL,
			userSM: userSM,
			userLR: userLR,
			userLN: userLN,
			userFN: userFN,
			userMN: userMN,
			userSF: userSF,
			userGD: userGD,
			userCN: userCN,
			userAD: userAD,
			userMA: userMA,
			userFA: userFA,
			userGA: userGA,
			userCP: userCP,
			userCR: userCR,
			userEC: userEC,
			userON: userON
		};

		const getSchoolID = doc(db, 'schools', '0303001');
		const getUsers = collection(getSchoolID, 'users');

		const q = query(getUsers, where('userID', '==', selectedRowIds06.toString()));
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

	async function deleteDataByReferenceCode(referenceCode) {
		const getSchoolID = doc(db, 'schools', '0303001');
		const getUsers = collection(getSchoolID, 'users');

		const q = query(getUsers, where('userID', '==', referenceCode));
		const snapshot = await getDocs(q);

		if (snapshot.empty) {
			// No records found for the provided reference code
			console.log('fetchDataByReferenceCode: Empty records.');
			return null;
		}

		const docId = snapshot.docs[0].id;

		try {
			const docRef = doc(db, 'schools', '0303001', 'users', docId);
			await deleteDoc(docRef);
			return true; // Successfully deleted
		} catch (error) {
			console.error('Error deleting document: ', error);
			return false; // Deletion failed
		}
	}

	async function deleteLoadM06() {
		const wasSuccessful = await deleteDataByReferenceCode(userID); // Assuming userRF holds the current reference number

		if (wasSuccessful) {
			// Handle success, e.g., show a success message or update UI
		} else {
			// Handle failure, e.g., show an error message
		}
	}
	// #endregion
	// #region financial settings
	let headers17 = [
		{ key: 'tranID', value: 'Transaction ID' },
		{ key: 'tranNM', value: 'Transaction Name' }
	];

	let rows17 = [];
	let pageSize17 = 5;
	let page17 = 1;

	let selectedRowIds17 = [];

	function openM17() {
		if (!open17) {
			clearOpen();
			clearAxis();
			open17 = true;
			headTX = 'Pick a module.';
			descTX = 'On the sidebar, choose the module that you want to work on.';
			const allowedOnM17 = ['god']; // allowed classes
			axis17 = allowedOnM17.includes(logdCL); // check if class is allowed
			loadUserData();
		} else {
			clearOpen();
			clearAxis();
			headTX = 'Pick a module.';
			descTX = 'On the sidebar, choose the module that you want to work on.';
		}
	}

	function generateTranID() {
		const characters = 'ABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789';
		let result = '';
		for (let i = 0; i < 6; i++) {
			result += characters.charAt(Math.floor(Math.random() * characters.length));
		}
		tranID = result;
	}

	async function checkDuplicateTID(field, value) {
		const getSchoolID = doc(db, 'schools', '0303001');
		const getTransactions = collection(getSchoolID, 'financial');
		const q = query(getTransactions, where(field, '==', value));
		const snapshot = await getDocs(q);
		return !snapshot.empty; // Returns true if the entry exists
	}

	async function createM17(event) {
		generateTranID();
		let duplID = await checkDuplicateTID('tranID', tranID);

		if (tranID == '' || tranNM == '' || tranAM == '' || tranPY == '') {
		} else {
		}

		// Construct the data object
		const formData = {
			tranAY,
			tranID,
			tranNM,
			tranDC,
			tranAM,
			tranPY
		};

		// check duplicates
		if (duplID) {
			return; // Don't proceed with saving the data if there are duplicates
		} else {
			try {
				await uploadM17(formData);
			} catch (e) {
				console.log('Failed to save data. Please try again.');
			}
		}
	}

	async function uploadM17(data) {
		try {
			const docRef = await addDoc(collection(db, 'schools', '0303001', 'financial'), data);
			console.log('Document written with ID: ', docRef.id);
			return docRef.id; // you can return the ID to further use it if needed
		} catch (e) {
			console.error('Error adding document: ', e);
			throw e; // re-throwing the error for the caller to handle
		}
	}

	async function getFinancialData() {
		const getSchoolID = doc(db, 'schools', '0303001');
		const getTransactions = collection(getSchoolID, 'financial');
		const q = query(getTransactions);
		const snapshot = await getDocs(q);
		const data = snapshot.docs.map((doc) => doc.data());
		return data;
	}

	async function loadFinancialData() {
		const data = await getFinancialData();
		rows17 = data.map((item) => ({
			id: item.tranID,
			tranAY: item.tranAY,
			tranID: item.tranID,
			tranNM: item.tranNM,
			tranDC: item.tranDC,
			tranAM: item.tranAM,
			tranPY: item.tranPY
		}));
	}

	// #endregion
</script>

{#if dtbsCN}
	{#if godmode}
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
				<SideNavLink icon={Debug} text="Reported Bugs" />
				<SideNavDivider />
				<SideNavLink icon={Dashboard} href="../system" text="System Dashboard" />
				<SideNavLink icon={InformationFilled} href="../system/about" text="System Information" />
				<SideNavLink icon={Settings} href="../system/settings" text="System Settings" />
				<SideNavDivider />
				<SideNavLink icon={Dashboard} href="../admin" text="Management Dashboard" />
				<SideNavLink
					bind:isSelected={open06}
					on:click={openM06}
					icon={Events}
					text="User Management"
				/>
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
				<SideNavLink icon={Dashboard} href="../academic" text="Academic Dashboard" />
				<SideNavLink icon={Education} href="../academic/students" text="Students" />
				<SideNavLink icon={Book} href="../academic/subjects" text="Subjects" />
				<SideNavLink icon={NotebookReference} href="../academic/gradebook" text="Gradebook" />
				<SideNavLink icon={Friendship} href="../guidance/records" text="Guidance Records" />
				<SideNavDivider />
				<SideNavLink icon={Dashboard} href="../financial" text="Financial Dashboard" />
				<SideNavLink icon={Money} href="../financial/transactions" text="Financial Transactions" />
				<SideNavLink
					bind:isSelected={open17}
					on:click={openM17}
					icon={Currency}
					text="Financial Settings"
				/>
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

		<!-- user management -->
		{#if axis06}
			<!-- adapts depending on the device -->
			<div class="flex flex-col pl-10 pt-10 block relative">
				<Content>
					<div class="flex flex-col gap-4">
						<div class="flex lg:hidden" />
						<div class="flex gap-4 items-center">
							<Group />
							<h1>User Management</h1>
							<Tooltip direction="right">
								<p>This module manages all basic user information.</p>
							</Tooltip>
						</div>
						<div class="hidden lg:flex gap-2">
							<div class="w-screen">
								<DataTable
									radio
									zebra
									sortable
									size="short"
									headers={headers06}
									rows={rows06}
									{page06}
									{pageSize06}
									bind:selectedRowIds={selectedRowIds06}
									on:click:row--select={handleReferenceCodeInput}
								>
									<Toolbar>
										<ToolbarContent>
											<ToolbarSearch shouldFilterRows />
											<Button
												kind="ghost"
												on:click={loadUserData}
												icon={Recycle}
												iconDescription="Reload"
												tooltipPosition="left"
											/>
											<ToolbarMenu icon={Filter} iconDescription="Filter">
												<ToolbarMenuItem on:click={loadStudentData}>Students Only</ToolbarMenuItem>
												<ToolbarMenuItem on:click={loadEmployeeData}>Employees Only</ToolbarMenuItem
												>
											</ToolbarMenu>
											<Button
												kind="tertiary"
												icon={Edit}
												iconDescription="Edit Selected"
												tooltipPosition="left"
												on:click={enableEditM06}
											/>
											<Button
												kind="primary"
												icon={Save}
												iconDescription="Save Changes"
												tooltipPosition="left"
												bind:disabled={editM06}
												on:click={enableEditM06}
												on:click={updateLoadM06}
											/>
											<Button
												kind="danger"
												icon={TrashCan}
												iconDescription="Delete Selected"
												tooltipPosition="left"
												bind:disabled={editM06}
												on:click={deleteLoadM06}
											/>
										</ToolbarContent>
									</Toolbar>
								</DataTable>
								<Pagination
									bind:pageSize06
									bind:page06
									totalItems={rows06.length}
									pageSizeInputDisabled
								/>
								<!-- <Pagination {rows} /> -->
							</div>
						</div>
					</div>
					<div class="flex flex-col">
						<br />
						<FluidForm class="flex flex-col lg:flex-row">
							<TextInput
								bind:value={userID}
								labelText="Account Code"
								placeholder="Selected account code"
								readonly
							/>
							<TextInput
								bind:value={userCL}
								labelText="User Class"
								placeholder="registrar, cashier, guidance, faculty, student"
								bind:readonly={editM06}
							/>
							<TextInput
								bind:value={userON}
								labelText="User Status"
								placeholder="Change to ACTIVE or INACTIVE"
								bind:readonly={editM06}
							/>
						</FluidForm>
						<br />
						<FluidForm class="flex flex-col lg:flex-row">
							<TextInput
								bind:value={acadYR}
								readonly
								labelText="Academic Year"
								placeholder="2023-2024"
							/>
							<TextInput bind:value={acadSM} readonly labelText="Semester" placeholder="Second" />
							<TextInput
								bind:value={userLR}
								labelText="LRN / Government ID Number"
								placeholder="Selected LRN / Government ID Number"
								bind:readonly={editM06}
							/>
						</FluidForm>
						<FluidForm class="flex flex-col lg:flex-row">
							<TextInput
								bind:value={userLN}
								labelText="Last Name"
								placeholder="Selected last name"
								bind:readonly={editM06}
							/>
							<TextInput
								bind:value={userFN}
								labelText="First Name"
								placeholder="Selected first name"
								bind:readonly={editM06}
							/>
							<TextInput
								bind:value={userMN}
								labelText="Middle Name"
								placeholder="Selected middle name"
								bind:readonly={editM06}
							/>
							<TextInput
								bind:value={userSF}
								labelText="Suffix (if any)"
								placeholder="Sr., Jr., III., etc."
								bind:readonly={editM06}
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
								placeholder="Selected contact number"
								bind:readonly={editM06}
							/>
							<TextInput
								bind:value={userAD}
								labelText="Student Address"
								placeholder="Selected current address"
								bind:readonly={editM06}
							/>
						</FluidForm>
						<FluidForm class="flex flex-col lg:flex-row">
							<TextInput
								bind:value={userMA}
								labelText="Mother's Name"
								placeholder="Selected mother's name"
								bind:readonly={editM06}
							/>
							<TextInput
								bind:value={userFA}
								labelText="Father's Name"
								placeholder="Selected father's name"
								bind:readonly={editM06}
							/>
							<TextInput
								bind:value={userGA}
								labelText="Guardian's Name (optional)"
								placeholder="Selected guardian's name"
								bind:readonly={editM06}
							/>
						</FluidForm>
						<FluidForm class="flex flex-col lg:flex-row">
							<TextInput
								bind:value={userCP}
								labelText="Contact Person"
								placeholder="Selected contact person's name"
								bind:readonly={editM06}
							/>
							<TextInput
								bind:value={userCR}
								labelText="Relation to Contact Person (optional)"
								placeholder="Selected relation to your contact person"
								bind:readonly={editM06}
							/>
							<TextInput
								bind:value={userEC}
								labelText="Emergency Contact Number"
								placeholder="Selected emergency contact number"
								bind:readonly={editM06}
							/>
						</FluidForm>
					</div>
				</Content>
			</div>
			<!-- financial settings  -->
		{:else if axis17}
			<div class="flex flex-col pl-10 pt-10">
				<Content>
					<div class="flex flex-col gap-4">
						<div class="flex lg:hidden">
							<p>
								If you are seeing this message, your screen is too small.<br /><br />Change to a
								device with a larger screen or rotate your device to view this section.
							</p>
						</div>
						<div class="flex gap-4 items-center">
							<OptimizeCashFlow_02 />
							<h1>Financial Settings</h1>
							<Tooltip direction="right">
								<p>This module allows you to setup transaction defaults.</p>
							</Tooltip>
						</div>
						<div>
							<div class="flex flex-col lg:flex-row gap-4">
								<div class="flex flex-col lg:w-1/2">
									<DataTable
										radio
										zebra
										sortable
										size="short"
										headers={headers17}
										rows={rows17}
										{page17}
										{pageSize17}
										bind:selectedRowIds={selectedRowIds17}
										on:click:row--select={handleReferenceCodeInput}
									>
										<Toolbar>
											<ToolbarContent>
												<ToolbarSearch shouldFilterRows />
												<Button
													kind="ghost"
													on:click={loadFinancialData}
													icon={Recycle}
													iconDescription="Reload"
													tooltipPosition="left"
												/>
												<Button
													kind="ghost"
													icon={Add}
													iconDescription="Create New"
													tooltipPosition="left"
													on:click={createM17}
												/>
												<Button
													kind="tertiary"
													icon={Edit}
													iconDescription="Edit Selected"
													tooltipPosition="left"
													on:click={enableEditM06}
												/>
												<Button
													kind="primary"
													icon={Save}
													iconDescription="Save Changes"
													tooltipPosition="left"
													bind:disabled={editM06}
													on:click={enableEditM06}
													on:click={updateLoadM06}
												/>
												<Button
													kind="danger"
													icon={TrashCan}
													iconDescription="Delete Selected"
													tooltipPosition="left"
													bind:disabled={editM06}
													on:click={deleteLoadM06}
												/>
											</ToolbarContent>
										</Toolbar>
									</DataTable>
									<Pagination
										bind:pageSize17
										bind:page17
										totalItems={rows17.length}
										pageSizeInputDisabled
									/>
								</div>
								<div class="flex flex-col lg:w-1/2 gap-4">
									<ComboBox
										bind:value={tranAY}
										titleText="Academic Year"
										placeholder="Academic Year"
										items={[{ id: '0', text: '2023-2024' }]}
									/>
									<TextInput
										bind:value={tranID}
										readonly
										labelText="Transaction ID"
										placeholder="This is system-generated"
										required
									/>
									<TextInput
										bind:value={tranNM}
										labelText="Transaction Name"
										placeholder="Title of transaction"
										required
									/>
									<TextInput
										bind:value={tranDC}
										labelText="Transaction Description"
										placeholder="Provide a short description for this transaction"
										required
									/>
									<TextInput
										bind:value={tranAM}
										labelText="Amount to Pay"
										placeholder="Php0.00"
										required
									/>
									<ComboBox
										bind:value={tranPY}
										titleText="To be paid by"
										placeholder="Year Level"
										items={[
											{ id: '0', text: 'Grade 01' },
											{ id: '1', text: 'Grade 02' },
											{ id: '2', text: 'Grade 03' },
											{ id: '3', text: 'Grade 04' },
											{ id: '4', text: 'Grade 05' },
											{ id: '5', text: 'Grade 06' },
											{ id: '6', text: 'Grade 07' },
											{ id: '7', text: 'Grade 08' },
											{ id: '8', text: 'Grade 09' },
											{ id: '9', text: 'Grade 10' },
											{ id: '10', text: 'Grade 11' },
											{ id: '11', text: 'Grade 12' }
										]}
									/>
								</div>
							</div>
						</div>
					</div>
				</Content>
			</div>
		{:else}
			<div class="flex flex-col items-stretch lg:flex-row">
				<!-- displayed on mobile -->
				<Content class="flex flex-col h-60 bg-neutral-900 lg:hidden">
					<h1>{headTX}</h1>
					<p class="italic">{descTX}</p>
				</Content>
				<Content class="w-1/3 h-screen hidden lg:flex" />
				<Content class="self-center w-11/12 lg:w-2/5">
					<h1>{headTX}</h1>
					<p class="italic">{descTX}</p>
				</Content>
				<Content />
			</div>
		{/if}
		<!-- #endregion -->
	{:else}
		<div class="flex flex-col items-stretch lg:flex-row">
			<!-- displayed on mobile -->
			<Content class="w-1/3 h-screen hidden lg:flex" />
			<Content class="self-center w-11/12 lg:w-2/5">
				<Warning_02 />
				<br />
				<h1>Thou shalt not enter.</h1>
				<br />
				<p class="italic">
					The <strong>god mode</strong> is designed only for testing purposes. It cannot be accessed
					by anyone other than the developer or the system administrator.
				</p>
				<br />
				<p class="italic">
					You have been warned, <strong>{logdFN}</strong>, <br />Cease and desist.
				</p>
				<br />
				<Link class="italic" href="/home">Leave at once.</Link>
			</Content>
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
