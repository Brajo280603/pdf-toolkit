<script>
	import '../app.css';
	import { AppBar } from '@skeletonlabs/skeleton-svelte'; 
	import { page_name } from '$lib/store/page_name';
	import { Home,Github } from 'lucide-svelte';
	import Links from '$lib/Links.svelte'
	


</script>

<main class="bg-neutral-100 h-dvh
    dark:bg-neutral-900">
<AppBar >
	<div class="w-full justify-between flex items-center">
		<!-- {#snippet lead()} -->
		<Links Href="/" LucideIcon={Home}></Links>
		<!-- {/snippet} -->
		
		<span>{$page_name}</span>
		<Links Href="https://github.com/Brajo280603/pdf-toolkit" LucideIcon={Github}></Links>
	</div>
	
	
	
    </AppBar>
    
    <slot />
</main>
