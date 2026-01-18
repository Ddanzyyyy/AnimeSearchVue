<template>
  <div class="anime-search">
    <h1>Cari Anime</h1>
    <input v-model="query" @keyup.enter="searchAnime" placeholder="Masukkan judul anime..." />
    <button @click="searchAnime">Cari</button>
    <div v-if="loading">Loading...</div>
    <div v-if="error" class="error">{{ error }}</div>
    <div v-if="results.length">
      <h2>Hasil Pencarian:</h2>
      <ul>
        <li v-for="anime in results" :key="anime.mal_id">
          <img :src="anime.images.jpg.image_url" alt="cover" width="50" />
          <strong>{{ anime.title }}</strong> ({{ anime.type }})<br />
          <span>Score: {{ anime.score }}</span>
        </li>
      </ul>
    </div>
    <div v-else-if="!loading && !error && searched">Tidak ada hasil ditemukan.</div>
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
.anime-search {
  max-width: 600px;
  margin: 40px auto;
  padding: 20px;
  background: #fff;
  border-radius: 8px;
  box-shadow: 0 2px 8px rgba(0,0,0,0.1);
}
input {
  padding: 8px;
  width: 70%;
  margin-right: 8px;
  border: 1px solid #ccc;
  border-radius: 4px;
}
button {
  padding: 8px 16px;
  background: #42b983;
  color: #fff;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}
button:hover {
  background: #369870;
}
.error {
  color: red;
  margin-top: 10px;
}
ul {
  list-style: none;
  padding: 0;
}
li {
  margin-bottom: 16px;
  display: flex;
  align-items: center;
}
li img {
  margin-right: 12px;
  border-radius: 4px;
}
</style>
