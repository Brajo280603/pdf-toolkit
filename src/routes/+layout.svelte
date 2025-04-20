<script>
	import '../app.css';
	import { AppBar } from '@skeletonlabs/skeleton-svelte'; 
	import { page_name } from '$lib/store/page_name';
	import { Home } from 'lucide-svelte';


</script>

<AppBar>
	{#snippet lead()}
		<a href="/" class="px-5"><Home size={24}/></a>
	{/snippet}
	
	<span>{$page_name}</span>
</AppBar>

<slot/>
