<script setup>
import { ref, computed, onMounted } from "vue";

const query = ref("");
const my_anime = ref([]);
const search_results = ref([]);

const my_anime_asc = computed(() => {
  return my_anime.value.sort((a, b) => {
    return a.title.localeCompare(b.title);
  });
});

const searchAnime = async () => {
  const url = `https://api.jikan.moe/v4/anime?q=${query.value}`;
  await fetch(url)
    .then((res) => res.json())
    .then((res) => {
      search_results.value = res.data;
    });
};

const handleInput = (e) => {
  if (!e.target.value) {
    search_results.value = [];
  }
};

const addAnime = (anime) => {
  search_results.value = [];
  query.value = "";

  my_anime.value.push({
    id: anime.mal_id,
    title: anime.title,
    image: anime.images.jpg.image_url,
    total_episodes: anime.episodes,
    watched_episodes: 0,
  });

  localStorage.setItem("my_anime", JSON.stringify(my_anime.value));
};

const increaseWatch = (anime) => {
  anime.watched_episodes++;
  localStorage.setItem("my_anime", JSON.stringify(my_anime.value));
};

const decreaseWatch = (anime) => {
  anime.watched_episodes--;
  localStorage.setItem("my_anime", JSON.stringify(my_anime.value));
};

onMounted(() => {
  my_anime.value = JSON.parse(localStorage.getItem("my_anime")) || [];
});
</script>

<template>
  <main>
    <h1>Anime Tracker</h1>
    <form @submit.prevent="searchAnime">
      <input
        type="text"
        placeholder="Search Anime"
        v-model="query"
        @input="handleInput"
      />
      <button type="submit">Search</button>
    </form>

    <div class="results" v-if="search_results.length > 0">
      <div class="result" v-for="anime in search_results">
        <img :src="anime.images.jpg.image_url" />
        <div class="details">
          <h3>{{ anime.title }}</h3>
          <p :title="anime.synopsis" v-if="anime.synopsis">
            {{ anime.synopsis.slice(0, 120) }} ...
          </p>
          <span class="flex-1"></span>
          <button @click="addAnime(anime)">Add to my anime</button>
        </div>
      </div>
    </div>

    <div class="myanime" v-if="my_anime.length > 0">
      <h2>My anime</h2>
      <div v-for="anime in my_anime_asc" class="anime">
        <img :src="anime.image" alt="image" />
        <h3>{{ anime.title }}</h3>
        <div class="flex-1">
          <span class="episodes">
            {{ anime.watched_episodes }} / {{ anime.total_episodes }}
          </span>
          <button
            class="btn"
            v-if="anime.total_episodes !== anime.watched_episodes"
            @click="increaseWatch(anime)"
          >
            +
          </button>
          <button
            class="btn"
            v-if="anime.watched_episodes > 0"
            @click="decreaseWatch(anime)"
          >
            -
          </button>
        </div>
      </div>
    </div>
  </main>
</template>

<style scoped>
input {
  padding: 0.5rem 1rem;
  border: none;
  font-size: 1.2rem;
  border-radius: 10px;
}

.myanime h2 {
  color: #363636;
  font-family: Arial, Helvetica, sans-serif;
  font-weight: bolder;
}

.myanime,
.results {
  display: flex;
  flex-direction: column;
  background-color: #919098;
  margin-top: 2rem;
  width: 100%;
  border-radius: 15px;
}

.results .result {
  display: flex;
  justify-content: flex-start;
  margin: 1rem auto;
  align-items: center;
  width: 80%;
  padding: 1.2rem;
  border-radius: 15px;
  background-color: #363636;
}

.details {
  width: 70%;
  margin: 0 auto;
}

.myanime .anime {
  display: flex;
  justify-content: space-between;
  flex-wrap: wrap;
  margin: 1rem auto;
  align-items: center;
  width: 80%;
  padding: 1.2rem;
  border-radius: 15px;
  background-color: #363636;
}

.myanime .anime img,
.results .result img {
  width: 150px;
  object-fit: contain;
  border-radius: 20px;
}

.myanime .anime h3 {
  width: 40%;
}

.myanime .flex-1 {
  width: 25%;
  display: flex;
  flex-direction: row;
  justify-content: flex-end;
}

.flex-1 span {
  margin-right: 15px;
}

.flex-1 button {
  padding: 5px 10px;
  margin-right: 15px;
}

@media screen and (max-width: 768px) {
  .myanime .anime,
  .results .result {
    flex-direction: column;
    padding: 1rem;
  }

  .myanime .anime h3 {
    width: 100%;
  }

  .myanime .flex-1 {
    width: 100%;
    justify-content: center;
  }
}
</style>
