<script>
    import { FileUpload,Progress } from '@skeletonlabs/skeleton-svelte';
    import IconDropzone from '@lucide/svelte/icons/file-plus-2';
    import IconFile from '@lucide/svelte/icons/paperclip';
    import IconRemove from '@lucide/svelte/icons/circle-x';
    import { ArrowLeft,ArrowUp,ArrowDown } from 'lucide-svelte';
    import JSZIP from 'jszip';
    import imageCompression from "browser-image-compression";


    //changing page name
    import { page_name } from '$lib/store/page_name';

    page_name.set("Image Compress");
    
    let zipper = new JSZIP();
    let imageSize;
    let file_img;
    let pdf;
    let pdf_file_url;
    let images;
    let files_remaining_percentage = 0; // for progress
    let input_pdf = [];
    let api;
    let many_images = false;
    let optionSelect = "Size"; // Set a default value

    let input_file_index_counter = -1;
    let merged_file_name = "";





    async function imageCompressorFunction(image) {
        // document.querySelector(".images").innerHTML = ``;
        imageSize = imageSize ? imageSize : 1024;

        const imageFile = image;
        console.log("originalFile instanceof Blob", imageFile instanceof Blob); // true
        console.log(`originalFile size ${imageFile.size / 1024 / 1024} MB`);

        const options = {
            maxSizeMB: imageSize / 1024,
            // maxWidthOrHeight: 1920,
            useWebWorker: true,
        };

        try {
            const compressedFile = await imageCompression(imageFile, options);
            console.log(
                "compressedFile instanceof Blob",
                compressedFile instanceof Blob
            ); // true
            console.log(
                `compressedFile size ${compressedFile.size / 1024 / 1024} MB`
            ); // smaller than maxSizeMB

            return await compressedFile;
        } catch (error) {
            console.log(error);
        }
    }

    async function complete() {
        many_images= false;
        // file_pdf = [];
        // file_pdf[0]=input_pdf

        let images = input_pdf;

        // console.log(file_pdf);
        files_remaining_percentage = null;
        // files_remaining_percentage += 3;
        // let images = await convertPdfToImages(file_pdf[0]); //converts pdf to images
        // console.log(images);

        



        let image_file_array = [];

        for(let i = 0;i<images.length;i++){
            // console.log(images[i])
            // image_file_array.push(await base64ToFile(images[i],input_pdf.name.split('.')[0]+"_page_"+(i+1))); //converts each images base64 strings to jpgfiles
            
            image_file_array.push(await imageCompressorFunction(images[i]))

            if(i > 1){
                many_images = true;
            }

            console.log((i/images.length) * 100)
            files_remaining_percentage = (i/images.length) * 100;
        }

        console.log(image_file_array)


        let zip_blob;
        if(many_images){
            zip_blob =  await create_zip(image_file_array);
        }else{
            // image_file_array[0].name += ".jpg"
            zip_blob = image_file_array[0];
        }


        const link = document.querySelector("#file_downloader")
            if (link.download !== undefined) {

                const url = URL.createObjectURL(zip_blob);

                // ___ the following is for downloading the file___
                link.setAttribute('href', url);
                if(many_images){
                    link.setAttribute('download', input_pdf[0].name.split('.')[0]);
                }else{
                    console.log(input_pdf)
                    
                    // input_pdf = input_pdf[0];
                    link.setAttribute('download', input_pdf[0].name.split('.')[0] + ".jpg");
                }
                
            }

        files_remaining_percentage = 100;

        document.querySelector("#file_downloader").classList.remove("hidden");
    }

    async function create_zip(file_array){
        const imagesZip = zipper.folder("images_folder")
        // console.log(file_array)
        file_array.forEach(file=>{
            imagesZip.file(file.name+".jpg",file)
        })

        // console.log(imagesZip);

        let zip_file = await imagesZip.generateAsync({type:"blob"})

        zipper.remove("images_folder");
        return zip_file
    }


    // file_drop functions
    function button_style_controller(e){
        

        
        
        input_pdf.length == 0? document.querySelector("#file_downloader").classList.add("hidden"):null;

        files_remaining_percentage = 0;
        console.log(e.acceptedFiles)
        // input_pdf = e.acceptedFiles;// only accept new files.
        input_pdf = OnlyAcceptNewValue(e.acceptedFiles,input_pdf)
        console.log(input_pdf)

        merged_file_name = merged_file_name == ""?input_pdf[0].name.split(".")[0]:merged_file_name;
        
        if(input_pdf.length > 0){
            document.querySelector("#ocr_btn").disabled = false
        } else {
            document.querySelector("#ocr_btn").disabled = true
        } 
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
                console.log(el)
                temp_arr.push(el)
                input_file_index_counter++;
            }
        })

        console.log(input_file_index_counter)
        console.log(temp_arr)

        return temp_arr;
    }



</script>

<main class="flex flex-col justify-center items-center  py-5">
    <div class="w-1/2 flex flex-col gap-5">
        <FileUpload
            maxFiles={50}
            accept="image/jpeg"
            subtext="Attach Images"
            onFileChange={e=>{button_style_controller(e)}}
            classes="w-full"
            capture
        >
            <!-- {#snippet iconInterface()}<IconDropzone class="size-8" />{/snippet}
            {#snippet iconFile()}<IconFile class="size-4" />{/snippet}
            {#snippet iconFileRemove()}<IconRemove class="size-4" />{/snippet} -->

            <div class="btn preset-outlined-surface-500 h-48 w-full">
                
                <span>
                    Upload Images
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
        <label for="" class="label">
            <span class="label-text">File Size In KB</span>
            <input type="number" bind:value = {merged_file_name} class="input">
         </label>


        <button id="ocr_btn" type="button" class=" w-full btn preset-filled-primary-500" disabled on:click={()=>{complete()}}>Start Compressing</button>

        <Progress value={files_remaining_percentage} max={100} >{files_remaining_percentage != null?files_remaining_percentage+"%":"loading..."}</Progress>
         <!-- <label for="" class="label">
            <span class="label-text">PDF File Name</span>
            <input type="text" bind:value = {merged_file_name} class="input">
         </label> -->
        <a id="file_downloader" class="btn preset-filled-success-500 hidden" >Download image{many_images ? "s zip" : ""}</a>
    </div>

    

</main>