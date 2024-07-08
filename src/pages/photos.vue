<template>
  <HeaderPage />
  <div class="container mx-auto p-4 bg-[#F3EBF6] min-h-screen">
    <h1 class="text-center text-2xl font-bold text-[#8C55AA] mb-4 hidden lg:block">Album photo mariage</h1>
    <h2 class="text-center text-ml font-medium text-[#8C55AA] mb-8">Bienvenue sur le site des photos de mariage de Gaelle et Nicolas</h2>
    <div v-if="loading" class="text-center">Chargement...</div>
    <div v-else>
      <div v-if="photos.length === 0" class="text-center">Pas de photos disponibles</div>
      <div class="flex justify-center md:justify-start">
        <button @click="downloadAllPhotos" class="mb-4 bg-gradient-to-r from-purple-500 to-pink-500 hover:from-purple-600 hover:to-pink-600 text-white font-bold py-2 px-4 rounded-full shadow-lg focus:outline-none focus:ring-2 focus:ring-purple-600 focus:ring-opacity-50">
          Télécharger toutes les photos
        </button>
      </div>
      <div class="grid grid-cols-1 sm:grid-cols-2 md:grid-cols-3 lg:grid-cols-4 gap-4">
        <div v-for="photo in photos" :key="photo.id" class="photo-container">
          <img :class="{'cursor-pointer': !isMobile}" @click="!isMobile && openModal(photo)" class="w-full h-48 object-cover rounded-xl" :src="getPhotoUrl(photo)" :alt="photo.id" />
        </div>
      </div>
    </div>
    
    <div v-if="isDownloading" class="fixed inset-0 flex items-center justify-center bg-black bg-opacity-50 z-50">
      <div class="bg-white p-6 rounded-lg shadow-lg text-center">
        <h3 class="text-xl font-bold mb-4">Téléchargement en cours...</h3>
        <p>Veuillez patienter pendant les photos sont en téléchargement</p>
        <div class="mt-4">
          <svg class="animate-spin h-5 w-5 text-purple-500 mx-auto" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24">
            <circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4"></circle>
            <path class="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4z"></path>
          </svg>
        </div>
      </div>
    </div>
    
    <div v-if="modalPhoto" @click="closeModal" class="fixed inset-0 flex items-center justify-center bg-black bg-opacity-75 z-50">
      <div class="relative bg-white p-4 rounded-lg shadow-lg max-w-3xl mx-auto" @click.stop>
        <button @click="closeModal" class="absolute top-2 right-2 text-gray-700 hover:text-gray-800">
          <svg class="w-6 h-6" fill="none" stroke="currentColor" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12"></path>
          </svg>
        </button>
        <img :src="getPhotoUrl(modalPhoto)" class="w-full h-auto rounded-lg" :alt="modalPhoto.id" />
      </div>
    </div>
  </div>
</template>

<script>
import PocketBase from 'pocketbase';
import JSZip from 'jszip';
import saveAs from 'file-saver';
import HeaderPage from '@/components/HeaderPage.vue';

let pocketbase_ip =''
if (process.env.NODE_ENV === 'production') {
  pocketbase_ip = 'https://mariage.alexisbernardev.fr/'
} else {
  pocketbase_ip = 'http://localhost:8090/'
}

export default {
  components: {
    HeaderPage,
  },
  data() {
    return {
      photos: [],
      loading: true,
      isDownloading: false,
      modalPhoto: null,
      isMobile: false, // Add this line
    };
  },
  methods: {
    getPhotoUrl(photo) {
      return `${pocketbase_ip}api/files/photos/${photo.id}/${photo.photo}`;
    },
    async downloadAllPhotos() {
      this.isDownloading = true;
      const zip = new JSZip();
      const folder = zip.folder("photos");

      const promises = this.photos.map(async photo => {
        const url = this.getPhotoUrl(photo);
        const response = await fetch(url);
        const blob = await response.blob();
        folder.file(photo.photo, blob);
      });

      await Promise.all(promises);

      zip.generateAsync({ type: "blob" }).then(content => {
        saveAs(content, "photos.zip");
        this.isDownloading = false;
      }).catch(() => {
        this.isDownloading = false;
      });
    },
    openModal(photo) {
      this.modalPhoto = photo;
    },
    closeModal() {
      this.modalPhoto = null;
    },
    checkIfMobile() {
      this.isMobile = window.innerWidth < 768;
    }
  },
  mounted() {
    this.checkIfMobile();
    window.addEventListener('resize', this.checkIfMobile);
  },
  beforeDestroy() {
    window.removeEventListener('resize', this.checkIfMobile);
  },
  async created() {
    const pb = new PocketBase(pocketbase_ip);
    if (!pb.authStore.isValid) {
      this.$router.push('/');
      return;
    }

    try {
      const user = pb.authStore.model;
      const userTags = user.Tags;
      const filters = userTags.map(tag => `Tags ~ "${tag}"`).join(' || ');
      const resultList = await pb.collection('photos').getFullList({
        filter: filters,
      });
      this.photos = resultList;
    } catch (error) {
      alert('Failed to load photos: ' + error.message);
    } finally {
      this.loading = false;
    }
  },
};
</script>
