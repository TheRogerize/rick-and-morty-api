<template>
  <div class="modal-card" id="modal-card">
    <b-loading
      :is-full-page="false"
      v-model="isLoading"
      :can-cancel="false"
    ></b-loading>
    <div class="character-modal" v-if="character">
      <div class="modal-header">
        <div class="close" @click="$emit('close')">
          <b-icon pack="mdi" icon="close-circle" size="is-medium"> </b-icon>
        </div>
      </div>
      <div class="modal-character-header">
        <div class="avatar-container">
          <div
            class="avatar"
            :style="{ backgroundImage: 'url(' + character.image + ')' }"
          ></div>
        </div>
        <div class="character-info">
          <small>{{ character.status }}</small>
          <p>{{ character.name }}</p>
          <small>{{ character.species }}</small>
        </div>
        <div class="character-content">
          <h3>Información</h3>
          <div class="columns is-mobile">
            <div class="column is-one-third">
              <div class="data-card">
                <small>
                  <b-tooltip position="is-right" multilined>
                    <img src="@/assets/info.png" width="12px" height="12px" />
                    <template v-slot:content>
                      <b
                        >The gender of the character ('Female', 'Male',
                        'Genderless' or 'unknown')</b
                      >.
                    </template>
                  </b-tooltip>
                  Gender:
                </small>
                <p>{{ character.gender }}</p>
              </div>
            </div>
            <div class="column is-one-third">
              <div class="data-card">
                <small>
                  <b-tooltip position="is-top" multilined>
                    <img src="@/assets/info.png" width="12px" height="12px" />
                    <template v-slot:content>
                      <b>Name and link to the character's origin location</b>.
                    </template>
                  </b-tooltip>
                  Origin:
                </small>
                <p>{{ character.origin.name }}</p>
              </div>
            </div>
            <div class="column is-one-third" v-if="character.type.length">
              <div class="data-card">
                <small>
                  <b-tooltip position="is-top" multilined>
                    <img src="@/assets/info.png" width="12px" height="12px" />
                    <template v-slot:content>
                      <b>The type or subspecies of the character</b>.
                    </template>
                  </b-tooltip>
                  Type:
                </small>

                <p>{{ character.type }}</p>
              </div>
            </div>
          </div>

          <hr class="is-divider" />
          <span v-if="appearances.length">
            <h3>Episodios</h3>
            <div class="columns is-multiline is-mobile mb-3">
              <div
                class="
                  column
                  is-one-quarter-desktop is-one-third-tablet is-half-mobile
                "
                v-for="(appearance, index) in appearances"
                :key="index"
              >
                <div class="data-card">
                  <small>{{ appearance.name }}</small>
                  <p>{{ appearance.episode }}</p>
                  <small class="is-uppercase">{{ appearance.air_date }}</small>
                </div>
              </div>
            </div>
          </span>

          <hr class="is-divider" />
          <span v-if="randomCharacters.length">
            <h3>Personajes interesantes</h3>
            <div
              class="
                container
                character-cards character-cards-just-two
                is-mobile
              "
            >
              <div
                class="character-card"
                v-for="(randomCharacter, index) in randomCharacters"
                :key="index"
                @click="selectCharacter(randomCharacter.url)"
              >
                <div
                  class="image"
                  :style="{
                    backgroundImage: 'url(' + randomCharacter.image + ')',
                  }"
                >
                  <div class="is-favorite">
                    <b-button rounded class="favorites-btn" size="is-small">
                      <b-icon pack="mdi" icon="star" size="is-small"> </b-icon>
                    </b-button>
                  </div>
                </div>
                <div class="character-info">
                  <div class="status">
                    <b-icon
                      :type="{
                        'is-success': randomCharacter.status === 'Alive',
                        'is-danger': randomCharacter.status !== 'Alive',
                      }"
                      pack="mdi"
                      icon="circle"
                      size="is-small"
                    >
                    </b-icon>
                    {{ randomCharacter.status }} - {{ randomCharacter.species }}
                  </div>
                  <div class="character-name">
                    {{ randomCharacter.name }}
                  </div>
                  <small class="mt-1">Last known location</small>
                  <p>{{ randomCharacter.location.name }}</p>
                  <small class="mt-1">First seen in</small>
                  <p>{{ randomCharacter.episode.name }}</p>
                </div>
              </div>
            </div>
          </span>
          <div class="share-container">
            <b-button class="share-btn" rounded>Compartir personaje</b-button>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import axios from "axios";

export default {
  name: "CharacterModal",
  props: {
    url: {
      type: String,
      required: false,
    },
    episodes: {
      type: Array,
      required: true,
    },
  },
  data() {
    return {
      isLoading: true,
      character: null,
      appearances: [],
      randomCharacters: [],
      config: {
        headers: {
          "Access-Control-Allow-Origin": "*",
          "Content-type": "application/json",
        },
      },
    };
  },
  mounted() {
    this.getCharacter(this.url);
  },
  methods: {
    async getCharacter(url) {
      this.isLoading = true;
      axios
        .get(url, this.config)
        .then((res) => {
          this.character = res.data;
          this.getRandomCharacters();
          this.filterEpisodes("character");
        })
        .catch(() => {
          this.errorToast(
            "Ha ocurrido un error al buscar la información del personaje seleccionado☹️"
          );
          this.isLoading = false;
        });
    },

    async selectCharacter(url) {
      this.getCharacter(url);
    },

    async getRandomCharacters() {
      let randomIndex = null;
      let random = [];
      for (let i = 0; i < 3; i++) {
        randomIndex = Math.floor(Math.random() * (100 - 1 + 1) + 1);
        random.push(randomIndex);
      }

      axios.defaults.headers.common["Access-Control-Allow-Origin"] = "*";
      axios
        .get(
          `https://rickandmortyapi.com/api/character/${random[0]},${random[1]},${random[2]}`,
          this.config
        )
        .then((res) => {
          this.randomCharacters = res.data;
          this.filterEpisodes("random");
          this.isLoading = false;
        })
        .catch(() => {
          this.errorToast(
            "Ha ocurrido un error al buscar personajes de interés, por favor intenta de nuevo."
          );
        });
    },

    filterEpisodes(type) {
      switch (type) {
        case "character":
          this.appearances = this.episodes.filter((episode) => {
            return episode.characters.includes(this.character.url);
          });
          break;
        case "random":
          this.randomCharacters.forEach((character, index) => {
            this.randomCharacters[index].episode = this.episodes.find(
              (episode) => {
                return episode.url === character.episode[0];
              }
            );
          });
          break;
      }
    },

    errorToast(message) {
      this.$buefy.toast.open({
        message,
        duration: 5000,
        type: "is-danger",
        position: "is-bottom",
        pauseOnHover: true,
      });
    },
  },
};
</script>
