<template>
  <div class="search-container">
    <div class="search-header">
      <h1 class="title">Discover Anime</h1>
      <p class="subtitle">Find your next favorite anime</p>
      <div class="search-box">
        <input
          v-model="query"
          @keyup.enter="searchAnime"
          placeholder="Search anime by title..."
          class="search-input"
        />
        <button @click="searchAnime" class="search-btn">
          <svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><circle cx="11" cy="11" r="8"></circle><path d="m21 21-4.35-4.35"></path></svg>
          Search
        </button>
      </div>
    </div>

    <div v-if="loading" class="loader">
      <div class="spinner"></div>
      <p>Loading...</p>
    </div>

    <div v-if="error" class="error-message">{{ error }}</div>

    <div v-if="results.length" class="results">
      <div class="anime-grid">
        <div
          v-for="anime in results"
          :key="anime.mal_id"
          class="anime-card"
          @click="openDetail(anime)"
        >
          <div class="card-image">
            <img :src="anime.images.jpg.image_url" :alt="anime.title" />
            <div class="card-overlay">
              <div class="score" v-if="anime.score">
                <svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" viewBox="0 0 24 24" fill="currentColor"><path d="M12 2l3.09 6.26L22 9.27l-5 4.87 1.18 6.88L12 17.77l-6.18 3.25L7 14.14 2 9.27l6.91-1.01L12 2z"></path></svg>
                {{ anime.score }}
              </div>
            </div>
          </div>
          <div class="card-content">
            <h3 class="card-title">{{ anime.title }}</h3>
            <div class="card-meta">
              <span class="badge">{{ anime.type }}</span>
              <span class="episodes" v-if="anime.episodes">{{ anime.episodes }} eps</span>
            </div>
          </div>
        </div>
      </div>
    </div>

    <div v-else-if="!loading && !error && searched" class="no-results">
      <svg xmlns="http://www.w3.org/2000/svg" width="64" height="64" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"><circle cx="11" cy="11" r="8"></circle><path d="m21 21-4.35-4.35"></path></svg>
      <p>No results found</p>
    </div>

    <!-- Modal -->
    <transition name="modal">
      <div v-if="showDetail" class="modal-backdrop" @click="closeDetail">
        <div class="modal" @click.stop>
          <button @click="closeDetail" class="close-btn">
            <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><line x1="18" y1="6" x2="6" y2="18"></line><line x1="6" y1="6" x2="18" y2="18"></line></svg>
          </button>
          <div class="modal-content">
            <div class="modal-header">
              <img :src="selectedAnime.images.jpg.large_image_url || selectedAnime.images.jpg.image_url" :alt="selectedAnime.title" class="modal-image" />
              <div class="modal-info">
                <h2 class="modal-title">{{ selectedAnime.title }}</h2>
                <div class="modal-badges">
                  <span class="badge">{{ selectedAnime.type }}</span>
                  <span class="badge" v-if="selectedAnime.status">{{ selectedAnime.status }}</span>
                  <span class="badge" v-if="selectedAnime.year">{{ selectedAnime.year }}</span>
                </div>
                <div class="modal-stats">
                  <div class="stat" v-if="selectedAnime.score">
                    <svg xmlns="http://www.w3.org/2000/svg" width="18" height="18" viewBox="0 0 24 24" fill="currentColor"><path d="M12 2l3.09 6.26L22 9.27l-5 4.87 1.18 6.88L12 17.77l-6.18 3.25L7 14.14 2 9.27l6.91-1.01L12 2z"></path></svg>
                    <span>{{ selectedAnime.score }}</span>
                  </div>
                  <div class="stat" v-if="selectedAnime.episodes">
                    <svg xmlns="http://www.w3.org/2000/svg" width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><polygon points="5 3 19 12 5 21 5 3"></polygon></svg>
                    <span>{{ selectedAnime.episodes }} episodes</span>
                  </div>
                  <div class="stat" v-if="selectedAnime.rating">
                    <svg xmlns="http://www.w3.org/2000/svg" width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M12 2a10 10 0 1 0 10 10H12V2z"></path></svg>
                    <span>{{ selectedAnime.rating }}</span>
                  </div>
                </div>
              </div>
            </div>
            <div class="modal-body">
              <div class="section">
                <h3>Synopsis</h3>
                <p>{{ selectedAnime.synopsis || 'No synopsis available.' }}</p>
              </div>
              <div class="section" v-if="selectedAnime.genres && selectedAnime.genres.length">
                <h3>Genres</h3>
                <div class="tags">
                  <span v-for="genre in selectedAnime.genres" :key="genre.mal_id" class="tag">{{ genre.name }}</span>
                </div>
              </div>
              <div class="section" v-if="selectedAnime.studios && selectedAnime.studios.length">
                <h3>Studios</h3>
                <div class="tags">
                  <span v-for="studio in selectedAnime.studios" :key="studio.mal_id" class="tag tag-blue">{{ studio.name }}</span>
                </div>
              </div>
              <a :href="selectedAnime.url" target="_blank" class="mal-link">
                View on MyAnimeList
                <svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M18 13v6a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2V8a2 2 0 0 1 2-2h6"></path><polyline points="15 3 21 3 21 9"></polyline><line x1="10" y1="14" x2="21" y2="3"></line></svg>
              </a>
            </div>
          </div>
        </div>
      </div>
    </transition>
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
      searched: false,
      showDetail: false,
      selectedAnime: {}
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
    },
    openDetail(anime) {
      this.selectedAnime = anime;
      this.showDetail = true;
    },
    closeDetail() {
      this.showDetail = false;
      this.selectedAnime = {};
    }
  }
};
</script>

