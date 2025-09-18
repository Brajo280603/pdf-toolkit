<script>
	import '../app.css';
	import { AppBar } from '@skeletonlabs/skeleton-svelte'; 
	import { page_name } from '$lib/store/page_name';
	import { Home,Github } from 'lucide-svelte';


</script>

<AppBar >
	<div class="w-full justify-between flex ">
		<!-- {#snippet lead()} -->
			<a href="/" class="ps-5"><Home size={24}/></a>
		<!-- {/snippet} -->
		
		<span>{$page_name}</span>
		
		<a href="https://github.com/Brajo280603/pdf-toolkit" class="pe-5">
			<!-- <Home size={24}/> -->
			<Github size={24}/>
		</a>
	</div>
	
	
	
</AppBar>

<slot/>
