<template>
  <main>
    <b-overlay :show="show" rounded="sm" :opacity="0.7" blur="2px">
      <template #overlay>
        <div class="text-center">
          <b-spinner type="grow" variant="dark"></b-spinner>
          <h2>Loading data...</h2>
        </div>
      </template>
      <b-container>
        <h1 class="text-center text-white pt-5">Sports Interactive Players</h1>
        <div>
          <b-form-input
            v-model="search"
            placeholder="Search players by TName or PFName"
            trim
            debounce="500"
            autofocus
          ></b-form-input>
        </div>
        <div class="cards" v-if="filteredPlayerList.length > 0">
          <b-card
            :title="player.PFName"
            :img-src="`/images/player-images/${player.Id}.jpg`"
            :img-alt="player.PFName"
            img-top
            tag="article"
            class="mb-3 mx-2"
            v-for="(player, index) in filteredPlayerList"
            :key="index"
          >
            <b-card-text>
              <div
                class="d-flex justify-content-between align-items-center pt-2"
              >
                <div>
                  <span>Skill:</span>&nbsp;
                  <span class="skill-badge" v-text="player.SkillDesc"></span>
                </div>
                <div>
                  <span class="font-weight-normal">Value:</span>&nbsp;
                  <strong>${{ player.Value }}</strong>
                </div>
              </div>
              <div
                v-if="
                  'UpComingMatchesList' in player &&
                  player.UpComingMatchesList.length > 0
                "
              >
                <div
                  class="py-2"
                  v-if="
                    player.UpComingMatchesList[0].CCode &&
                    player.UpComingMatchesList[0].VsCCode
                  "
                >
                  Upcoming match :
                  <span
                    ><code
                      ><small>{{
                        player.UpComingMatchesList[0].CCode
                      }}</small></code
                    >
                    vs
                    <code
                      ><small>{{
                        player.UpComingMatchesList[0].VsCCode
                      }}</small></code
                    ></span
                  >
                </div>
                <h6
                  class="text-muted"
                  v-if="player.UpComingMatchesList[0].MDate"
                >
                  Next Match: {{ getDate(player.UpComingMatchesList[0].MDate) }}
                </h6>
              </div>
            </b-card-text>
          </b-card>
        </div>
        <div v-else class="no-data-div">
          <b-img src="/images/no-data.svg" height="400"></b-img>
          <h2 class="text-white mt-3">
            No data found {{ search.length > 0 ? " for " + search : "" }}
          </h2>
        </div>
      </b-container>
    </b-overlay>
  </main>
</template>

<script>
import axios from "axios";
import moment from "moment";

export default {
  data() {
    return {
      teamList: [],
      playerList: [],
      filteredPlayerList: [],
      show: false,
      search: "",
    };
  },
  async beforeMount() {
    try {
      this.show = true;

      let { data } = await axios.get(process.env.VUE_APP_API_BASE_URL);

      this.teamList = data.teamList;

      this.filteredPlayerList = this.playerList = data.playerList;

      this.show = false;
    } catch (error) {
      console.log(error);
      this.show = false;
    }
  },
  watch: {
    search() {
      this.filteredPlayerList = this.playerList;
      if (this.search.length > 2) {
        let filter = new RegExp(this.search, "i");
        this.filteredPlayerList = this.playerList.filter((player) => {
          return player.TName.match(filter) || player.PFName.match(filter);
        });
      }
    },
  },
  methods: {
    getDate(date) {
      if (!date) return "";
      return moment(date).format("DD-MM-YYYY h:mm:ss a");
    },
  },
};
</script>

<style scoped>
main {
  background-color: #303032;
  min-height: 100vh;
}

.no-data-div {
  min-height: 80vh;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
}

.cards {
  display: flex;
  flex-flow: row wrap;
  justify-content: center;
  padding: 20px 0px;
}

.cards > * {
  flex-basis: 33.33%;
  max-width: 20rem;
  cursor: pointer;
}

.skill-badge {
  width: auto;
  padding: 3px 5px;
  border-radius: 5px;
  color: white;
  background-color: #28a745;
  display: inline-block;
  font-size: 12px;
}

.card-img-top {
  height: 318px;
  display: flex;
  justify-content: center;
  align-items: center;
  position: relative;
  min-width: 100%;
}

.card-img-top:after {
  content: "🚫 image not loaded";
  display: flex;
  justify-content: center;
  align-items: center;
  flex-direction: column;
  min-height: 100%;
  position: absolute;
  z-index: 2;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background-color: lightgray;
  margin-bottom: 5px;
}

@media (max-width: 992px) {
  .cards > * {
    flex-basis: 50%;
    max-width: 20rem;
  }
}

@media (max-width: 768px) {
  .no-data-div > img {
    width: 80vw;
    height: auto;
  }
  .cards > * {
    flex-basis: 100%;
  }
}
</style>