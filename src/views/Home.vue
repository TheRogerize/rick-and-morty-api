<template>
  <div class="home">
    <b-loading :is-full-page="true" v-model="isLoading" :can-cancel="true"></b-loading>
    <div class="header">
      <div class="bg header-portal"></div>
      <div class="bg overlay"></div>
      <div class="header-search">
        <img src="@/assets/rickandmorty.png" width="300">
         <b-field>
            <b-input custom-class="search-bar" 
            placeholder="Buscar personaje..."
            v-model="searchTerm"
            type="search"
            icon="magnify"
            >
          </b-input>
        </b-field>
      </div>
    </div>
    <div class="content">
      <b-tabs size="is-medium is-centered" expanded v-model="genderFilter" class="block">
        <b-tab-item v-for="(tab, index) in genderFilters"
        :key="index"
        :name="tab.value"
        :value="tab.value"
        :label="tab.label"
        :icon="tab.icon">
        </b-tab-item>
      </b-tabs>
      <b-tabs size="is-medium is-centered" expanded v-model="statusFilter" class="block">
        <b-tab-item v-for="(status, index) in statusFilters"
        :key="index"
        :name="status.value"
        :value="status.value"
        :label="status.label"
        :icon="status.icon">
        </b-tab-item>
      </b-tabs>

    <div class="container is-max-desktop not-found" v-if="error">
      <div class="columns">
        <div class="column column-text">
          <h3>¡Uh-oh!</h3>
          <p>¡Pareces perdido en tu viaje!</p>
          <b-button rounded type="remove-filters" @click="removeFilters" >Eliminar filtros</b-button>
        </div>
      </div>
    </div>

    <div class="container is-small" v-else>
      <div class="favorites-tile">
        Mostrar favoritos
             <b-button 
              rounded
              class="favorites-btn"
              size="is-small"
              :class="{ 'active': favorites }"
              @click="favorites = !favorites">
               <b-icon
                pack="mdi"
                icon="star"
                size="is-small">
                
              </b-icon>
             </b-button>
        </div>
      </div>

      <div class="container is-medium character-cards is-variable is-2 my-5">
          <div 
          class="character-card" 
          v-for="(character, index) in characters"
          :key="index"
          @click="selectCharacter(character.url)">
          <div class="image" :style="{ backgroundImage: 'url('+ character.image +')' }">
            <div class="is-favorite">
              <b-button 
                rounded
                class="favorites-btn"
                size="is-small"
                :class="{ 'active': favorites }">
                 <b-icon
                  pack="mdi"
                  icon="star"
                  size="is-small">
                </b-icon>
               </b-button>
              </div>
            </div>
            <div class="character-info">
              <div class="status">
                <b-icon
                :type="{'is-success': character.status === 'Alive', 'is-danger': character.status !== 'Alive'}"
                pack="mdi"
                icon="circle"
                size="is-small">
                </b-icon>
                {{ character.status }} - {{ character.species }}
              </div>
              <div class="character-name">
                {{ character.name }}
              </div>
              <small class="mt-1">Last known location</small>
              <p>{{ character.location.name }}</p>
              <small class="mt-1">First seen in</small>
              <p>{{ character.episode[0] | firstSeen(episodes) }}</p>
            </div>
        </div>
      </div>
    </div>
    <hr>
    <div class="pagination" v-if="characters.length">
      <b-pagination
          class="contain"
          v-model="currentPage"
          :total="total"
          :range-before="rangeBefore"
          :range-after="rangeAfter"
          :order="order"
          :size="size"
          :simple="isSimple"
          :rounded="isRounded"
          :per-page="perPage"
          :icon-prev="prevIcon"
          :icon-next="nextIcon"
          aria-next-label="Next page"
          aria-previous-label="Previous page"
          aria-page-label="Page"
          aria-current-label="Current page">
      </b-pagination>
    </div>
  </div>
</template>

<script>
import CharacterModal from "@/components/CharacterModal.vue";
import axios from "axios";