<style scoped>
.search-container {
  max-width: 1200px;
  margin: 0 auto;
  padding: 3rem 1.5rem;
}

.search-header {
  text-align: center;
  margin-bottom: 3rem;
}

.title {
  font-size: 3rem;
  font-weight: 800;
  color: white;
  margin-bottom: 0.5rem;
  text-shadow: 0 2px 10px rgba(0, 0, 0, 0.2);
}

.subtitle {
  font-size: 1.125rem;
  color: rgba(255, 255, 255, 0.9);
  margin-bottom: 2rem;
}

.search-box {
  display: flex;
  gap: 0.75rem;
  max-width: 600px;
  margin: 0 auto;
}

.search-input {
  flex: 1;
  padding: 1rem 1.5rem;
  border: none;
  border-radius: 12px;
  font-size: 1rem;
  background: white;
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.1);
  transition: all 0.3s ease;
}

.search-input:focus {
  outline: none;
  box-shadow: 0 6px 30px rgba(0, 0, 0, 0.15);
  transform: translateY(-2px);
}

.search-btn {
  padding: 1rem 2rem;
  background: white;
  color: #667eea;
  border: none;
  border-radius: 12px;
  font-weight: 600;
  font-size: 1rem;
  cursor: pointer;
  display: flex;
  align-items: center;
  gap: 0.5rem;
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.1);
  transition: all 0.3s ease;
}

.search-btn:hover {
  background: #667eea;
  color: white;
  transform: translateY(-2px);
  box-shadow: 0 6px 30px rgba(102, 126, 234, 0.4);
}

.loader {
  text-align: center;
  padding: 4rem 0;
  color: white;
}

.spinner {
  width: 48px;
  height: 48px;
  border: 4px solid rgba(255, 255, 255, 0.2);
  border-top-color: white;
  border-radius: 50%;
  animation: spin 0.8s linear infinite;
  margin: 0 auto 1rem;
}

@keyframes spin {
  to { transform: rotate(360deg); }
}

.error-message {
  background: rgba(239, 68, 68, 0.1);
  border: 1px solid rgba(239, 68, 68, 0.3);
  color: #fee;
  padding: 1rem;
  border-radius: 12px;
  text-align: center;
  margin-bottom: 2rem;
}

.results {
  animation: fadeIn 0.5s ease;
}

@keyframes fadeIn {
  from { opacity: 0; transform: translateY(20px); }
  to { opacity: 1; transform: translateY(0); }
}

.anime-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(200px, 1fr));
  gap: 1.5rem;
}

.anime-card {
  background: white;
  border-radius: 16px;
  overflow: hidden;
  cursor: pointer;
  transition: all 0.3s ease;
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.1);
}

.anime-card:hover {
  transform: translateY(-8px);
  box-shadow: 0 12px 40px rgba(0, 0, 0, 0.2);
}

.card-image {
  position: relative;
  width: 100%;
  padding-top: 140%;
  overflow: hidden;
  background: #f3f4f6;
}

.card-image img {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  object-fit: cover;
  transition: transform 0.3s ease;
}

.anime-card:hover .card-image img {
  transform: scale(1.1);
}

.card-overlay {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: linear-gradient(to top, rgba(0,0,0,0.7) 0%, transparent 50%);
  display: flex;
  align-items: flex-end;
  padding: 1rem;
  opacity: 0;
  transition: opacity 0.3s ease;
}

.anime-card:hover .card-overlay {
  opacity: 1;
}

.score {
  display: flex;
  align-items: center;
  gap: 0.25rem;
  background: rgba(255, 255, 255, 0.95);
  padding: 0.25rem 0.75rem;
  border-radius: 8px;
  font-weight: 600;
  font-size: 0.875rem;
  color: #f59e0b;
}

.card-content {
  padding: 1rem;
}

