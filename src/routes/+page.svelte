<script>

import MenuCards from "$lib/MenuCards.svelte";


import { FileSearch,FileStack,SquareSplitHorizontal,WholeWord,FileImage,Projector,Sheet,ImageUp,Text,PanelLeftOpen,NotebookTabs,Image } from "lucide-svelte";

import { page_name } from '$lib/store/page_name';

page_name.set("pdf-toolkit")
</script>

<main class="p-5 grid grid-cols-2 md:grid-cols-4 lg:grid-cols-5 sm:grid-cols-3  gap-5">
    <MenuCards link="/ocr" name="PDF OCR" icon_component={FileSearch}/>
    <MenuCards link="/merge" name="Merge PDF" icon_component={FileStack}/>
    <MenuCards link="/split" name="Split PDF" icon_component={SquareSplitHorizontal}/>
    <MenuCards link="/pdf_to_jpg" name="PDF to JPG" icon_component={FileImage}/>
    <MenuCards link="/jpg_to_pdf" name="JPG to PDF" icon_component={ImageUp}/>
    <!-- <MenuCards link="/pdf_to_word" name="PDF to Word" icon_component={Text}/>
    <MenuCards link="/word_to_pdf" name="Word to PDF" icon_component={WholeWord}/>
    <MenuCards link="/pdf_to_ppt" name="PDF to PPT" icon_component={Projector}/>
    <MenuCards link="/ppt_to_pdf" name="PPT to PDF" icon_component={PanelLeftOpen}/>
    <MenuCards link="/pdf_to_excel" name="PDF to Excel" icon_component={Sheet}/>
    <MenuCards link="/excel_to_pdf" name="Excel to PDF" icon_component={NotebookTabs}/> -->
    <MenuCards link="/image_compress" name="Image Compress" icon_component={Image}/>
    
</main>