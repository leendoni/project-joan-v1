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
		<SideNavLink icon={Dashboard} href="../user/" text="User Dashboard" />
		<SideNavLink icon={UserSettings} href="" text="User Settings" />
		<SideNavLink icon={Notification} href="" text="Notifications" />
		<SideNavDivider />
		<SideNavLink icon={Settings} href="" text="System Settings" />
		<SideNavLink icon={RepoArtifact} href="" text="System Reports" />
		<SideNavDivider />
		<SideNavLink icon={UserAdmin} href="" text="Administrator Dashboard" />
		<SideNavDivider />
		<SideNavLink icon={UserMultiple} href="" text="User Management" />
		<SideNavLink icon={IbmCloudForEducation} href="" text="Subject Management" />
		<SideNavLink icon={CalendarSettings} href="" text="Schedule Management" />
		<SideNavLink icon={WatsonHealthStackedScrolling_1} href="" text="Bulletin Management" />
		<SideNavDivider />
		<SideNavLink icon={Money} href="" text="Financial Transactions" />
		<SideNavLink icon={Currency} href="" text="Financial Settings" />
		<SideNavDivider />
		<SideNavLink icon={Education} href="" text="Students" />
		<SideNavLink icon={Book} href="" text="Subjects" />
		<SideNavLink icon={NotebookReference} href="" text="Gradebook" />
		<SideNavLink icon={Friendship} href="" text="Guidance Records" />
		<SideNavDivider />
		<SideNavLink icon={Calendar} href="" text="Schedules" />
		<SideNavLink icon={WatsonHealthStackedScrolling_2} href="" text="Bulletin" />
		<SideNavLink icon={Building} href="" text="The School" />
		<SideNavDivider />
		<SideNavLink icon={Box} href="" text="Archives" />
		<SideNavLink icon={ReportData} href="" text="Reports" />
	</SideNavItems>
</SideNav>
