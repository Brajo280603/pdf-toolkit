<script>
    import { goto } from "$app/navigation";
    export let link
    export let name
</script>

<div on:click={goto(link)} class="card w-full max-w-md preset-outlined-surface-500 p-4 text-center cursor-pointer ">
    <p>{name}</p>
</div>