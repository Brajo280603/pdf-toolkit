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


    import { page_name } from '$lib/store/page_name';

    page_name.set("Split PDFs");

    let workerSrc = "https://unpkg.com/pdfjs-dist@5.0.375/build/pdf.worker.min.mjs";
    
    let zipper = new JSZIP();
    let file_img;
    let pdf;
    let pdf_file_url;
    let images;
    let files_remaining_percentage = 0; // for progress
    let input_pdf;
    let pages_string;

    async function create_zip(file_array){
        const imagesZip = zipper.folder("images_folder")
        console.log(file_array)
        file_array.forEach(file=>{
            imagesZip.file(file.name+".pdf",file)
        })

        console.log(imagesZip);

        let zip_file = await imagesZip.generateAsync({type:"blob"})

        zipper.remove("images_folder");
        return zip_file
    }

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

    async function recognizeText(images,index,array_length) {

        console.log(images);
        try {
            const worker = await createWorker('eng', 1, {
                logger: (m) => {
                    console.log(m);
                    if(m.status = ""){}
                }
            });
            const { data } = await worker.recognize(
                images,
                { pdfTitle: `part_${index}` },
                { pdf: true }
            );
            console.log(data.text);

            pdf = data.pdf;

            worker.terminate();

            // files_remaining_percentage += (1/array_length) * 100;
            
            // files_remaining_percentage = files_remaining_percentage > 100 ? 100 : Math.floor(files_remaining_percentage);
            return Promise.resolve({
                                        "pdf_raw":data.pdf,
                                        "index":index
                                    });
        } catch (err) {
            console.error(err);
            return Promise.reject(err);
        }
    }

    function raw_data_to_files(raw_data,fileName) {
        const blob = new Blob([new Uint8Array(raw_data)], { type: 'application/pdf' });

        blob.name = fileName;
        blob.lastModifiedData = new Date();
        
        return blob;
    }

    async function  merge_pdfs(pdfs_detailed_array,file_name) {
        const merger = new PDFMerger();

        console.log(pdfs_detailed_array);
        pdfs_detailed_array.map(el=>{
            el.pdf_file = raw_data_to_files(el.pdf,`${el.index}.pdf`);
        })

        
        for(let i = 0;i<pdfs_detailed_array.length;i++){
            await merger.add(pdfs_detailed_array[i].pdf_raw);
        }

        await merger.setMetadata({
            title : `${file_name}`
        })

        let merged_pdf = await merger.saveAsBlob()
        // await merger.save()

        console.log("error not above");

        return new File([merged_pdf],file_name)
        

        // const link = document.createElement('a');]
        // const link = document.querySelector("#file_downloader")
        // if (link.download !== undefined) {

        //     const url = URL.createObjectURL(merged_pdf);

        //     // ___ the following is for downloading the file___
        //     link.setAttribute('href', url);
        //     link.setAttribute('download', file_img[0].name.split('.')[0]);
        //     // link.style.visibility = 'hidden';
        //     // document.body.appendChild(link);
        //     // link.click();
        //     // document.body.removeChild(link);
        //     // link.classList.remove("hidden")


        //     pdf_file_url = url;// this shows the pdf file on iframe or can be used in pdfjs
        // }
    }

    async function complete() {
        file_img = [];
        file_img[0]=input_pdf
        console.log(file_img);
        files_remaining_percentage = null;
        // files_remaining_percentage += 3;
        let image_arr = await convertPdfToImages(file_img[0]); //TODO: convert only required pages
        console.log(image_arr);

        let split_img_arr = split_pdf_pages(image_arr,pages_string)
        console.log(split_img_arr);

        let final_pdf_array = [];

        files_remaining_percentage = 0;

        for(let a = 0;a < split_img_arr.length;a++){

            console.log(split_img_arr[a])

            let ocr_pdfs = [];
            let images = split_img_arr[a];
            for(let i = 0;i<images.length;i++){
                await recognizeText(images[i],i,images.length).then((data_json) => {
                    ocr_pdfs.push(data_json);

                });
            }

            ocr_pdfs.sort((a,b)=>{
                    return a.index - b.index;
            })

            final_pdf_array.push(await merge_pdfs(ocr_pdfs,input_pdf.name.split(".")[0] + "_split_"+a))
            
            files_remaining_percentage += (1/split_img_arr.length) * 100;
            
            files_remaining_percentage = files_remaining_percentage > 100 ? 100 : Math.floor(files_remaining_percentage);

        }
        
        
        console.log(final_pdf_array);
        
        let zip_blob =  await create_zip(final_pdf_array);

        const link = document.querySelector("#file_downloader")
            if (link.download !== undefined) {

                const url = URL.createObjectURL(zip_blob);

                // ___ the following is for downloading the file___
                link.setAttribute('href', url);
                link.setAttribute('download', input_pdf.name.split('.')[0]);
                
            }

        // files_remaining_percentage = 0;
        document.querySelector("#file_downloader").classList.remove("hidden")
        
        files_remaining_percentage = files_remaining_percentage < 100 ? 100 : files_remaining_percentage;

        // console.log(ocr_pdfs);
    }

    function button_style_controller(e){
        
        if(e.acceptedFiles.length > 0 && !!pages_string){
            document.querySelector("#ocr_btn").disabled = false
        } else {
            document.querySelector("#ocr_btn").disabled = true
        } 
        document.querySelector("#file_downloader").classList.add("hidden");
        files_remaining_percentage = 0;
        input_pdf = e.acceptedFiles[0];
    }

    function pages_given(e){
        console.log()
        if(!!input_pdf && !!e.target.value){
            document.querySelector("#ocr_btn").disabled = false
        } else {
            document.querySelector("#ocr_btn").disabled = true
        } 
        document.querySelector("#file_downloader").classList.add("hidden");
        files_remaining_percentage = 0;

        pages_string = e.target.value;
    }

    function split_pdf_pages(pdf_array,split_string){
        let splitted_pdfs = [];

        let all_split_strings = split_string.split(",");

        all_split_strings.forEach(s_str =>{
            let indexes = s_str.split("-");
            if(indexes.length > 1){
                splitted_pdfs.push(pdf_array.slice((indexes[0]-1),indexes[1]));
            }else{
                splitted_pdfs.push(pdf_array.slice((indexes[0]-1),(indexes[0])));
            }
        })

        return splitted_pdfs;
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
    
        <label class="label">
            <span class="label-text">Pages to Extract</span>
            <input class="input" on:input={(e)=>{pages_given(e)}} bind:value={pages_string} type="text" placeholder="Example: 1, 5-8" />
        </label>
        
        <button id="ocr_btn" type="button" class=" w-full btn preset-filled-primary-500" disabled on:click={()=>{complete()}}>Split PDFs</button>
        
        <Progress value={files_remaining_percentage} max={100} >{files_remaining_percentage != null?files_remaining_percentage+"%":"loading..."}</Progress>
        
    <a id="file_downloader" class="btn preset-filled-success-500 hidden" >Download Pdfs</a>
</div>



</main>