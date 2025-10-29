<script>
    import { goto } from "$app/navigation";
    export let link
    export let name
    export let icon_component
</script>


<a href={link} class="w-full max-w-md p-4 text-center cursor-pointer h-36 flex items-center justify-center
     flex-col transition duration-400 rounded-sm
        dark:bg-neutral-800 dark:inset-shadow-neutral-500 dark:shadow-neutral-950
        inset-shadow-xs shadow-sm bg-neutral-100 inset-shadow-white
        hover:shadow-lg hover:inset-shadow-sm ">
    <div class="flex flex-col items-center gap-2">
        <svelte:component this={icon_component}/>
        <p>{name}</p>
    </div>
</a>