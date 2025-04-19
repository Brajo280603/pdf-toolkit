<script>
    import { goto } from "$app/navigation";
    export let link
    export let name
    export let icon_component
</script>


<div on:click={goto(link)} class="card w-full max-w-md preset-outlined-surface-500 p-4 text-center cursor-pointer h-36 flex items-center justify-center">
    <div class="flex flex-col items-center gap-2">
        <svelte:component this={icon_component}/>
        <p>{name}</p>
    </div>
</div>