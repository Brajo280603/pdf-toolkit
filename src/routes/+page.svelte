<script>

import MenuCards from "$lib/MenuCards.svelte";

</script>

<main class="p-5 grid grid-cols-5  gap-5">
    <MenuCards link="/ocr" name="PDF OCR"/>
    <!-- <MenuCards link="/compress" name="Compress PDF"/> -->
    <MenuCards link="/split" name="Split PDF"/>
    <MenuCards link="/merge" name="Merge PDF"/>
    
</main>