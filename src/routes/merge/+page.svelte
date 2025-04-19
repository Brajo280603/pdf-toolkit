<script>
    import { FileUpload,Progress } from '@skeletonlabs/skeleton-svelte';
    
    // Icons
    import IconDropzone from '@lucide/svelte/icons/file-plus-2';
    import IconFile from '@lucide/svelte/icons/paperclip';
    import IconRemove from '@lucide/svelte/icons/circle-x';
    import { ArrowLeft,ArrowUp,ArrowDown } from 'lucide-svelte';
 

    

    import PDFMerger from 'pdf-merger-js/browser';

    import { GlobalWorkerOptions, getDocument } from "pdfjs-dist";

    let workerSrc = "https://unpkg.com/pdfjs-dist@5.0.375/build/pdf.worker.min.mjs";

    let file_img;
    let pdf;
    let pdf_file_url;
    let images;
    let files_remaining_percentage = 0; // for progress
    let input_pdf = [];
    let api;

    let input_file_index_counter = -1;
    let merged_file_name = "";

    
    async function  merge_pdfs(pdfs_array) {
        const merger = new PDFMerger();
        
        for(let i = 0;i<pdfs_array.length;i++){
            await merger.add(pdfs_array[i]);
        }
        

        await merger.setMetadata({
            title : `${merged_file_name}`
        })

        let merged_pdf = await merger.saveAsBlob()
        console.log(merged_pdf);
        

        // const link = document.createElement('a');]
        const link = document.querySelector("#file_downloader")
        if (link.download !== undefined) {

            const url = URL.createObjectURL(merged_pdf);

            // ___ the following is for downloading the file___
            link.setAttribute('href', url);
            link.setAttribute('download', merged_file_name.split('.')[0]);
            // link.style.visibility = 'hidden';
            // document.body.appendChild(link);
            // link.click();
            // document.body.removeChild(link);
            // link.classList.remove("hidden")


            pdf_file_url = url;// this shows the pdf file on iframe or can be used in pdfjs
        }
    }

    function button_style_controller(e){
        
        if(input_pdf.length > 0){
            document.querySelector("#ocr_btn").disabled = false
        } else {
            document.querySelector("#ocr_btn").disabled = true
        } 
        
        
        input_pdf.length == 0? document.querySelector("#file_downloader").classList.add("hidden"):null;

        files_remaining_percentage = 0;
        // input_pdf = e.acceptedFiles;// only accept new files.
        input_pdf = OnlyAcceptNewValue(e.acceptedFiles,input_pdf)
        console.log(input_pdf)

        merged_file_name = merged_file_name == ""?input_pdf[0].name.split(".")[0]:merged_file_name;

    }

    function moveFile(index, direction) {
        const files = [...input_pdf];
        const newIndex = direction === "up" ? index - 1 : index + 1;
        console.log(files)
        let temp_file = files[index];

        document.querySelector("#file_downloader").classList.contains("hidden") ? null:document.querySelector("#file_downloader").classList.add("hidden");


        if (newIndex >= 0 && newIndex < files.length) {
            [files[index], files[newIndex]] = [files[newIndex], files[index]];
            console.log(files);
            
            
            // api.setFiles(files);
            console.log(api.acceptedFiles)
            input_pdf = files;
        }
    }

    function DeleteFile(index){
        input_pdf = input_pdf.filter((_,i)=>i!=index)
    }

    function OnlyAcceptNewValue(arr1,arr2){
        let temp_arr = [];

        temp_arr = arr2;

        arr1.forEach((el,i)=>{
            if ( input_file_index_counter < i){
                temp_arr.push(el)
                input_file_index_counter++;
            }
        })

        console.log(input_file_index_counter)

        return temp_arr;
    }

    function complete(){
        merge_pdfs(input_pdf);
        document.querySelector("#file_downloader").classList.remove("hidden")
    }

</script>

<main class="flex flex-col justify-center items-center pt-5">
    <div class="w-1/2 flex flex-col gap-5">
        <FileUpload
            onApiReady={(_api) => (api = _api)}
            allowDrop={true}
            directory={true}
            maxFiles={50}
            accept="application/pdf"
            subtext="Attach PDFs"
            onFileChange={e=>{button_style_controller(e)}}
            classes="w-full"
        >
            <!-- {#snippet iconInterface()}<IconDropzone class="size-8" />{/snippet}
            {#snippet iconFile()}<IconFile class="size-4" />{/snippet}
            {#snippet iconFileRemove()}<IconRemove class="size-4" />{/snippet} -->

            <div class="btn preset-outlined-surface-500 h-48 w-full">
                
                <span>
                    Upload PDFs
                </span>
                <IconDropzone/>
            </div>
        </FileUpload>

        <div class="border-surface-500 border rounded-md h-full w-full">
            {#if input_pdf.length > 0}
            <div class="file-list p-2 flex flex-col gap-2 items-center">
              {#each input_pdf as file, index (file.name + index)}
                <div class="file-item border border-secondary-100 rounded-sm w-full flex items-center gap-5 p-2">
                  
                  <div class="controls flex">
                    <button 
                        class="cursor-pointer btn preset-ghost-tertriary-500"
                        on:click={() => moveFile(index, 'up')} 
                        disabled={index === 0}
                    >
                        <ArrowUp/>
                    </button>
                    <button 
                        class="cursor-pointer btn preset-ghost-tertriary-500"
                        on:click={() => moveFile(index, 'down')} 
                        disabled={index === input_pdf.length - 1}
                    >
                        <ArrowDown/>
                    </button>
                  </div>
        
                  
                  <div class="w-full flex gap-2">
                    <IconFile/>
                    <span >{file.name}</span>
                    <span>({Math.round(file.size / 1024)} KB)</span>
                  </div>

        
                  
                  <button on:click={() => DeleteFile(index)}>
                    <IconRemove/>
                  </button>
                </div>
              {/each}
            </div>
          {/if}
        </div>
    </div>

    <div class="w-1/2 mt-12 flex flex-col gap-12">
        <button id="ocr_btn" type="button" class=" w-full btn preset-filled-primary-500" disabled on:click={()=>{complete()}}>Merge PDFs</button>

        <!-- <Progress value={files_remaining_percentage} max={100} >{files_remaining_percentage != null?files_remaining_percentage+"%":"loading..."}</Progress> -->
         <label for="" class="label">
            <span class="label-text">Merged File Name</span>
            <input type="text" bind:value = {merged_file_name} class="input">
         </label>
        <a id="file_downloader" class="btn preset-filled-success-500 hidden" download="{merged_file_name.split('.')[0]}">Download pdf</a>
    </div>

    <a href="/" class="btn preset-filled-surface-500 rounded-full h-12 w-12 fixed top-5 left-5"><ArrowLeft class="size-8"/></a>

</main>