<script>
	import {
		Button,
		Checkbox,
		Content,
		DataTable,
		Pagination,
		RadioButton,
		RadioButtonGroup,
		Tag,
		TextInput,
		Toolbar,
		ToolbarContent,
		ToolbarSearch
	} from 'carbon-components-svelte';

	import {
	CloseOutline,
		Home,
		Login,
		Recycle,
		UserAdmin,
		UserFollow,
		UserSponsor
	} from 'carbon-icons-svelte';

	// for pre-registration
	let currAY, currSM; // system defaults

	let userLN, userFN, userMN, userSF; // basic information
	let userLR, userRN, userDP, userUN, userPW, userST; // user information

	let loadLN, loadFN, loadMN, loadSF; // basic information
	let loadLR, loadRN, loadDP, loadUN, loadPW; // load information

	// for login
	let nputUN, nputPW;
	let invaUN, invaPW;

	import { db } from '../../firebase'; // assuming firebase.js is in the same directory
	import {
		collection,
		addDoc,
		getDocs,
		query,
		where,
		doc,
		updateDoc,
		deleteDoc
	} from 'firebase/firestore';

	// for generating reference numbers and usernames
	function generateReferenceNumber() {
		const characters = 'ABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789';
		let result = '';
		for (let i = 0; i < 6; i++) {
			result += characters.charAt(Math.floor(Math.random() * characters.length));
		}
		return result;
	}

	function generateUsername(studLast, studLRN) {
		const lastSixDigitsOfLRN = studLRN.slice(-6); // Get the last 6 characters of the LRN
		return `${studLast}.${lastSixDigitsOfLRN}@csjd.project-joan.cloud`;
	}

	// check duplicates
	async function checkIfFieldExists(fieldName, value) {
		const preRegistrationRef = collection(db, 'users');
		const q = query(preRegistrationRef, where(fieldName, '==', value));
		const snapshot = await getDocs(q);
		return !snapshot.empty; // Returns true if the entry exists
	}

	let duplLR = false;

	async function savePreReg(data) {
		try {
			const docRef = await addDoc(collection(db, 'users'), data);
			console.log('Document written with ID: ', docRef.id);
			return docRef.id; // you can return the ID to further use it if needed
		} catch (e) {
			console.error('Error adding document: ', e);
			throw e; // re-throwing the error for the caller to handle
		}
	}

	async function handlePreReg(event) {
		event.preventDefault();

		const hashedPassword = await hashPassword(userPW);
		const generatedRefNumber = generateReferenceNumber();
		const generatedUsername = generateUsername(userLN, userLR);

		duplLR = await checkIfFieldExists('userLR', userLR);

		// Construct the data object
		const formData = {
			currAY: '2023-2024',
			currSM: 'First',
			userDP: 'Junior High School',
			userLR,
			userLN,
			userFN,
			userMN,
			userUN: generatedUsername,
			userPW: hashedPassword,
			userRN: generatedRefNumber
		};

		// check duplicates
		if (duplLR) {
			return; // Don't proceed with saving the data if there are duplicates
		} else {
			try {
				await savePreReg(formData);
				alert('Data saved successfully!');
				resetPreReg();
			} catch (e) {
				alert('Failed to save data. Please try again.');
			}
		}
	}

	function resetPreReg() {
		userLN = '';
		userFN = '';
		userMN = '';
		userLR = '';
		userUN = '';
		userPW = '';
		userRN = '';
	}

	// password hashing
	import bcrypt from 'bcryptjs';
	import { onMount } from 'svelte';

	async function hashPassword(userPW) {
		const salt = await bcrypt.genSalt(10); // Generate a salt
		const hashedPassword = await bcrypt.hash(userPW, salt); // Hash the password
		return hashedPassword;
	}

	// login
	async function handleLogin() {
		try {
			// Fetch the user's data from Firestore using the entered username
			console.log(nputUN);
			const userQuery = query(collection(db, 'users'), where('userRN', '==', nputUN));

			const querySnapshot = await getDocs(userQuery);

			if (querySnapshot.empty) {
				invaUN = true;
				logAccess(nputUN, false, 'Username not found');
				return;
			}

			// Assuming one matching document for unique usernames
			const userData = querySnapshot.docs[0].data();

			// Check the password using the function from before
			const isPasswordValid = await bcrypt.compare(nputPW, userData.userPW);

			if (isPasswordValid) {
				// You can proceed to redirect the user, set a session, etc.
				logAccess(nputUN, true, 'Successful login');
			} else {
				invaPW = true;
			}
		} catch (error) {
			logAccess(nputUN, false, 'Incorrect password');
			console.error('Login error:', error);
			alert('An error occurred during login. Please try again.');
		}
	}

	// used to format date for access logs
	function formatDateToCustom(date) {
		const YYYY = date.getFullYear();
		const MM = String(date.getMonth() + 1).padStart(2, '0'); // Months are 0-indexed in JavaScript
		const DD = String(date.getDate()).padStart(2, '0');
		const HH = String(date.getHours()).padStart(2, '0');
		const MIN = String(date.getMinutes()).padStart(2, '0');
		const SS = String(date.getSeconds()).padStart(2, '0');

		return `${YYYY}${MM}${DD}-${HH}:${MIN}:${SS}`;
	}

	// access logs
	async function logAccess(userUN, success, message) {
		try {
			const formattedDate = formatDateToCustom(new Date());
			const logData = {
				userUN,
				success,
				message,
				timestamp: formattedDate
			};

			await addDoc(collection(db, 'access'), logData);
		} catch (error) {
			console.error('Failed to log access:', error);
		}
	}

	// loading the datagrid component
	async function getUserData() {
		const preRegistrationRef = collection(db, 'users');
		const snapshot = await getDocs(preRegistrationRef);
		const data = snapshot.docs.map((doc) => doc.data());
		return data;
	}

	let headers = [
		{ key: 'userLR', value: 'Student LRN' },
		{ key: 'userLN', value: 'Last Name' },
		{ key: 'userFN', value: 'First Name' },
		{ key: 'userMN', value: 'Middle Name' },
		{ key: 'userRN', value: 'Account Code' },
		{ key: 'userST', value: 'Status' },
	];

	let rows = [];
	let pageSize = 5;
	let page = 1;

	onMount(loadUserData); // load datatable values on start

	// use for refresh
	async function loadUserData() {
		const data = await getUserData();
		rows = data.map((item) => ({
			id: item.userRN,
			userRN: item.userRN,
			userLR: item.userLR,
			userLN: item.userLN,
			userFN: item.userFN,
			userMN: item.userMN
		}));
	}

	let selectedRowIds = []; // get toggled radio

	// use for loading selected data
	function getSelectedIndex() {
		console.log(selectedRowIds);
		handleReferenceCodeInput();
	}

	// load selected data on forms
	async function fetchDataByReferenceCode(referenceCode) {
		const preRegistrationRef = collection(db, 'users');
		const q = query(preRegistrationRef, where('userRN', '==', referenceCode));
		const snapshot = await getDocs(q);

		if (snapshot.empty) {
			// No records found for the provided reference code
			console.log('fetchDataByReferenceCode: Empty records.');
			return null;
		}

		return snapshot.docs[0].data();
	}

	async function handleReferenceCodeInput() {
		const data = await fetchDataByReferenceCode(selectedRowIds.toString()); // convert to string or it wont load

		if (data) {
			loadLN = data.userLN;
			loadFN = data.userFN;
			loadMN = data.userMN;
			// Assign other fields similarly...
		} else {
			// Handle the case when no records are found for the provided reference code
			// You can clear the TextInput fields or show an alert message, for example
			loadLN = '';
			loadFN = '';
			loadMN = '';
			// ... clear other fields and maybe show an alert or message
		}
	}

	// update the loaded data
	async function updateSelectedData() {
		const updatedData = {
			userLN: loadLN,
			userFN: loadFN,
			userMN: loadMN
			// ... add any other fields you want to update
		};

		const preRegistrationRef = collection(db, 'users');
		const q = query(preRegistrationRef, where('userRN', '==', selectedRowIds.toString()));
		const snapshot = await getDocs(q);

		if (snapshot.empty) {
			console.error('No records found for the selected reference code.');
			return;
		}

		const docId = snapshot.docs[0].id;
		const docRef = doc(db, 'users', docId);

		try {
			await updateDoc(docRef, updatedData);
			console.log('Successfully updated');
		} catch (error) {
			console.error('Error updating document:', error);
		}
	}

	// delete the loaded data
	async function deleteDataByReferenceCode() {
		const preRegistrationRef = collection(db, 'users');
		const q = query(preRegistrationRef, where('userRN', '==', selectedRowIds.toString()));
		const snapshot = await getDocs(q);

		if (snapshot.empty) {
			console.error('Document with the provided reference code not found.');
			return false;
		}

		// Assuming referenceCode is unique, so only one document should match
		const docId = snapshot.docs[0].id;

		try {
			const docRef = doc(db, 'users', docId);
			await deleteDoc(docRef);
			return true; // Successfully deleted
		} catch (error) {
			console.error('Error deleting document: ', error);
			return false; // Deletion failed
		}
	}

	async function handleDelete() {
		const wasSuccessful = await deleteDataByReferenceCode(userRN); // Assuming userRF holds the current reference number

		if (wasSuccessful) {
			// Handle success, e.g., show a success message or update UI
		} else {
			// Handle failure, e.g., show an error message
		}
	}
