<template>
  <div>
    <div style="background: grey; padding-bottom: 10px; padding-top: 10px;">
      <div class="fb-header">
        <h1 class="text-center" style="font-size: 5rem;">Walton Robotics Team 2974</h1>
        <img src="~/assets/wrtlogo.png" style="height: 128px;" />
      </div>
    </div>
    <div class="fb-page-container">
      <div style="flex: 4">
        <div class="container-fluid px-3 py-3">
          <div class="row">
            <div v-bind:class="{'col-lg-12 col-xl-8': eventIsSyncing,  'col-lg-16 col-xl-12': !eventIsSyncing}">
              <h2 class="text-center">{{ eventName }}
                - Matches</h2>
              <table class="table table-striped table-dark table-borderless rounded text-center">
                <thead>
                  <tr>
                    <th class="bg-secondary">Match</th>
                    <th class="bg-secondary" v-if="eventIsSyncing">Time
                      <small>(scheduled / predicted / actual)</small>
                    </th>
                    <th class="bg-secondary" v-if="!eventIsSyncing">Time
                      <small>(scheduled / actual)</small>
                    </th>
                    <th colspan="3" class="font-weight-bold bg-danger">Red</th>
                    <th colspan="3" class="font-weight-bold bg-primary">Blue</th>
                    <th colspan="3" class="bg-secondary">Status</th>
                  </tr>
                </thead>
                <tbody>
                  <tr v-for="match in matches">
                    <th>{{match.comp_level.toUpperCase()}}#{{match.match_number}}</th>
                    <td>{{formatTime(match.time)}}
                      /
                      {{ eventIsSyncing ? formatTime(match.predicted_time) + ' / ' : '' }}
                      {{formatTime(match.actual_time)}}</td>
                    <td v-for="team in match.alliances.red.team_keys"
                      :class="{myteam: isMyTeam(team), 'bg-danger': isMyTeam(team)}">{{team.replace('frc','')}}</td>
                    <td v-for="team in match.alliances.blue.team_keys"
                      :class="{myteam: isMyTeam(team), 'bg-primary': isMyTeam(team)}">{{team.replace('frc','')}}</td>
                    <td :class="{'text-danger': redWin(match), 'text-primary': blueWin(match)}"
                      style="font-weight: bold;">{{calcStatus(match)}}</td>
                  </tr>
                </tbody>
              </table>
              <span class="text-muted">
                Last Updated:
                {{moment(updated).format('h:mm:ssa YYYY-MMM-DD')}}
                <br />
                Data powered by The Blue Alliance API
                <br />
                Original dashboard written by FRC Team 4909 - TechplexEngineer/pipitdisplay on GitHub
              </span>
            </div>
            <div class="col-lg-12 col-xl-4" v-if="eventIsSyncing">
              <div class="row">
                <div class="col-lg-6 col-xl-12">
                  <h2 class="text-center display-5">
                    <small class="smaller" v-if="team">FRC{{ team }}</small>
                    {{nextMatch.title}}
                    <small class="smaller">{{nextMatch.comp_level}}#{{nextMatch.match_number}}</small>
                  </h2>
                  <div class="row text-center">
                    <div class="col text-nowrap">Scheduled:
                      {{formatTime(nextMatch.time)}}</div>
                    <div class="col text-nowrap">Predicted:
                      {{formatTime(nextMatch.predicted_time)}}</div>
                  </div>
                  <h2 class="text-center pt-2 pb-2" style="white-space: nowrap;">
                    <small style="font-size: 75%;">Countdown:</small>
                    <VueCountdown :time="nextMatch.countdown">
                      <template slot-scope="props">
                        <span v-if="props.hours>0">{{ props.hours }}h
                        </span>
                        {{ props.minutes }}m
                        {{ props.seconds }}s</template>
                    </VueCountdown>
                    <small style="font-size: 35%;">(to
                      {{nextMatch.countdownTo}})</small>
                  </h2>
                  <table class="table table-striped table-dark table-borderless rounded shadow-sm">
                    <thead class="text-center thead-dark">
                      <tr class="rounded-top">
                        <th class="font-weight-bold bg-danger">Red</th>
                        <th class="font-weight-bold bg-primary">Blue</th>
                      </tr>
                    </thead>
                    <tbody class="text-center rounded-bottom">
                      <tr v-if="'alliances' in nextMatch" v-for="idx in 3">
                        <td :class="{'bg-danger': isMyTeam(nextMatch.alliances.red.team_keys[idx-1])}">
                          <span
                            :class="{myteam: isMyTeam(nextMatch.alliances.red.team_keys[idx-1])}">{{nextMatch.alliances.red.team_keys[idx-1].replace('frc',
                            '').trim()}}</span>
                          <br>
                          <small>{{getRanking(nextMatch.alliances.red.team_keys[idx-1])}}</small>
                        </td>
                        <td :class="{'bg-primary': isMyTeam(nextMatch.alliances.blue.team_keys[idx-1])}">
                          <span
                            :class="{myteam: isMyTeam(nextMatch.alliances.blue.team_keys[idx-1]), 'bg-primary': isMyTeam(nextMatch.alliances.blue.team_keys[idx-1])}">{{nextMatch.alliances.blue.team_keys[idx-1].replace('frc','').trim()}}</span>
                          <br>
                          <small>{{getRanking(nextMatch.alliances.blue.team_keys[idx-1])}}</small>
                        </td>
                      </tr>
                    </tbody>
                  </table>
                </div>
                <div class="col-lg-6 col-xl-12">
                  <h2 class="text-center">Rankings
                    <small class="smaller">Rank
                      {{ourRank}}
                      of
                      {{teamCount}}</small>
                  </h2>
                  <table class="table table-striped table-dark table-borderless rounded shadow-sm">
                    <thead class="text-center thead-dark">
                      <tr class="rounded-top">
                        <th class="font-weight-bold bg-secondary">Rank</th>
                        <th class="font-weight-bold bg-secondary">Team</th>
                        <th class="bg-secondary">RP</th>
                        <th class="bg-secondary">RP Avg.</th>
                        <th class="bg-secondary">W-L-T</th>
                        <th class="bg-secondary">Played</th>
                      </tr>
                    </thead>
                    <tbody class="text-center rounded-bottom">
                      <tr v-for="team in rankings.slice(0,8)" :class="{'bg-danger': isMyTeam(team.team_key)}">
                        <td>{{team.rank}}</td>
                        <td :class="{myteam: isMyTeam(team.team_key)}">{{team.team_key.replace('frc', '')}}</td>
                        <td>{{team.extra_stats[0]}}</td>
                        <td>{{team.sort_orders[0]}}</td>
                        <td>{{team.record.wins}}-{{team.record.losses}}-{{team.record.ties}}</td>
                        <td>{{team.matches_played}}</td>
                      </tr>
                    </tbody>
                  </table>
                </div>
              </div>
            </div>
          </div>
        </div>
        <br />
        <br />
        <!-- Data Input -->
        <div class="row text-muted">
          <div class="col-2 text-center">
            Optional Team Filter:
            <b-form-input type="number" v-model="team" placeholder="####"></b-form-input>
          </div>
          <div class="col-4 text-center">
            Event:
            <b-form-select v-model="event" :options="eventOptions"></b-form-select>
          </div>
          <div class="col-2 text-center">
            Event is syncing:
            <b-form-select v-model="eventIsSyncing">
              <option :value='true'>Yes</option>
              <option :value='false'>No</option>
            </b-form-select>
          </div>
        </div>
      </div>
      <div style="flex: 1;" class="sponsors">
        <img src="~/assets/novelis.png" />
        <img src="~/assets/carrier.svg" />
        <img src="~/assets/automated-logic.svg" />
        <img src="~/assets/difco.webp" />
        <img src="~/assets/ccsd.webp" />
      </div>
    </div>
  </div>
