<script>
	import { onMount } from 'svelte';
	import { Content } from 'carbon-components-svelte';
	import { goto } from '$app/navigation';

	onMount(async () => {
		redirect();
	});

	function redirect() {
		goto('/registration/student/');
	}
</script>

<Content />
