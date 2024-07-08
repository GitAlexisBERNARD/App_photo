<template>
  <div class="flex items-center justify-center min-h-screen bg-[#F3EBF6]">
    <div class="bg-white lg:w-[32rem] h-auto p-8 rounded-lg shadow-lg">
      <h1 class="text-center text-2xl font-bold text-[#8C55AA] mb-8">Album photo mariage</h1>
      <h2 class="text-center text-ml font-medium text-[#edccff] mb-8">Bienvenue sur le site des photos de mariage de Gaelle et Nicolas. Pour accéder aux photos, vous pouvez vous connecter avec les identifiants reçus sur la carte de remerciement</h2>
      <form @submit.prevent="login" class="space-y-6">
        <div>
          <input 
            class="w-full py-3 px-4 bg-[#F3EBF6] text-gray-800 text-sm font-medium rounded-lg border border-transparent focus:border-[#8C55AA] focus:outline-none" 
            id="username" 
            v-model="username" 
            type="text" 
            placeholder="Nom d'utilisateur"
          />
        </div>
        <div>
          <input 
            class="w-full py-3 px-4 bg-[#F3EBF6] text-gray-800 text-sm font-bold rounded-lg border border-transparent focus:border-[#8C55AA] focus:outline-none" 
            id="password" 
            type="password" 
            v-model="password" 
            placeholder="Mot de passe"
          />
        </div>
        <div class="flex items-center justify-center">
          <button 
            class="py-3 px-8 bg-gradient-to-r from-purple-500 to-pink-500 text-white font-bold rounded-full shadow-lg focus:outline-none focus:ring-2 focus:ring-purple-600 focus:ring-opacity-50" 
            type="submit">
            Connexion
          </button>
        </div>
      </form>
    </div>
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
        username: '',
        password: '',
      };
    },
    methods: {
      async login() {
        const pb = new PocketBase(pocketbase_ip);
        try {
          await pb.collection('users').authWithPassword(this.username, this.password);
          this.$router.push('/photos');
        } catch (error) {
          alert('Login failed: ' + error.message);
        }
      },
    },
  };
  </script>
  