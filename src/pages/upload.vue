<template>
  <div class="min-h-screen bg-[#F3EBF6]">
    <HeaderPage />
    <div class="container mx-auto p-4">
      <h1 class="text-center text-2xl font-bold text-[#8C55AA] mb-4">Ajouter des Photos</h1>
      <div class="max-w-md mx-auto bg-white p-8 rounded-lg shadow-lg">
        <input 
          class="block w-full text-sm text-gray-900 border border-gray-300 rounded-lg cursor-pointer bg-gray-50 focus:outline-none focus:ring-2 focus:ring-purple-600 focus:border-transparent" 
          id="multiple_files" 
          type="file" 
          multiple  
          @change="handleFileChange"
        >
        <button 
          class="w-full mt-4 bg-gradient-to-r from-purple-500 to-pink-500 hover:from-purple-600 hover:to-pink-600 text-white font-bold py-2 px-4 rounded-full shadow-lg focus:outline-none focus:ring-2 focus:ring-purple-600 focus:ring-opacity-50" 
          @click="uploadPhotos"
        >
          Upload
        </button>
        <div v-if="uploading" class="text-center mt-4">Uploading...</div>
        <div v-if="error" class="text-center mt-4 text-red-500">{{ error }}</div>
      </div>
    </div>
  </div>
  </template>
  
  <script>
  import PocketBase from 'pocketbase';
  import HeaderPage from '@/components/HeaderPage.vue';
  
  let pocketbase_ip =''
if (process.env.NODE_ENV === 'production') {
  pocketbase_ip = 'https://mariage.alexisbernardev.fr/'
} else {
  pocketbase_ip = 'http://localhost:8090'
}

  export default {
    components: {
      HeaderPage,
    },
    data() {
      return {
        files: [],
        uploading: false,
        error: null,
      };
    },
    methods: {
      handleFileChange(event) {
        this.files = Array.from(event.target.files);
      },
      async uploadPhotos() {
        const pb = new PocketBase(pocketbase_ip);
        if (!pb.authStore.isValid) {
          this.$router.push('/login');
          return;
        }
  
        this.uploading = true;
        this.error = null;
  
        try {
          const user = pb.authStore.model;
          const userTags = user.Tags;
          if (!userTags || !Array.isArray(userTags) || userTags.length === 0) {
            throw new Error("User has no tags defined.");
          }
  
          const tag = userTags[0]; // Prendre le premier tag
  
          for (let file of this.files) {
            const formData = new FormData();
            formData.append('photo', file);
            formData.append('Tags', tag);
  
            const createdRecord = await pb.collection('photos').create(formData);
            console.log('Created record:', createdRecord);
          }
          
          this.$router.push('/photos');
        } catch (error) {
          console.error('Error details:', error);
          this.error = 'Failed to upload photos: ' + error.message;
        } finally {
          this.uploading = false;
        }
      },
    },
  };
  </script>