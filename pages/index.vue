<template>
  <section>
    <div class="container mt-5">
      <div class="row">
        <div class="col-md-12">
          <strong><h5>xCrucibleDataExporter Test</h5></strong>
        </div>
      </div>
      <div class="row">
        <div class="col-md-3">
          <input type="text" class="form-control w-100" v-model="handle" placeholder="Username">
          <select class="form-control mt-2" v-model="membershipType">
            <option selected disabled>Platform</option>
            <option value="1">XBox</option>
            <option value="2">PS4</option>
            <option value="4">PC</option>
          </select>
          <button :disabled="membershipType === 'Platform'" class="btn btn-primary mt-2 w-100" @click="getIDByUsername()">Get Characters</button>
        </div>
        <div class="col-md-3">
          <select :disabled="memID === null" class="form-control" v-model="currChar">
            <option default disabled selected>Character</option>
            <option v-for="char in charIDs" default :value="char.characterId" :key="char.characterId">
              <span v-if="char.classType === 1">
                Hunter
              </span>
              <span v-if="char.classType === 0">
                Titan
              </span>
              <span v-if="char.classType === 2">
                Warlock
              </span>
              {{ char.light }} 
            </option>
          </select>
          <button :disabled="currChar === 'Character'" class="btn btn-light w-100 mt-2" @click="getHistoricalForChar()">Get PVP Historical Data</button> 
          <button :disabled="currChar === 'Character'" class="btn btn-light w-100 mt-2" @click="getActivityHistory()">Get PVP Game History</button> 
        </div>
      </div>
      <div class="row mt-3">
        <div class="col-md-12">
          Raw JSON <br>
          <textarea class="w-100" rows="10" v-model="crucibleData"></textarea>
          <!-- <button @click="convertToCSV()">Convert to CSV</button> -->
          <button class="btn btn-light mt-2">Copy to Clipboard</button>
        </div>
      </div>
    </div>
  </section>
</template>

<script>
import Logo from '~/components/Logo.vue'

export default {
  // get memID
  // get Characters
  // get history

  // Xbox = 1, PSN = 2, eventually Blizzard = 4)
  components: {
    Logo
  },
  data: function () {
    return {
      APIconfig: {
        headers: { 'X-API-Key': 'c078f480a5894d9486ed4c0a7366a0f9' }
      },
      crucibleData: '',
      membershipType: 'Platform',
      charIDs: [],
      currChar: 'Character',
      memID: null,
      handle: null,
      games: []
    }
  },
  methods: {
    searchForUser: function () {
      let query = 'gsxrclyde'
      let url = 'https://www.bungie.net/platform/User/SearchUsers/?q=' + query

      this.$axios(url, this.APIconfig).then(response => {
        console.log(response)
      })
    },
    getIDByUsername: function () {
      let query = this.handle.replace('#','%23')
      console.log(query)
      let url = 'https://www.bungie.net/platform/Destiny2/SearchDestinyPlayer/-1/' + query

      this.$axios(url, this.APIconfig).then(response => {
        this.memID = response.data.Response[0].membershipId

        console.log(response)

        this.getCharsByID()
      })

      // reset character select
      this.currChar = 'Character'
    },
    getCharsByID: function () {
      let query = this.handle
      let url = 'https://www.bungie.net/platform/Destiny2/' + this.membershipType + '/Profile/' + this.memID + '/?components=200'

      this.$axios(url, this.APIconfig).then(response => {
        let chars = response.data.Response.characters.data

        this.charIDs = []
        
        for (let char in chars) {
          this.charIDs.push(chars[char])
        }
      })
    },
    getHistoricalForChar: function () {
      let memID = this.memID
      let url = 'https://www.bungie.net/Platform/Destiny2/' + this.membershipType + '/Account/' + this.memID + '/Character/' + this.currChar + '/Stats/'
      
      // get overview of PvP history for character

      this.$axios(url, this.APIconfig).then(response => {
        console.log(response.data.Response.allPvP.allTime)
        this.crucibleData = JSON.stringify(response.data.Response.allPvP.allTime)
      })
    },
    getActivityHistory: function () {
      let memID = this.memID
      let url1 = 'https://www.bungie.net/Platform/Destiny2/' + this.membershipType + '/Account/' + this.memID + '/Character/' + this.currChar + '/Stats/'

      // get count of max games
      // determine num x pages to get
      // get x pages, aggregate, return data

      this.$axios(url1, this.APIconfig)
      .then(response => {
        let gamesCount = response.data.Response.allPvP.allTime.activitiesEntered.basic.value
        let currPage = 0
        let curr = 0
        let urls = []

        while (curr <= gamesCount) {
          // create unique URLs for each page

          let thisURL = 'https://www.bungie.net/Platform/Destiny2/' + this.membershipType + '/Account/' + this.memID + '/Character/' + this.currChar + '/Stats/Activities/?mode=5&count=250&page=' + currPage
          urls.push( this.$axios(thisURL, this.APIconfig))

          currPage += 1
          curr += 250
        }

        let games = []

        Promise.all(urls).then((response) => {
          response.forEach((gamesData) => {
            games.push(...gamesData.data.Response.activities)
          }) 

          this.crucibleData =  JSON.stringify(games)
        })
      })
    },
    getPGCR: function () {
      let activityID = '3816936971'
      let url = 'https://www.bungie.net/Platform/Destiny2/Stats/PostGameCarnageReport/' + activityID
      
      this.$axios(url, this.APIconfig).then(response => {
        // console.log(response)
      })
    }
  }
}
</script>

<style>
</style>