</template>

<script>
  import VueCountdown from '@chenfengyuan/vue-countdown';
  import moment from 'moment';
  import tba from '~/plugins/tba.js';

  // from https://codeburst.io/throttling-and-debouncing-in-javascript-b01cad5c8edf
  // https://stackoverflow.com/questions/45178621/how-to-correctly-use-vue-js-watch-with-lodash-debounce
  const debounce = (func, delayms) => {
    let inDebounce
    return function () {
      const context = this
      const args = arguments
      clearTimeout(inDebounce)
      inDebounce = setTimeout(() => func.apply(context, args), delayms)
    }
  }

  let updateHandle = null;

  export default {
    components: {
      VueCountdown
    },
    data: function () {
      return {
        team: null,
        event: null,
        eventIsSyncing: true,
        rankings: [],
        nextMatch: {},
        matches: [],
        ourRank: 'unknown',
        teamCount: 'unknown',
        updated: new Date(),
        events: [],
        isFinals: false
      }
    },
    async asyncData({ params }) {
    },
    mounted() {
      // if we have a team number and/or event key stored, restore them into the
      // component's data. Changes to the components data trigger the corresponding
      // functions in the watch section.
      if (localStorage.getItem("teamNumber")) {
        this.team = localStorage.getItem("teamNumber") || "2974";
      }
      if (localStorage.getItem("eventKey")) {
        this.event = localStorage.getItem("eventKey") || "2024scand";
      }

      tba.defaults.headers.common['X-TBA-Auth-Key'] = this.$config.tbaApiKey;

      updateHandle = setInterval(this.updateData, 10 * 1000);
    },
    beforeDestroy() {
      clearInterval(updateHandle);
      updateHandle = null;
    },
    methods: {
      calcStatus(match) {
        let out = "????";

        // match not yet played
        if (!match.actual_time) {
          return "";
        }

        // no team filter
        if (!this.team) {

          // the actual_time can be returned before the score_breakdown is posted
          if (!match.score_breakdown) {
            return "...."
          }
          if (match.alliances.red.score > match.alliances.blue.score) {
            out = "Red";
            // Show the ranking points for qualification matches
            if (match.comp_level == "qm") {
              out += ` ${match.score_breakdown.red.rp} / ${match.score_breakdown.blue.rp}`
            }
          } else if (match.alliances.red.score < match.alliances.blue.score) {
            out = "Blue";
            // Show the ranking points for qualification matches
            if (match.comp_level == "qm") {
              out += ` ${match.score_breakdown.blue.rp} / ${match.score_breakdown.red.rp}`
            }
          } else {
            out = "Tie";
          }

          return out;
        }

        if (match.alliances.red.team_keys.includes(`frc${this.team}`)) {

          if (match.alliances.red.score > match.alliances.blue.score) {
            out = "Win";
          } else if (match.alliances.red.score < match.alliances.blue.score) {
            out = "Loss";
          } else {
            out = "Tie";
          }
          if (match.comp_level == "qm") {
            out += ` ${match.score_breakdown.red.rp}`
          }

        } else if (match.alliances.blue.team_keys.includes(`frc${this.team}`)) {

          if (match.alliances.blue.score > match.alliances.red.score) {
            out = "Win";
          } else if (match.alliances.blue.score < match.alliances.red.score) {
            out = "Loss";
          } else {
            out = "Tie";
          }
          if (match.comp_level == "qm") {
            out += ` ${match.score_breakdown.blue.rp}`
          }
        } else {
          return "????";
        }
        return out;
      },
      redWin(match) {
        if (!match.actual_time) {
          return false;
        }
        return match.alliances.red.score > match.alliances.blue.score;
      },
      blueWin(match) {
        if (!match.actual_time) {
          return false;
        }
        return match.alliances.blue.score > match.alliances.red.score
      },
      formatTime(timestamp) {
        if (!timestamp) {
          return "--:----";
        }
        return moment(timestamp * 1000).format("h:mma");
      },
      getRanking(teamKey) {
        if (!this.rankings || !this.rankings.length) {
          return "no rankings loaded yet";
        }
        const team = this.rankings.filter((t) => (t.team_key === teamKey))[0];
        //Null check for teams with no rank
        if (team) {
          return `Rank ${team.rank} / RP ${team.sort_orders[0]}`;
        }
        return 'No Rank';
      },
      isMyTeam(teamKey) {
        return (`frc${this.team}` == teamKey);
      },
      updateData: debounce(function () {
        console.log("refresh data");
        if (!this.event) {
          console.log(`Missing required info: event:${this.event}`);
          return;
        }
        let team = this.team;
        let event = this.event;
        this.updated = new Date();

        tba.get(`/event/${event}/rankings`).then((res) => {
          // console.log("rankings", JSON.stringify(res.data, null, 4));
          this.rankings = res.data.rankings;
        });

        if (!this.team) {
          console.log('No team filter set, showing all matches');

          tba.get(`/event/${event}/matches`).then((res) => {
            // sort by scheduled time
            let matches = res.data.sort((a, b) => (a.time - b.time));
            // console.log("matches", JSON.stringify(matches[0], null, 4))

            // hide qualifing matches once quarter finals start
            let matchCompLevels = matches.map((m) => (m.comp_level));
            this.isFinals = matchCompLevels.includes("qf") || matchCompLevels.includes("sf");
            if (this.isFinals) {
              matches = matches.filter((m) => (m.comp_level !== "qm"));
              matches.forEach((m) => {
                if (m.comp_level == "sf") {
                  m.match_number = m.set_number;
                }
              });
            }

            this.matches = matches;

            // get a list of matches that haven't been played yet
            let notPlayed = matches.filter(m => (!m.actual_time));

            // current match is [0] net match is [1]
            this.nextMatch = notPlayed.sort((a, b) => (a.time - b.time))[0]
            this.nextMatch.countdown = (this.nextMatch.time * 1000 - Date.now());
            this.nextMatch.countdownTo = "scheduled";
            if (this.nextMatch.countdown < 0) {
              console.log("Countdown is zero");
              this.nextMatch.countdown = 0;
            }
            this.nextMatch.title = "Next Match";
            this.nextMatch.comp_level = this.nextMatch.comp_level.toUpperCase();

            // console.log("matches", JSON.stringify(, null, 4));
          });
          return;
        }

        try {
          tba.get(`/team/frc${team}/event/${event}/matches`).then((res) => {
            // sort by scheduled time
            let matches = res.data.sort((a, b) => (a.time - b.time));

            // hide qualifing matches once quarter finals start
            let matchCompLevels = matches.map((m) => (m.comp_level));
            this.isFinals = matchCompLevels.includes("qf") || matchCompLevels.includes("sf");
            if (this.isFinals) {
              matches = matches.filter((m) => (m.comp_level !== "qm"));
              matches.forEach((m) => {
                if (m.comp_level == "sf") {
                  m.match_number = m.set_number;
                }
              });
            }

            this.matches = matches;
          });

          tba.get(`/team/frc${team}/event/${event}/status`).then(async (res) => {

            if (!res.data) {
              console.log("no data returned");
              return;
            }

            this.teamCount = res.data.qual.num_teams;
            this.ourRank = res.data.qual.ranking.rank;

            if (res.data.next_match_key) {
              let nextMatchRes = await tba.get(`/match/${res.data.next_match_key}/simple`);
              this.nextMatch = nextMatchRes.data;
              this.nextMatch.title = "Next Match";
              this.nextMatch.comp_level = this.nextMatch.comp_level.toUpperCase();
              if (this.nextMatch.comp_level == "SF") {
                this.nextMatch.match_number = this.nextMatch.set_number;
              }

              let countdownTo = this.nextMatch.predicted_time
              this.nextMatch.countdownTo = "predicted";
              if (this.nextMatch.time > this.nextMatch.predicted_time) {
                this.nextMatch.countdownTo = "scheduled";
                countdownTo = this.nextMatch.time;
              }

              this.nextMatch.countdown = (countdownTo * 1000 - Date.now());
              if (this.nextMatch.countdown < 0) {
                console.log("Countdown is zero");
                this.nextMatch.countdown = 0;
              }

              this.nextMatch.title = "Next Match";
              this.nextMatch.comp_level = this.nextMatch.comp_level.toUpperCase();

            } else if (res.data.last_match_key) {
              let nextMatchRes = await tba.get(`/match/${res.data.last_match_key}/simple`);
              this.nextMatch = nextMatchRes.data;
              this.nextMatch.title = "Last Match"; //temporary @fixme for screenshots
              this.nextMatch.comp_level = this.nextMatch.comp_level.toUpperCase();
              if (this.nextMatch.comp_level == "SF") {
                this.nextMatch.match_number = this.nextMatch.set_number;
              }
            } else {
              this.nextMatch = {};
            }
          });
        } catch (e) {
          // we expect that sometimes the debounce will happen before the user has completed
          // entering the team
        }
      }, 500),
      moment: moment

    },
    computed: {
      eventOptions: function () {
        // events aren't loaded yet
        if (!this.events) {
          return [];
        }
        let events = this.events.map(event => ({
          value: event.key,
          text: event.name
        }));
        events.sort((a, b) => (a.text.localeCompare(b.text)));
        events.unshift({ value: null, text: 'Please select an event' });
        return events;
        // <option v-for="event in events" name="event.key">{{ event.name }}</option>
      },
      eventName: function () {
        // console.log("options", this.eventOptions, this.event);
        let events = this.eventOptions.filter(e => (e.value == this.event));
        // console.log("events", events);
        if (events.length) {
          return events[0].text
        }
        return "";
      }
    },
    watch: {
      event: function (value, oldValue) {
        console.log(`Event changed from ${oldValue} to ${value}`);
        localStorage.setItem("eventKey", value);

        this.updateData();
      },
      team: function (value, oldValue) {
        console.log(`team changed from ${oldValue} to ${value}`);
        localStorage.setItem("teamNumber", value);

        // load the list of events for this year
        // used for the event dropdown
        tba.get(`/events/${new Date().getFullYear()}`).then((res) => {
          this.events = res.data;
        });

        this.updateData();
      }
    }

  }

</script>

<style scoped>
  .rounded-top-left {
    border-top-left-radius: .25rem !important;
  }

  .rounded-top-right {
    border-top-right-radius: .25rem !important;
  }

  .rounded-bottom-left {
    border-bottom-left-radius: .25rem !important;
  }

  .rounded-bottom-right {
    border-bottom-right-radius: .25rem !important;
  }

  .smaller {
    font-size: 70%;
  }

  .table td {
    padding: 0.65rem 0.75rem;
    vertical-align: middle;
  }

  .myteam {
    font-weight: bold;
    text-decoration: underline;
    /*     font-size: 125%; */
  }

  .fb-header {
    display: flex;
    flex-direction: row;
    flex-wrap: nowrap;
    justify-content: space-evenly;
    align-items: center;
    align-content: center;
  }

  .fb-page-container {
    display: flex;
    flex-direction: row;
    flex-wrap: nowrap;
    justify-content: space-evenly;
    align-items: flex-start;
    align-content: center;
  }

  .sponsors {
    display: flex;
    justify-content: center;
    align-items: center;
    flex-direction: column;
    padding-right: 1rem;
  }

  .sponsors img {
    padding-top: 20px;
    max-width: 100%;
  }
</style>
