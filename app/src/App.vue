<template>
  <v-app>
    <v-container class="fill-height" style="padding: 0; max-width: 450px">
      <v-row class="fill-height flex-column">
        <FlashCards class="flex-grow-1" />
        <ActionBar class="flex-grow-0" @deckSelected="changeDeck(deckName)"/>
      </v-row>
    </v-container>
    <ConfirmDownload />
    <DownloadProgress />
  </v-app>
</template>

<script>
import ActionBar from './components/ActionBar'
import FlashCards from './components/FlashCards'
import ConfirmDownload from './components/ConfirmDownload'
import DownloadProgress from './components/DownloadProgress'

import { mapState } from 'vuex'

export default {
  name: 'App',

  components: {
    ActionBar,
    FlashCards,
    ConfirmDownload,
    DownloadProgress
  },

  computed: mapState(['isApp', 'darkTheme']),

  watch: {
    darkTheme () {
      this.$vuetify.theme.dark = this.darkTheme
    }
  },

  async mounted () {
    if (this.isApp) {
      document.addEventListener('deviceReady', this.init)
    } else {
      this.init()
    }
  },

  methods: {
    async init () {
      this.$store.commit('setDownloadProgress', -1)
      this.$store.commit('setProposedDeck', {})

      await this.$store.dispatch('getDeckList')

      if (!this.$store.state.deck.name) {
        const deckList = this.$store.state.deckList
        for (let idx=0; idx<5; idx++) {
          const deck = deckList[Object.keys(deckList)[idx]]
          if (deck.type === 'gifs') continue
          await this.$store.dispatch('deckChange', deck)
          break
        }
      } else {
        this.$store.commit('initDeckDownloaded')
        await this.$store.dispatch('deckChange', this.$store.state.deck)
      }
    }
  }

};
</script>