.card-title {
  font-size: 0.9rem;
  font-weight: 600;
  color: #1f2937;
  margin-bottom: 0.5rem;
  display: -webkit-box;
  -webkit-line-clamp: 2;
  -webkit-box-orient: vertical;
  overflow: hidden;
  min-height: 2.6rem;
}

.card-meta {
  display: flex;
  align-items: center;
  gap: 0.5rem;
  flex-wrap: wrap;
}

.badge {
  background: #f3f4f6;
  color: #6b7280;
  padding: 0.25rem 0.625rem;
  border-radius: 6px;
  font-size: 0.75rem;
  font-weight: 500;
}

.episodes {
  color: #9ca3af;
  font-size: 0.75rem;
}

.no-results {
  text-align: center;
  padding: 4rem 0;
  color: white;
}

.no-results svg {
  opacity: 0.5;
  margin-bottom: 1rem;
}

.no-results p {
  font-size: 1.125rem;
  opacity: 0.9;
}

/* Modal */
.modal-backdrop {
  position: fixed;
  inset: 0;
  background: rgba(0, 0, 0, 0.7);
  backdrop-filter: blur(4px);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 1000;
  padding: 1rem;
}

.modal {
  background: white;
  border-radius: 24px;
  max-width: 700px;
  width: 100%;
  max-height: 90vh;
  overflow-y: auto;
  position: relative;
  box-shadow: 0 20px 60px rgba(0, 0, 0, 0.3);
}

.close-btn {
  position: absolute;
  top: 1rem;
  right: 1rem;
  background: rgba(255, 255, 255, 0.9);
  border: none;
  width: 40px;
  height: 40px;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  z-index: 10;
  transition: all 0.3s ease;
  color: #4b5563;
}

.close-btn:hover {
  background: white;
  transform: rotate(90deg);
  color: #1f2937;
}

.modal-content {
  padding: 2rem;
}

.modal-header {
  display: flex;
  gap: 1.5rem;
  margin-bottom: 2rem;
}

.modal-image {
  width: 180px;
  height: 250px;
  object-fit: cover;
  border-radius: 12px;
  box-shadow: 0 8px 30px rgba(0, 0, 0, 0.2);
}

.modal-info {
  flex: 1;
}

.modal-title {
  font-size: 1.75rem;
  font-weight: 700;
  color: #1f2937;
  margin-bottom: 1rem;
}

.modal-badges {
  display: flex;
  gap: 0.5rem;
  flex-wrap: wrap;
  margin-bottom: 1rem;
}

.modal-stats {
  display: flex;
  gap: 1.5rem;
  flex-wrap: wrap;
}

.stat {
  display: flex;
  align-items: center;
  gap: 0.5rem;
  color: #6b7280;
  font-size: 0.9rem;
}

.stat svg {
  color: #f59e0b;
}

.modal-body {
  display: flex;
  flex-direction: column;
  gap: 1.5rem;
}

.section h3 {
  font-size: 1rem;
  font-weight: 600;
  color: #374151;
  margin-bottom: 0.75rem;
}

.section p {
  color: #6b7280;
  line-height: 1.7;
  font-size: 0.9rem;
}

.tags {
  display: flex;
  gap: 0.5rem;
  flex-wrap: wrap;
}

.tag {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: white;
  padding: 0.375rem 0.875rem;
  border-radius: 8px;
  font-size: 0.8rem;
  font-weight: 500;
}

.tag-blue {
  background: linear-gradient(135deg, #3b82f6 0%, #2563eb 100%);
}

.mal-link {
  display: inline-flex;
  align-items: center;
  gap: 0.5rem;
  color: #667eea;
  text-decoration: none;
  font-weight: 600;
  transition: all 0.3s ease;
  font-size: 0.9rem;
}

.mal-link:hover {
  gap: 0.75rem;
  color: #764ba2;
}

/* Modal Transitions */
.modal-enter-active, .modal-leave-active {
  transition: opacity 0.3s ease;
}

.modal-enter-active .modal,
.modal-leave-active .modal {
  transition: transform 0.3s ease;
}

.modal-enter-from, .modal-leave-to {
  opacity: 0;
}

.modal-enter-from .modal,
.modal-leave-to .modal {
  transform: scale(0.9) translateY(20px);
}

@media (max-width: 768px) {
  .title {
    font-size: 2rem;
  }
  
  .search-box {
    flex-direction: column;
  }
  
  .anime-grid {
    grid-template-columns: repeat(auto-fill, minmax(150px, 1fr));
    gap: 1rem;
  }
  
  .modal-header {
    flex-direction: column;
    align-items: center;
    text-align: center;
  }
  
  .modal-image {
    width: 150px;
    height: 210px;
  }
}
</style>