</script>

<Content>
	<div class="flex flex-col gap-12">
		<!-- header -->
		<div class="flex justify-between">
			<h1>This is a function testing page.</h1>
			<Button
				href="/"
				iconDescription="Return to homepage"
				tooltipPosition="left"
				kind="danger-tertiary"
				icon={Home}
			/>
		</div>
		<!-- tags -->
		<div>
			<Tag type="blue">CREATE modules</Tag>
			<Tag type="green">READ modules</Tag>
			<Tag type="purple">UPDATE modules</Tag>
			<Tag type="red">DELETE modules</Tag>
			<Tag type="high-contrast">EDITABLE modules</Tag>
			<Tag type="outline">VIEW ONLY modules</Tag>
		</div>
		<!-- pre reg -->
		<div id="preg" class="flex flex-col gap-2 lg:gap-4">
			<h5 class="flex justify-between">
				Pre-Registration
				<div class="flex">
					<Tag type="blue">X</Tag>
					<Tag type="high-contrast">X</Tag>
				</div>
			</h5>
			<div id="preg1" class="flex flex-col lg:flex-row gap-2">
				<div class="w-full pb-2">
					<TextInput
						bind:value={userLN}
						labelText="Last Name"
						placeholder="Enter your last name"
						required
					/>
				</div>
				<div class="w-full pb-2">
					<TextInput
						bind:value={userFN}
						labelText="First Name"
						placeholder="Enter your first name"
						required
					/>
				</div>
				<div class="w-full pb-2">
					<TextInput
						bind:value={userMN}
						labelText="Middle Name"
						placeholder="Enter your middle name"
						required
					/>
				</div>
				<div class="w-full lg:w-4/12 pb-2">
					<TextInput
						bind:value={userSF}
						labelText="Suffix"
						placeholder="Jr., III., etc."
						required
					/>
				</div>
			</div>
			<div id="preg2" class="flex flex-col lg:flex-row gap-2">
				<div class="w-full pb-2">
					<TextInput
						bind:value={currAY}
						readonly
						labelText="Academic Year"
						placeholder="2023-2024"
						required
					/>
				</div>
				<div class="w-full pb-2">
					<TextInput
						bind:value={currSM}
						readonly
						labelText="Semester"
						placeholder="Second"
						required
					/>
				</div>
				<div class="w-full pb-2">
					<TextInput
						bind:value={userLR}
						labelText="Learner's Reference Number"
						placeholder="Enter your LRN"
						required
					/>
				</div>
			</div>
			<hr />
			<div id="preg1" class="flex flex-col lg:flex-row gap-2">
				<div class="w-full pb-2">
					<TextInput
						readonly
						bind:value={userRN}
						helperText="Present this to the registrar upon enrollment."
						labelText="Reference Number"
						placeholder="Reference Number"
						required
					/>
				</div>
				<div class="w-full pb-2">
					<TextInput
						readonly
						bind:value={userUN}
						helperText="This is system generated and cannot be changed."
						labelText="Username"
						placeholder="Username"
						required
					/>
				</div>
				<div class="w-full pb-2">
					<TextInput
						bind:value={userPW}
						helperText="You can change this later on your account settings."
						labelText="Password"
						placeholder="Password"
						required
					/>
				</div>
				<!-- pc only -->
				<div class="hidden lg:grid lg:visible lg:pt-4 lg:pb-6">
					<Button
						on:click={handlePreReg}
						size="xl"
						kind="primary"
						icon={UserFollow}
						iconDescription="Pre-Register"
						tooltipPosition="left"
					/>
				</div>
				<!-- mobile only -->
				<div class="flex lg:hidden">
					<Button
						size="field"
						kind="primary"
						icon={UserFollow}
						iconDescription="Pre-Register"
						tooltipPosition="left">Pre-Register</Button
					>
				</div>
			</div>
		</div>
		<!-- login test -->
		<div id="preg" class="flex flex-col gap-2 lg:gap-4">
			<h5 class="flex justify-between">
				Account Login Test
				<div class="flex">
					<Tag type="blue">X</Tag>
					<Tag type="high-contrast">X</Tag>
				</div>
			</h5>
			<div id="preg1" class="grid grid-cols-2 gap-2">
				<div class="w-full pb-2">
					<TextInput
						bind:value={nputUN}
						invalid={invaUN}
						invalidText="Invalid username."
						labelText="Username"
						placeholder="Username"
						required
					/>
				</div>
				<div class="w-full pb-2">
					<TextInput
						bind:value={nputPW}
						invalid={invaPW}
						invalidText="Incorrect password."
						labelText="Password"
						placeholder="Password"
						required
					/>
				</div>
				<div class="flex">
					<Button
						on:click={handleLogin}
						size="field"
						kind="primary"
						icon={Login}
						iconDescription="Login"
						tooltipPosition="left">Login</Button
					>
				</div>
			</div>
		</div>
		<!-- masterlist -->
		<div id="load" class="flex flex-col gap-4">
			<h5 class="flex justify-between">
				Student Masterlist
				<div class="flex">
					<Tag type="green">X</Tag>
					<Tag type="purple">X</Tag>
					<Tag type="high-contrast">X</Tag>
				</div>
			</h5>
			<div id="load0" class="flex lg:hidden">
				<p>
					If you are seeing this message, your screen is too small.<br /><br />Change to a device
					with a larger screen or rotate your device to view this section.
				</p>
			</div>
			<div id="load1" class="hidden lg:flex gap-2">
				<div class="w-screen">
					<DataTable
						radio
						zebra
						sortable
						size="short"
						{headers}
						{rows}
						{page}
						{pageSize}
						bind:selectedRowIds
						on:click:row--select={handleReferenceCodeInput}
						on:mouseenter:row={loadUserData}
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
							</ToolbarContent>
						</Toolbar>
					</DataTable>
					<Pagination bind:pageSize bind:page totalItems={rows.length} pageSizeInputDisabled />
					<!-- <Pagination {rows} /> -->
				</div>
			</div>
		</div>
		<!-- student info -->
		<div id="preg" class="flex flex-col gap-4">
			<h5 class="flex justify-between">
				Student Information
				<div class="flex flex-row">
					<Tag type="green">X</Tag>
					<Tag type="purple">X</Tag>
					<Tag type="red">X</Tag>
					<Tag type="high-contrast">X</Tag>
				</div>
			</h5>
			<div id="load2" class="flex flex-col lg:flex-row gap-2">
				<div class="w-full pb-2">
					<TextInput
						bind:value={loadLN}
						labelText="Last Name"
						placeholder="Your last name"
						required
					/>
				</div>
				<div class="w-full pb-2">
					<TextInput
						bind:value={loadFN}
						labelText="First Name"
						placeholder="Your first name"
						required
					/>
				</div>
				<div class="w-full pb-2">
					<TextInput
						bind:value={loadMN}
						labelText="Middle Name"
						placeholder="Your middle name"
						required
					/>
				</div>
			</div>
			<div id="load3" class="flex flex-col lg:flex-row gap-2">
				<div class="w-full pb-2">
					<TextInput
						bind:value={currAY}
						readonly
						labelText="Academic Year"
						placeholder="Academic Year"
						required
					/>
				</div>
				<div class="w-full pb-2">
					<TextInput
						bind:value={loadLR}
						readonly
						labelText="Learner's Reference Number"
						placeholder="Learner's Reference Number"
						required
					/>
				</div>
				<div class="w-full pb-2">
					<TextInput
						bind:value={currSM}
						readonly
						labelText="Semester"
						placeholder="Semester"
						required
					/>
				</div>
				<div class="w-full pb-2">
					<TextInput
						bind:value={loadDP}
						readonly
						labelText="Department"
						placeholder="Department"
						required
					/>
				</div>
			</div>
			<div id="load4" class="flex flex-col lg:flex-row gap-2">
				<div class="w-full pb-2">
					<TextInput readonly labelText="Grade Level" placeholder="Grade Level" required />
				</div>
				<div class="w-full pb-2">
					<TextInput readonly labelText="Section" placeholder="Section" required />
				</div>
				<div class="w-full pb-2">
					<TextInput readonly labelText="Class Adviser" placeholder="Class Adviser" required />
				</div>
			</div>
			<div id="load5" class="flex flex-col lg:justify-between">
				<div class="flex flex-col lg:flex-row lg:justify-between gap-4">
					<div class="flex flex-col">
						<Checkbox labelText="PSA / Birth Certificate" checked />
						<Checkbox labelText="Certificate of Good Moral" checked />
					</div>
					<div class="flex flex-col">
						<Checkbox labelText="Form 137" checked />
						<Checkbox labelText="Form 138" checked />
					</div>
					<div class="flex flex-col">
						<RadioButtonGroup selected="" orientation="vertical">
							<RadioButton labelText="paid downpayment (installment)" value="installment" />
							<RadioButton labelText="paid tuition (full payment)" value="full" />
						</RadioButtonGroup>
						<!-- <Checkbox labelText="paid downpayment (installment)" checked />
						<Checkbox labelText="paid tuition (full payment)" checked /> -->
					</div>
					<div class="flex">
						<Button
							size="field"
							kind="secondary"
							icon={UserFollow}
							iconDescription="Enroll"
							tooltipPosition="left"
							on:click={updateSelectedData}
							on:click={loadUserData}
						/>
						<Button
							size="field"
							kind="primary"
							icon={UserAdmin}
							iconDescription="Activate"
							tooltipPosition="left"
						/>
						<Button
							size="field"
							kind="danger"
							icon={UserSponsor}
							iconDescription="Deactivate"
							tooltipPosition="left"
							on:click={handleDelete}
							on:click={loadUserData}
						/>
					</div>
				</div>
			</div>
		</div>
	</div>
</Content>
