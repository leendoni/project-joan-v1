<script>
	import { goto } from '$app/navigation';
	// components
	import {
		Button,
		Content,
		Header,
		HeaderUtilities,
		SideNav,
		SideNavDivider,
		SideNavItems,
		SideNavLink
	} from 'carbon-components-svelte';
	// icons
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
		IbmCloudForEducation,
		InformationFilled,
		Logout,
		Money,
		NotebookReference,
		Notification,
		RepoArtifact,
		ReportData,
		Return,
		Settings,
		UserSettings,
		WatsonHealthStackedScrolling_1,
		WatsonHealthStackedScrolling_2
	} from 'carbon-icons-svelte';

	// pictograms

	// variables
	let isSideNavOpen = false;

	function handleBack() {
        window.history.back();
	}
</script>

<Header company="Project" platformName="JOAN" href="/" bind:isSideNavOpen>
	<div class="text-white hidden lg:flex pr-10">Colegio de San Juan de Dios, Incorporated</div>
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
					href="../user"
					tooltipAlignment="end"
					tooltipPosition="bottom"
					iconDescription="Dashboard"
					kind="secondary"
					icon={Dashboard}
				/>
				<Button
					tooltipAlignment="end"
					tooltipPosition="bottom"
					iconDescription="Notifications"
					kind="secondary"
					icon={Notification}
				/>
				<Button
					tooltipAlignment="end"
					tooltipPosition="bottom"
					iconDescription="User Settings"
					kind="secondary"
					icon={UserSettings}
				/>
				<Button
					tooltipAlignment="end"
					tooltipPosition="bottom"
					iconDescription="Return"
					kind="primary"
					icon={Return}
					on:click={handleBack}
				/>
			</div>
		</div>
	</HeaderUtilities>
</Header>

<SideNav bind:isOpen={isSideNavOpen}>
	<SideNavItems>
		<SideNavLink icon={Debug} href="../bugs" text="Bugs" />
		<SideNavDivider />
		<SideNavLink icon={Settings} href="../system/settings" text="System Settings" />
		<SideNavLink icon={RepoArtifact} href="../system/reports" text="System Reports" />
		<SideNavDivider />
		<SideNavLink icon={Events} href="../admin/users" text="User Management" />
		<SideNavLink icon={IbmCloudForEducation} href="../admin/subjects" text="Subject Management" />
		<SideNavLink icon={CalendarSettings} href="../admin/schedules" text="Schedule Management" />
		<SideNavLink
			icon={WatsonHealthStackedScrolling_1}
			href="../admin/bulletin"
			text="Bulletin Management"
		/>
		<SideNavDivider />
		<SideNavLink icon={Money} href="../financial/transactions" text="Financial Transactions" />
		<SideNavLink icon={Currency} href="../financial/settings" text="Financial Settings" />
		<SideNavDivider />
		<SideNavLink icon={Education} href="../academic/students" text="Students" />
		<SideNavLink icon={Book} href="../academic/subjects" text="Subjects" />
		<SideNavLink icon={NotebookReference} href="../academic/gradebook" text="Gradebook" />
		<SideNavLink icon={Friendship} href="../guidance/records" text="Guidance Records" />
		<SideNavDivider />
		<SideNavLink icon={Calendar} href="../info/schedules" text="Schedules" />
		<SideNavLink icon={WatsonHealthStackedScrolling_2} href="../info/bulletin" text="Bulletin" />
		<SideNavLink icon={Building} href="../info/school" text="The School" />
		<SideNavDivider />
		<SideNavLink icon={Box} href="../archives" text="Archives" />
		<SideNavLink icon={ReportData} href="../reports" text="Reports" />
		<SideNavDivider />
		<SideNavLink icon={InformationFilled} href="../system/about" text="System Information" />
		<SideNavLink icon={InformationFilled} href="../system/guide" text="System Guide" />
	</SideNavItems>
</SideNav>

<Content>
    
</Content>