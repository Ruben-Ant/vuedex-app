<template>
  <v-app id="inspire">
    <v-app-bar
        color="orange accent-1"
        app
    >
      <v-app-bar-nav-icon>
        <v-icon
          large
          color="red">
          mdi-pokeball

        </v-icon>
      </v-app-bar-nav-icon>
      <v-app-bar-title class="text-h6 mr-6 hidden-sm-and-down">
        VuéDex
      </v-app-bar-title>

      <v-autocomplete
          v-model="selectedPokemons"
          :items="items"
          :loading="isLoading"
          :search-input.sync="search"
          multiple
          item-value="url"
          deletable-chips
          clearable
          hide-details
          hide-selected
          item-text="name"
          label="Search for a poke..."
          solo
          @change="getPokemon"

      >
        <template v-slot:no-data>
          <v-list-item>
            <v-list-item-title>
              Search for your favorite
              <strong>Pokemons</strong>
            </v-list-item-title>
          </v-list-item>
        </template>
        <template v-slot:selection="{ attr, on, item, selected }">
          <v-chip
              v-bind="attr"
              :input-value="selected"
              color="blue-grey"
              class="white--text"
              v-on="on"
          >
            <v-icon left>
              mdi-pokeball
            </v-icon>
            <span v-text="item.name"></span>
          </v-chip>
        </template>
        <template v-slot:item="{ item }">
          <v-list-item-avatar
              color="grey"
              class="text-h5 font-weight-light white--text"
          >
            {{ item.name.charAt(0).toUpperCase() }}
          </v-list-item-avatar>
          <v-list-item-content>
            <v-list-item-title v-text="item.name"></v-list-item-title>
          </v-list-item-content>
          <v-list-item-action>
            <v-icon>mdi-pokemon-go</v-icon>
          </v-list-item-action>
        </template>
      </v-autocomplete>

    </v-app-bar>

    <v-main>
      <v-container>
        <v-row>

        </v-row>
        <v-row>
          <v-col
              v-for="pokemon in selectedPokemonsData"
              :key="pokemon"
              cols="4"
          >
            <v-card
                outlined
                elevation="3"
            >
              <v-img
                  height="250"
                  width="250"
                  :src='pokemon["sprites"]["other"]["official-artwork"]["front_default"]' >
              </v-img>

              <v-card-title> {{pokemon.name}}</v-card-title>

              <v-card-text>
                <v-row
                    align="center"
                    class="mx-0"
                >
                  <v-rating
                      :value="pokemon.base_experience*5/300"
                      color="amber"
                      background-color="grey lighten-2"
                      dense
                      half-increments
                      readonly
                      size="20"
                  ></v-rating>

                  <div class="grey--text ms-4">
                   Experiance gained <strong> {{ pokemon.base_experience }} </strong>
                  </div>
                </v-row>
              </v-card-text>

              <v-divider class="mx-4"></v-divider>

              <v-card-title>Abilities</v-card-title>
              <v-card-text>
                <v-chip-group
                >
                  <v-chip
                      v-for="ability in pokemon.abilities"
                      :key="ability"
                      class="ma-2"
                      color="indigo darken-3"
                      outlined
                      label>

                    <v-icon left>
                      mdi-label
                    </v-icon>
                    {{ ability.ability.name }}

                  </v-chip>
                </v-chip-group>
              </v-card-text>

              <v-card-title>Pokemon Type</v-card-title>
              <v-card-text>
                <v-chip-group
                >
                  <v-chip
                      v-for="type in pokemon.types"
                      :key="type"
                      :color=" colors[type.type.name] ">{{ type.type.name }}

                  </v-chip>
                </v-chip-group>
              </v-card-text>

              <v-divider class="mx-4"></v-divider>

              <v-card-title>Pokemon Stats</v-card-title>
              <v-card-text>
                  <v-container
                      v-for="stat in pokemon.stats"
                      :key="stat">
                        {{ stat.stat.name }}
                        <v-progress-linear
                            :color=" colors[stat.stat.name]"
                            height="10"
                            :value="stat.base_stat"
                        >
                        </v-progress-linear>
                  </v-container>
              </v-card-text>
            </v-card>
          </v-col>
        </v-row>
      </v-container>
    </v-main>
  </v-app>

</template>


<script>
export default {
  data: () => ({
    pokemon:[],
    isLoading: false,
    items: [],
    item: null,
    selectedPokemons: [],
    selectedPokemonsData: [],

    search: null,
    tab: null,

    //theme fais en deux deux
    colors :{
      /*TYPE*/
      normal: '#A8A77A',
      fire: '#EE8130',
      water: '#6390F0',
      electric: '#F7D02C',
      grass: '#7AC74C',
      ice: '#96D9D6',
      fighting: '#C22E28',
      poison: '#A33EA1',
      ground: '#E2BF65',
      flying: '#A98FF3',
      psychic: '#F95587',
      bug: '#A6B91A',
      rock: '#B6A136',
      ghost: '#735797',
      dragon: '#6F35FC',
      dark: '#705746',
      steel: '#B7B7CE',
      fairy: '#D685AD',

      /*STATS*/
      hp:'#42cb8a',
      attack:'#cb424d',
      defense:'#39d8e0',
      speed:'#67736f',
    },
  }),

  methods:{
    getPokemons: function() {
      // Lazy input system
      fetch('https://pokeapi.co/api/v2/pokemon/?limit=1200')
          .then(res => res.clone().json())
          .then(res => {
            this.items = res.results
          })
          .catch(err => {
            console.log(err)
          })
          .finally(() => (this.isLoading = false))},

    getPokemon: function (pokemonsUrl) {
      this.selectedPokemonsData = []
      pokemonsUrl.forEach(url => {
        fetch(url)
            .then(res => res.clone().json())
            .then(res => {

              //optimisable, checker si l'item existe et ne pas tout demadner a l'API
              this.selectedPokemonsData.push(res)
              this.selectedPokemonsData.reverse()
            })
            .catch(err => {
              console.log(err)
            })
      });
      }
  },
  mounted(){
    //on demande une seul fois toute la liste
    this.getPokemons()
  },

  watch: {
    selectedPokemons (val) {
      if (val != null) this.tab = 0
      else this.tab = null


    },
    search () {
      // Items have already been loaded
      if (this.items.length > 0) return
      this.isLoading = true
    },
  },
}
</script>
