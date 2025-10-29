<script>
    let {Class,LucideIcon,Href} = $props()
</script>

<a href="{Href}" class="p-2 flex flex-col items-center justify-evenly rounded-full transition duration-400
        dark:bg-neutral-800 dark:inset-shadow-neutral-500 dark:shadow-neutral-950
        inset-shadow-xs shadow-sm bg-neutral-100 inset-shadow-white
        hover:shadow-lg hover:inset-shadow-sm {Class}">
            <!-- <Home size={24}/>    -->
            <LucideIcon></LucideIcon>
</a>