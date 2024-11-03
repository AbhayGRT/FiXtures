<template>
  <div class="matches-container">
    <div
      class="match-container"
      v-for="match in feed.matches"
      :key="match.match_id"
    >
      <div class="header">
        <div class="match-info">
          <span class="date"
            >{{
              formatStartDate(match.start_date)
            }}
            -
          </span>
          <span class="match-title">{{ match.series_name }}</span>
          <span class="match-overs"> - {{ match.event_name }}</span>
        </div>

        <div class="match-status">
          <div :class="`${matchDetails(match).className}`">
            {{ matchDetails(match).type }}
          </div>
          <div class="live-indicator">
            <div :class="`${liveRemove(match)}-dot`">
              <img
                src="https://assets-icc.sportz.io/static-assets/images/live.svg?v=1.8"
                alt="live-image"
                class="live-image-SI"
              />
            </div>
            <span class="live-text">{{ eventState(match) }}</span>
          </div>
        </div>
      </div>

      <div class="teams">
        <div
          class="team"
          v-for="(participant, index) in match.participants"
          :key="index"
        >
          <div class="team-info">
            <div class="flag">
              <img
                src="./assets/images/india.png"
                alt="flag"
                class="ind-flag"
              />
            </div>
            <span class="team-name">{{ participant.short_name }}</span>
          </div>
          <span class="score">
            {{
              participant.value
                ? participant.value
                : participant.value === ""
                ? "Yet to Bat"
                : index === 0
                ? formatMatchTime(match.start_date)
                : ""
            }}
          </span>
        </div>
      </div>
      <div class="innings-break">
        {{ match.short_event_status }}
      </div>
      <a :href="`..match-center/${match.game_id}`" class="link-match-centre">
        <button class="match-centre-btn">MATCH CENTRE ></button>
      </a>
    </div>
  </div>
</template>

<script>
import _ from 'lodash';

export default {
  name: "App",
  data() {
    return {
      feed: {
        matches: []
      }
    };
  },

  created() {
    this.fetchData();
  },

  computed: {
    sortedMatches() {
      return this.groupAndSortMatches(this.feed.matches);
    },
  },

  methods: {
    async fetchData() {
      try {
        const response = await fetch('http://localhost:3000/matches');
        const data = await response.json();
        this.feed.matches = this.groupAndSortMatches(data);
      } catch (error) {
        console.error("Error fetching data:", error);
      }
    },

    groupAndSortMatches(matches) {
      const grouped = _.groupBy(matches, match => this.eventState(match));
      const orderedStates = ["LIVE", "RECENT", "UPCOMING"];
      return _.flatMap(orderedStates, state => grouped[state] || []);
    },

    matchDetails(match) {
      try {
        const compTypeId = match.comp_type_id;
        let type;
        let className;

        if (["16", "13", "3"].includes(compTypeId)) {
          type = "T20I";
          className = "match-type-t20i";
        } else if (["2", "12"].includes(compTypeId)) {
          type = "ODI";
          className = "match-type-odi";
        } else if (compTypeId == "1") {
          type = "Test";
          className = "match-type-test";
        } else {
          type = "First Class";
          className = "match-type-first-class";
        }

        return { type, className };
      } catch (error) {
        console.error("Error in matchDetails:", error);
        return { type: "Unknown", className: "" };
      }
    },

    eventState(match) {
      try {
        const eventStateValue = match.event_state;
        if (eventStateValue === "R") {
          return "RECENT";
        } else if (eventStateValue === "U") {
          return "UPCOMING";
        } else {
          return "LIVE";
        }
      } catch (error) {
        console.error("Error in eventState:", error);
        return "UNKNOWN"; 
      }
    },

    liveRemove(match) {
      try {
        const eventStatus = match.event_state;
        if (eventStatus === "L") {
          return "live";
        } else if (eventStatus === "U") {
          return "upcoming";
        } else {
          return "recent";
        }
      } catch (error) {
        console.error("Error in liveRemove:", error);
        return "unknown"; 
      }
    },

    formatStartDate(dateString) {
    try {
      return dateString.substring(0, 10).split("-").reverse().join(".");
    } catch (error) {
      console.error("Error formatting start date:", error);
      return "Invalid Date"; 
    }
  },

  formatMatchTime(dateString) {
    try {
      return dateString.substring(12).split("+").join("-");
    } catch (error) {
      console.error("Error formatting match time:", error);
      return "Invalid Time"; 
    }
  },
  }
};
</script>


<style>
.match-type-t20i {
  color: #fc1dec;
  font-weight: 1000;
  font-size: 1.2rem;
}

.match-type-odi {
  color: #21cccc;
  font-weight: 1000;
  font-size: 1.2rem;
}

.match-type-test {
  color: #25d457;
  font-weight: 1000;
  font-size: 1.2rem;
}

.match-type-first-class {
  color: #d51717;
  font-weight: 1000;
  font-size: 1.2rem;
}
</style>