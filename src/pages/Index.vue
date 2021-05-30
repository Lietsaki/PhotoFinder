<template>
  <q-page class="flex flex-center column">
    <q-btn color="black" label="Tap me to take a picture" @click="takePhoto" class="q-my-lg" />

    <img :src="image" v-if="image" class="q-my-lg"
      style="height: 280px; max-width: 300px" contain>
    <p class="q-my-md">{{ broughtFrom }}</p>
    <p class="q-my-md" v-if="filename">{{ filename }}</p>

    <q-form @submit="getPath" class="flex column flex-center q-mt-sm">
      <q-input filled v-model="filename" label="Enter filename with extension..." class="q-mb-lg"/>
      <q-btn type="submit" class="q-mb-md">Get that file!</q-btn>
      <p class="q-ma-md" v-if="notFound">File not found</p>
    </q-form>
  </q-page>
</template>


<script>
import { Capacitor, Plugins, CameraResultType, FilesystemDirectory, CameraSource } from '@capacitor/core';
const { Camera, Filesystem } = Plugins;

export default {
  name: 'Home',
  data() {
    return {
      image: null,
      broughtFrom: '',
      filename: null,
      notFound: false
    }
  },
  methods: {
    async takePhoto() {
      const options = {
        resultType: CameraResultType.Uri,
        source: CameraSource.Camera
      };

      const originalPhoto = await Camera.getPhoto(options);
      this.image = originalPhoto.webPath
      this.broughtFrom = 'Image brought from camera'
      const photoInTempStorage = await Filesystem.readFile({ path: originalPhoto.path });

      const fileName = Date.now() + ".jpeg";
      this.filename = fileName

      await Filesystem.writeFile({
        data: photoInTempStorage.data,
        path: fileName,
        directory: FilesystemDirectory.Data
      });

      const finalPhotoUri = await Filesystem.getUri({
        directory: FilesystemDirectory.Data,
        path: fileName
      });

      // Convert the path to a file src we can use in img src's attribute
      let photoPath = Capacitor.convertFileSrc(finalPhotoUri.uri);
    },
    async getPath() {
      if (!this.filename) return
      let file = null

      try {
        file = await Filesystem.readFile({
          path: this.filename,
          directory: FilesystemDirectory.Data
        })

        this.image = `data:image/jpeg;base64,${file.data}`
        this.broughtFrom = 'Image brought from file system'
      } catch (error) {
        if (error.message === 'File does not exist') {
          this.notFound = true
        }
        return
      }

      // Here's your blob ready to be uploaded to firebase
     const base64 = await fetch(file);
     const myBlob = await base64.blob();
     console.log(myBlob)
    }
  }
}
</script>
