<script>
    import { FileUpload,Progress } from '@skeletonlabs/skeleton-svelte';
    // Icons
    import IconDropzone from '@lucide/svelte/icons/file-plus-2';
    import IconFile from '@lucide/svelte/icons/paperclip';
    import IconRemove from '@lucide/svelte/icons/circle-x';
    import { ArrowLeft } from 'lucide-svelte';

    
    import { createWorker } from 'tesseract.js';
    import PDFMerger from 'pdf-merger-js/browser';

    import JSZIP from 'jszip';

    import { GlobalWorkerOptions, getDocument } from "pdfjs-dist";

    let workerSrc = "https://unpkg.com/pdfjs-dist@5.0.375/build/pdf.worker.min.mjs";

    let zipper = new JSZIP();
    let file_pdf;
    let pdf;
    let pdf_file_url;
    let images;
    let files_remaining_percentage = 0; // for progress
    let input_pdf;

    async function convertPdfToImages(file) {

        GlobalWorkerOptions.workerSrc = workerSrc;

        const reader = new FileReader();
        reader.readAsArrayBuffer(file);

        return new Promise((resolve) => {
            reader.onload = async () => {
                const arrayBuffer = reader.result;
                const pdf = await pdfjsLib.getDocument({ data: arrayBuffer }).promise;
                let pages = [];

                for (let i = 1; i <= pdf.numPages; i++) {
                    const page = await pdf.getPage(i);
                    const viewport = page.getViewport({ scale: 2 });
                    const canvas = document.createElement('canvas');
                    const context = canvas.getContext('2d');
                    canvas.width = viewport.width;
                    canvas.height = viewport.height;

                    await page.render({ canvasContext: context, viewport }).promise;
                    pages.push(canvas.toDataURL('image/png'));
                }

                resolve(pages);
            };
        });
    }

    async function base64ToFile(dataUrl,fileName){
       const res = await fetch(dataUrl);
       const blob = await res.blob();
       return new File([blob],fileName,{type: "image/jpg"}) 
    }

    async function create_zip(file_array){
        const imagesZip = zipper.folder("images_folder")
        console.log(file_array)
        file_array.forEach(file=>{
            imagesZip.file(file.name+".jpg",file)
        })

        console.log(imagesZip);

        let zip_file = await imagesZip.generateAsync({type:"blob"})

        zipper.remove("images_folder");
        return zip_file
    }

    async function complete() {
        file_pdf = [];
        file_pdf[0]=input_pdf
        console.log(file_pdf);
        files_remaining_percentage = null;
        // files_remaining_percentage += 3;
        let images = await convertPdfToImages(file_pdf[0]); //converts pdf to images
        console.log(images);

        let image_file_array = [];

        for(let i = 0;i<images.length;i++){
            console.log(images[i])
            image_file_array.push(await base64ToFile(images[i],input_pdf.name.split('.')[0]+"_page_"+(i+1))); //converts each images base64 strings to jpgfiles
        }

        console.log(image_file_array)

        let zip_blob =  await create_zip(image_file_array);

        const link = document.querySelector("#file_downloader")
            if (link.download !== undefined) {

                const url = URL.createObjectURL(zip_blob);

                // ___ the following is for downloading the file___
                link.setAttribute('href', url);
                link.setAttribute('download', input_pdf.name.split('.')[0]);
                
            }

        files_remaining_percentage = 100;

        document.querySelector("#file_downloader").classList.remove("hidden");
    }

    function button_style_controller(e){
        
        if(e.acceptedFiles.length > 0){
            document.querySelector("#ocr_btn").disabled = false
        } else {
            document.querySelector("#ocr_btn").disabled = true
        } 
        document.querySelector("#file_downloader").classList.add("hidden");
        files_remaining_percentage = 0;
        input_pdf = e.acceptedFiles[0];
    }
</script>

<main class="flex flex-col justify-center items-center pt-5">
<div class="w-1/2">
    <FileUpload
    accept="application/pdf"
    maxFiles={1}
    subtext="Attach single PDF."
    onFileChange={e=>{button_style_controller(e)}}
    classes="w-full"
  >
    {#snippet iconInterface()}<IconDropzone class="size-8" />{/snippet}
    {#snippet iconFile()}<IconFile class="size-4" />{/snippet}
    {#snippet iconFileRemove()}<IconRemove class="size-4" />{/snippet}
  </FileUpload>
</div>
<div class="w-1/2 mt-12 flex flex-col gap-12">
    <button id="ocr_btn" type="button" class=" w-full btn preset-filled-primary-500" disabled on:click={()=>{complete()}}>Convert To JPG</button>

    <Progress value={files_remaining_percentage} max={100} >{files_remaining_percentage != null?files_remaining_percentage+"%":"loading..."}</Progress>

    <a id="file_downloader" class="btn preset-filled-success-500 hidden" >Download images zip</a>
</div>

<a href="/" class="btn preset-filled-surface-500 rounded-full h-12 w-12 fixed top-5 left-5"><ArrowLeft class="size-8"/></a>

</main>