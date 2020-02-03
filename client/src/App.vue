<template>
  <div>
    <game-selection v-if="!gameOptionsSelected" :decks="decks" />
    <controller v-if="gameOptionsSelected" :cards="cards" :gameType="gameType" :deckDescriptions="deckDescriptions" />
  </div>
</template>

<script>
import GameSelection from './components/GameSelection.vue'
import CardDecksService from '../services/CardDecksService.js'
import Controller from "./components/Controller.vue"
import {eventBus} from "./main.js"
export default {
  name: "app",
  components: {
    "controller": Controller,
    "game-selection": GameSelection
  },
  data() {
    return {
      decks: [],
      cards: [],
      deckDescriptions: [],
      gameOptionsSelected: false,
      gameType: '',
      countries: [],
      countriesObject: {name: 'Countries',
                        deckImageURL: '',
                        deck: [],
                        descriptions: {}
                  }
    }
  },
  mounted(){
    this.fetchDecks()
    this.fetchCountries()

    // gets the game options and saves them so can be sent down as props
    eventBus.$on('game-options-selected', (payload) => {
      this.cards = payload.deck
      this.gameOptionsSelected = true
      this.gameType = payload.gameType
      this.deckDescriptions = payload.descriptions
    })

    //receives emit from main menu button and goes back to deck selection
    eventBus.$on('main-menu', () => {
      this.gameOptionsSelected = false
      this.cards = []
      this.deckDescriptions = []
      this.gameType = ''
      this.decks = []
      this.fetchDecks()
    })
  },
  methods: {
    //gets the decks from the DB
    fetchDecks() {
      CardDecksService.getDecks()
      .then(decks => this.decks = decks)
    },

    fetchCountries() {
      fetch("https://restcountries.eu/rest/v2/all")
      .then(response => response.json())
      .then(countries => {
        this.countries = countries.slice(0,30)
      } )
      .then(banana => this.putCountriesIntoDecks())
    },

    putCountriesIntoDecks() {
      this.countries.forEach((country, index) => {
        let object = {}
        object.name = country.name
        object.imageURL = country.flag
        let playableProps = {}
        playableProps.population = country.population
        playableProps.borders = country.borders.length
        playableProps.languages = country.languages.length
        object.playableProperties = playableProps
        this.countriesObject.descriptions = {population: 'Population', languages: 'Number of Languages', borders: 'Number of Borders'}
        this.countriesObject.deck[index] = object

      });
        this.decks.push(this.countriesObject)
    }
  }
};
</script>

<style>


</style>
