<script lang="ts">
  import { Cloudinary } from "@cloudinary/url-gen"
  import { backgroundRemoval } from "@cloudinary/url-gen/actions/effect"

  import { ImageStatus } from "../types.d"
  import { imageStatus, modifiedImage, originalImage } from "./store.js"
  import Dropzone from "dropzone"
  import "dropzone/dist/dropzone.css"

  import { onMount } from "svelte"

  const cloudinary = new Cloudinary({
    cloud: {
      cloudName: "midudev",
    },
    url: {
      secure: true,
    },
  })

  onMount(() => {
    const dropzone = new Dropzone("#dropzone", {
      uploadMultiple: false,
      acceptedFiles: ".jpg,.png,.webp",
      maxFiles: 1,
    })

    dropzone.on("sending", (file, xhr, formData) => {
      imageStatus.set(ImageStatus.UPLOADING)
      // aqui podemos añadir la apiKey, configuracion
      formData.append("upload_preset", "fh4kcx3r")
      formData.append("timestamp", Date.now() / 1000)
      formData.append("api_key", 898535479927365)
    })

    dropzone.on("success", (file, response) => {
      const { public_id: publicId, secure_url: url } = response

      // crear la imagen con fondo transparente
      // y guardar en el backgroundImage
      const imageWithoutBackground = cloudinary
        .image(publicId)
        .effect(backgroundRemoval())

      console.log(imageWithoutBackground.toURL())

      imageStatus.set(ImageStatus.DONE)
      modifiedImage.set(imageWithoutBackground.toURL())
      originalImage.set(url)
    })

    dropzone.on("error", (file, response) => {
      console.log("HA IDO MAL")
      console.log(response)
    })
  })
</script>

<form
  id="dropzone"
  class="shadow-2xl border-dashed border-2 border-gray-300 rounded-lg aspect-video w-full flex items-center justify-center flex-col"
  action="https://api.cloudinary.com/v1_1/midudev/image/upload"
>
  {#if $imageStatus === ImageStatus.READY}
    <button
      class="font-bold pointer-events-none bg-blue-600 rounded-full text-bold text-white text-xl px-6 py-4"
    >
      Upload files
    </button>
    <strong class="text-lg mt-4 text-gray-800">or drop a file</strong>
  {:else if $imageStatus === ImageStatus.UPLOADING}
    <strong class="text-lg mt-4 text-gray-800">Uploading files...</strong>
  {/if}
</form>