export default {
  name: "Home",
  data() {
    return {
      config: {
        headers: {
        'Access-Control-Allow-Origin': '*',
        'Content-type': 'application/json',
        }
      },
      characterURL: "https://rickandmortyapi.com/api/character",
      episodeURL: "https://rickandmortyapi.com/api/episode?page=",
      requestsURL: [
        this.characterURL,
        this.episodeURL
      ],
      characters: [],
      episodes: [],
      singleCharacter: null,
      pages: 0,
      total: 0,
      current: 1,
      perPage: 20,
      rangeBefore: 3,
      rangeAfter: 1,
      order: 'is-centered',
      size: '',
      isSimple: false,
      isRounded: false,
      prevIcon: 'chevron-left',
      nextIcon: 'chevron-right',
      index: 0,
      prev: "",
      next: "",
      error: false,
      isLoading: true,
      favorites: false,
      statusFilters: [
        {label: "All", value: "", icon:""},
        {label: "Alive", value: "alive", icon:"person"},
        {label: "Dead", value: "dead", icon:"sentiment_very_dissatisfied"},
        {label: "Unknown", value: "unknown", icon:"help"}
      ],
      genderFilters: [
        {label: "All", value: "", icon:""},
        {label: "Unknown", value: "unknown", icon:"gender-male-female-variant"},
        {label: "Female", value: "female", icon:"gender-female"},
        {label: "Male", value: "male", icon:"gender-male"},
        {label: "Genderless", value: "genderless", icon:"account-off"}
      ],
    }
  },
  components: {
  },
  computed: {
    currentPage: {
      set(page) {
        let query = {...this.$route.query};
        query.page = page;
        if(!page) {
          query.page = 1;
        } else {
          query.page = page;
        }
        this.$router.replace({ query: query });
        this.getCharacters();
      },
      get() {
        if(!this.$route.query.page) {
          let query = {...this.$route.query};
          query.page = 1;
          this.$router.replace({ query: query });
        } 
        return this.$route.query.page ? this.$route.query.page : 1;
      }
    },
     genderFilter: {
      set(gender) {
        let query = {...this.$route.query};
        query.gender = gender;
        if(!gender.length) {
          delete query.gender;
        } else {
          query.gender = gender;
        }
        query.page = 1;
        this.$router.replace({ query: query });
        this.getCharacters();
      },
      get() {
          return this.$route.query.gender ? this.$route.query.gender : '';
      }
    },

     statusFilter: {
      set(status) {
        let query = {...this.$route.query};
        query.status = status;
        if(!status.length) {
          delete query.status;
        } else {
          query.status = status;
        }
        query.page = 1;
        this.$router.replace({ query: query });
        this.getCharacters();
      },
      get() {
        return this.$route.query.status ? this.$route.query.status : '';
      }
    },

    searchTerm: {
      set(name) {
          let query = Object.assign({}, this.$route.query);
          if(!name.length) {
            delete query.name;
          } else {
            query.name = name;
          }
          query.page = 1;
          this.$router.replace({query: query });
          this.getCharacters();
      },
      get() {
        return  this.$route.query.name ? this.$route.query.name : "";
      }
    }
  },
  mounted () {
    this.getCharacters();
  },
  filters: {
    firstSeen: function (episodeURL, episodes) {
      if (!episodeURL) return "";
      const firstEpisode = episodes.find((episode) => {
        return episode.url === episodeURL;
      }); 
      return firstEpisode ? firstEpisode.name : "";
    }
  },

  methods: {
    setCharacters(res) {
      if(res.status === 200) {
        this.characters = res.data.results;
        this.pages = res.data.info.pages;
        this.prev = res.data.info.prev;
        this.next = res.data.info.next;
        this.total = res.data.info.count;
        this.error = false;
        if(!this.episodes.length) this.getEpisodes();
      } else {
        this.error = true;
        this.errorToast("No se han encontrado personajes ☹️")
      }
      this.isLoading = false;
    },

    async getEpisodes() {
      let episodePromise = null;
      let episodeRequests = [];
      let episodes = null;
      for(let i=1; i <=3; i++) {
        episodePromise = axios.get(this.episodeURL+i);
        episodeRequests.push(episodePromise);
      }
      
      axios.all(episodeRequests).then(axios.spread((...responses) => {
        responses.forEach(page => {
          episodes = page.data.results;
          this.episodes.push(...episodes);
        });
      }))
      .catch(() => {
        this.error = true;
        this.errorToast("Ha ocurrido un problema con el servidor☹️")
      });
    },

    async getCharacters() {
      let url = this.characterURL;
      axios.defaults.headers.common['Access-Control-Allow-Origin'] = '*';
      let concat = "?";
      if(this.searchTerm && this.searchTerm.length) {
        url += `${concat}name=${this.searchTerm}`; 
        concat = "&";
      }
      if(this.genderFilter && this.genderFilter.length) {
        url += `${concat}gender=${this.genderFilter}`; 
        concat = "&";
      }
      if(this.statusFilter && this.statusFilter.length) {
        url += `${concat}status=${this.statusFilter}`; 
        concat = "&";
      }
      url += `${concat}page=${this.currentPage}`; 
      return axios.get(url, this.config)
        .then(res => {
          this.setCharacters(res);
        })
        .catch(() => {
          this.error = true;
          this.errorToast("No se han encontrado personajes ☹️")
          this.isLoading = false;
      });
      },

     async selectCharacter(url) {
      this.$buefy.modal.open({
        active: true,
        parent: this,
        component: CharacterModal,
        hasModalCard: true,
        props: { url, episodes: this.episodes },
        trapFocus: false
      })
    },

    removeFilters() {
      let query = Object.assign({}, this.$route.query);
      query.page = 1;
      delete query.name;
      delete query.gender;
      delete query.status;
      this.error = false;
      this.isLoading = true;
      this.$router.replace({query: query });
      this.getCharacters();
    },

    errorToast(message) {
      this.$buefy.toast.open({
        message,
        duration: 5000,
        type: 'is-danger',
        position: 'is-bottom',
        pauseOnHover: true
      })
    }
  }
};
</script>

