<script>
	// components
	import {
		Button,
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
		Education,
		Friendship,
		IbmCloudForEducation,
		InformationFilled,
		Logout,
		Money,
		NotebookReference,
		Notification,
		RepoArtifact,
		ReportData,
		Settings,
		UserAdmin,
		UserMultiple,
		UserSettings,
		WatsonHealthStackedScrolling_1,
		WatsonHealthStackedScrolling_2
	} from 'carbon-icons-svelte';

	// pictograms

	// variables
	let isSideNavOpen = false;
</script>

<Header company="Project" platformName="JOAN" href="/" bind:isSideNavOpen>
	<div class="text-white hidden lg:flex pr-10">Colegio de San Juan de Dios, Incorporated</div>
	<HeaderUtilities>
		<div class="flex">
			<div class="flex">
				<Button
					tooltipPosition="left"
					iconDescription="Account Settings"
					kind="secondary"
					icon={UserSettings}
				/>
				<Button
					class="hidden lg:flex"
					tooltipPosition="left"
					iconDescription="System Guide"
					kind="secondary"
					icon={NotebookReference}
				/>
				<Button
					tooltipPosition="left"
					iconDescription="Logout"
					kind="danger-tertiary"
					icon={Logout}
				/>
			</div>
		</div>
	</HeaderUtilities>
</Header>

<SideNav bind:isOpen={isSideNavOpen}>
	<SideNavItems>
		<SideNavLink icon={Dashboard} href="../user" text="User Dashboard" />
		<SideNavLink isSelected icon={UserSettings} href="../user/settings" text="User Settings" />
		<SideNavLink icon={Notification} href="../user/notifications" text="Notifications" />
		<SideNavDivider />
		<SideNavLink icon={Dashboard} href="../system" text="System Dashboard" />
		<SideNavLink icon={Settings} href="../system/settings" text="System Settings" />
		<SideNavLink icon={RepoArtifact} href="../system/reports" text="System Reports" />
		<SideNavDivider />
		<SideNavLink icon={Dashboard} href="../admin" text="Administrator Dashboard" />
		<SideNavLink icon={UserMultiple} href="../admin/users" text="User Management" />
		<SideNavLink icon={IbmCloudForEducation} href="../admin/subjects" text="Subject Management" />
		<SideNavLink icon={CalendarSettings} href="../admin/schedules" text="Schedule Management" />
		<SideNavLink icon={WatsonHealthStackedScrolling_1} href="../admin/bulletin" text="Bulletin Management" />
		<SideNavDivider />
		<SideNavLink icon={Dashboard} href="../financial" text="Financial Dashboard" />
		<SideNavLink icon={Money} href="../financial/transactions" text="Financial Transactions" />
		<SideNavLink icon={Currency} href="../financial/settings" text="Financial Settings" />
		<SideNavDivider />
		<SideNavLink icon={Dashboard} href="../academic" text="Academic Dashboard" />
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
