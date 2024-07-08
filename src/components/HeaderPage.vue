<template>
    <header class="bg-white p-4">
      <div class="container mx-auto flex justify-between items-center">
        <div class="text-xl font-bold text-[#8C55AA]">Album Photo Mariage</div>
        <div class="block lg:hidden">
          <button @click="toggleMenu" class="focus:outline-none text-[#8C55AA]">
            <svg class="w-6 h-6" fill="none" stroke="currentColor" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4 6h16M4 12h16M4 18h16"></path>
            </svg>
          </button>
        </div>
        <nav class="hidden lg:flex space-x-4">
          <RouterLink to="/photos" class="text-[#8C55AA] hover:text-[#6B3B8E]">Photos</RouterLink>
          <RouterLink to="/upload" class="text-[#8C55AA] hover:text-[#6B3B8E]">Ajoute des photos</RouterLink>
          <button @click="logout" class="py-2 px-4 bg-gradient-to-r from-purple-500 to-pink-500 text-white font-bold rounded-full shadow-lg focus:outline-none focus:ring-2 focus:ring-purple-600 focus:ring-opacity-50">
            Déconnexion
          </button>
        </nav>
      </div>
    </header>

    <div v-if="menuOpen" class="fixed inset-0 bg-white z-50 flex flex-col items-center justify-center space-y-6 p-4">
      <button @click="toggleMenu" class="absolute top-4 right-4 text-[#8C55AA]">
        <svg class="w-8 h-8" fill="none" stroke="currentColor" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg">
          <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12"></path>
        </svg>
      </button>
      <RouterLink @click="toggleMenu" to="/photos" class="text-2xl text-[#8C55AA] hover:text-[#6B3B8E]">Photos</RouterLink>
      <RouterLink @click="toggleMenu" to="/upload" class="text-2xl text-[#8C55AA] hover:text-[#6B3B8E]">Ajoute des photos</RouterLink>
      <button @click="logout" class="py-2 px-4 bg-gradient-to-r from-purple-500 to-pink-500 text-white font-bold rounded-full shadow-lg focus:outline-none focus:ring-2 focus:ring-purple-600 focus:ring-opacity-50">
        Déconnexion
      </button>
    </div>
</template>


<script>
import PocketBase from 'pocketbase';

let pocketbase_ip =''
if (process.env.NODE_ENV === 'production') {
  pocketbase_ip = 'https://mariage.alexisbernardev.fr/'
} else {
  pocketbase_ip = 'http://localhost:8090'
}

export default {
  data() {
    return {
      menuOpen: false,
    };
  },
  methods: {
    toggleMenu() {
      this.menuOpen = !this.menuOpen;
    },
    async logout() {
      const pb = new PocketBase(pocketbase_ip);
      pb.authStore.clear();
      this.$router.push('/');
    },
  },
};
</script>

