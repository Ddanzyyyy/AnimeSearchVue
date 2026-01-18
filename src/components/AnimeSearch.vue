<template>
  <div class="max-w-2xl mx-auto p-6">
    <h1 class="text-3xl font-bold mb-6 text-center">Cari Anime</h1>
    <div class="flex gap-2 mb-6">
      <input
        v-model="query"
        @keyup.enter="searchAnime"
        placeholder="Masukkan judul anime..."
        class="flex-1 px-4 py-2 border rounded-lg focus:outline-none focus:ring-2 focus:ring-primary"
      />
      <button
        @click="searchAnime"
        class="px-6 py-2 bg-primary text-white font-semibold rounded-lg shadow hover:bg-primary-dark transition"
      >
        Cari
      </button>
    </div>
    <div v-if="loading" class="text-center py-8 text-gray-500">Loading...</div>
    <div v-if="error" class="text-red-600 text-center mb-4">{{ error }}</div>
    <div v-if="results.length" class="space-y-6">
      <h2 class="text-xl font-semibold mb-2">Hasil Pencarian:</h2>
      <div v-for="anime in results" :key="anime.mal_id" class="bg-white rounded-xl shadow p-4 flex gap-4 items-start border hover:shadow-lg transition">
        <img :src="anime.images.jpg.image_url" alt="cover" class="w-24 h-32 object-cover rounded-lg border" />
        <div class="flex-1">
          <div class="flex items-center gap-2 mb-1">
            <a :href="anime.url" target="_blank" class="text-lg font-bold text-primary hover:underline">{{ anime.title }}</a>
            <span class="text-xs bg-gray-200 rounded px-2 py-0.5 ml-2">{{ anime.type }}</span>
            <span v-if="anime.rating" class="text-xs bg-gray-100 rounded px-2 py-0.5">{{ anime.rating }}</span>
          </div>
          <div class="flex flex-wrap gap-2 mb-2">
            <span v-if="anime.score" class="text-sm text-yellow-600 font-semibold">⭐ {{ anime.score }}</span>
            <span v-if="anime.episodes" class="text-sm text-gray-600">Ep: {{ anime.episodes }}</span>
            <span v-if="anime.status" class="text-sm text-gray-600">Status: {{ anime.status }}</span>
            <span v-if="anime.year" class="text-sm text-gray-600">Tahun: {{ anime.year }}</span>
          </div>
          <div class="mb-2 text-sm text-gray-700 line-clamp-3">{{ anime.synopsis || 'Tidak ada sinopsis.' }}</div>
          <div class="flex flex-wrap gap-2 mb-1">
            <span v-for="studio in anime.studios" :key="studio.mal_id" class="text-xs bg-blue-100 text-blue-800 rounded px-2 py-0.5">{{ studio.name }}</span>
            <span v-for="genre in anime.genres" :key="genre.mal_id" class="text-xs bg-green-100 text-green-800 rounded px-2 py-0.5">{{ genre.name }}</span>
          </div>
        </div>
      </div>
    </div>
    <div v-else-if="!loading && !error && searched" class="text-center text-gray-500">Tidak ada hasil ditemukan.</div>
  </div>
</template>

<script>
export default {
  name: 'AnimeSearch',
  data() {
    return {
      query: '',
      results: [],
      loading: false,
      error: '',
      searched: false
    };
  },
  methods: {
    async searchAnime() {
      if (!this.query.trim()) return;
      this.loading = true;
      this.error = '';
      this.results = [];
      this.searched = false;
      try {
        const response = await fetch(`https://api.jikan.moe/v4/anime?q=${encodeURIComponent(this.query)}`);
        const data = await response.json();
        this.results = data.data || [];
        this.searched = true;
      } catch (e) {
        this.error = 'Gagal mengambil data.';
      } finally {
        this.loading = false;
      }
    }
  }
};
</script>

<style scoped>
@import url('https://fonts.googleapis.com/css2?family=Inter:wght@400;600;700&display=swap');
.bg-primary { background: #18181b; }
.bg-primary-dark { background: #111113; }
.text-primary { color: #18181b; }
.shadow { box-shadow: 0 2px 8px rgba(0,0,0,0.08); }
.hover\:bg-primary-dark:hover { background: #111113; }
.line-clamp-3 {
  display: -webkit-box;
  -webkit-line-clamp: 3;
  -webkit-box-orient: vertical;
  overflow: hidden;
}
</style>
