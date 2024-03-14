<script setup>
import { ref, watch, reactive } from 'vue'

const STATES = {
	"Bayern": {
  	votes: 6,
  },
  "Baden-Württemberg": {
  	votes: 6,
  },
  "Berlin": {
  	votes: 4,
  },
  "Brandenburg": {
  	votes: 4,
  },
  "Bremen": {
  	votes: 3,
  },
  "Hamburg": {
  	votes: 3,
  },
  "Hessen": {
  	votes: 5,
  },
  "Mecklenburg-Vorpommern": {
  	votes: 3,
  },
  "Niedersachsen": {
  	votes: 6,
  },
  "Nordrhein-Westfalen": {
  	votes: 6,
  },
  "Rheinland-Pfalz": {
  	votes: 4,
  },
  "Saarland": {
  	votes: 3,
  },
  "Sachsen": {
  	votes: 4,
  },
  "Sachsen-Anhalt": {
  	votes: 4,
  },
  "Schleswig-Holstein": {
  	votes: 4,
  },
  "Thürigen": {
  	votes: 4,
  },
}

const VOTE_TO_SLUG = {
  [VOTE_YES]: "1",
  [VOTE_NO]: "2",
  [VOTE_PRESENT]: "3",
  [VOTE_UNDECIDED]: "4"
}

const VOTE_FROM_SLUG = {
  "1": VOTE_YES,
  "2": VOTE_NO,
  "3": VOTE_PRESENT,
  "4": VOTE_UNDECIDED,
}

const VOTE_YES = "yes"
const VOTE_NO = "no"
const VOTE_PRESENT = "present"
const VOTE_UNDECIDED = "undecided"

const VOTE_COUNT_TOTAL = Object.values(STATES).reduce((sum, state) => sum+state.votes, 0)
const VOTE_COUNT_SUCCESS = Math.ceil(VOTE_COUNT_TOTAL / 2)

const widthPerVoteCount = voteCount => (voteCount / VOTE_COUNT_TOTAL * 100) + "%"

const voteDescription = voteCount => voteCount + " " + "|".repeat(voteCount)

const isVoteSuccess = () => votes.value[VOTE_YES] > VOTE_COUNT_TOTAL / 2

const VOTE_SLUG_KEY = "v"
const getVoteSlug = () => states.map(s => VOTE_TO_SLUG[s.votesFor]).join("")
const getVoteLink = () => `${window.location.protocol}//${window.location.host}${window.location.pathname}?${VOTE_SLUG_KEY}=${getVoteSlug()}`
const copyVoteLink = () => {
  const link = getVoteLink()
  navigator.clipboard.writeText(link).catch(console.error)
}

let states = reactive(Object.entries(STATES).map(([name, state]) => ({
  name,
  votes: state.votes,
  votesFor: VOTE_YES,
})))

states.sort((a, b) => {
  const nameA = a.name.toUpperCase()
  const nameB = b.name.toUpperCase()
  if (nameA < nameB) {
    return -1
  }
  if (nameA > nameB) {
    return 1
  }
  return 0
})

const initStateVotesFromURL = () => {
  const params = new URLSearchParams(window.location.search)
  const slug = params.get(VOTE_SLUG_KEY)
  if (!slug || slug.length != states.length) {
    return
  }
  const stateVotes = Array(states.length)
  // Read vote from string and return an empty one if it is invalid.
  for (let i = 0; i < slug.length; i++) {
    const key = slug.charAt(i)
    const vote = VOTE_FROM_SLUG[key]
    if (!vote) {
      console.error("Invalid vote parameter")
      return
    }
    stateVotes[i] = vote
  }
  stateVotes.forEach((v, i) => states[i].votesFor = v)
}
initStateVotesFromURL()

const newVote = () => ({
  [VOTE_YES]: 0,
  [VOTE_NO]: 0,
  [VOTE_PRESENT]: 0,
  [VOTE_UNDECIDED]: 0,
})

const countVotes = states => {
  const res = newVote()
  states.forEach(state => {
    res[state.votesFor] += state.votes
  })
  return res
}

let votes = ref(countVotes(states))

watch(states, states => {
  votes.value = countVotes(states)
})

</script>

<template>
  <div class="container maxw-sm d-flex flex-column align-items-center">
    <header>
      <h1>Bundesratstimme</h1>
      <div>Stimmenrechner für den Deutschen Bundesrat</div>
    </header>

    <div class="mt-4 w-100 overflow-x-auto">
      <table class="table minw-xs">
        <thead>
          <th rowspan="2">Bundesland</th>
          <th class="pe-3" rowspan="2">Stimmen</th>
          <th class="text-center" colspan="4">Abstimmungsverhalten</th>
        </thead>
        <thead>
          <th></th>
          <th></th>
          <th class="text-center ps-2 pe-2">Ja</th>
          <th class="text-center ps-2 pe-2">Nein</th>
          <th class="text-center ps-2 pe-2">Enth.</th>
          <th class="text-center ps-2 pe-2">Unklar</th>
        </thead>
        <tr v-for="state in states">
          <td class="pe-3">{{ state.name }}</td>
          <td class="text-start pe-3">{{ voteDescription(state.votes) }}</td>
          <td class="text-center">
            <!-- <label :for="state.name+'_yes'">Ja</label> -->
            <input type="radio" class="form-check-input" :id="state.name+'_yes'" :name="state.name" :value="VOTE_YES" :checked="state.votesFor===VOTE_YES" @input="state.votesFor=VOTE_YES" />
          </td>
          <td class="text-center">
            <!-- <label :for="state.name+'_no'">Nein</label> -->
            <input type="radio" class="form-check-input" :id="state.name+'_no'" :name="state.name" :value="VOTE_NO" :checked="state.votesFor===VOTE_NO" @input="state.votesFor=VOTE_NO"  />
          </td>
          <td class="text-center">
            <!-- <label :for="state.name+'_present'">Enthaltung</label> -->
            <input type="radio" class="form-check-input" :id="state.name+'_present'" :name="state.name" :value="VOTE_PRESENT" :checked="state.votesFor===VOTE_PRESENT" @input="state.votesFor=VOTE_PRESENT"  />
          </td>
          <td class="text-center">
            <!-- <label :for="state.name+'_undecided'">Unklar</label> -->
            <input type="radio" class="form-check-input" :id="state.name+'_undecided'" :name="state.name" :value="VOTE_UNDECIDED" :checked="state.votesFor===VOTE_UNDECIDED" @input="state.votesFor=VOTE_UNDECIDED"  />
          </td>
        </tr>
      </table>
    </div>

    <div class="mt-4 fs-6 fw-light">
      Für eine aktuelle Zusammensetzung der einzelnen Landesregierungen siehe den entsprechenden
      <a href="https://de.wikipedia.org/wiki/Land_(Deutschland)#Rahmendaten_der_L%C3%A4nder" target="_blank">Wikipedia Artikel</a>.
    </div>

    <div class="mt-4 w-100">
      <div class="text-center mb-2">Für eine Mehrheit notwendig sind {{ VOTE_COUNT_SUCCESS }} Stimmen.</div>
      <div class="d-flex justify-content-center align-items-center">
        <span class="square-box bcolor-yes ms-2 me-2"></span>
        <span>Ja: {{ votes[VOTE_YES] }}</span>
        <span class="square-box bcolor-no ms-2 me-2"></span>
        <span>Nein: {{ votes[VOTE_NO] }}</span>
        <span class="square-box bcolor-present ms-2 me-2"></span>
        <span>Enthaltung: {{ votes[VOTE_PRESENT] }}</span>
        <span class="square-box bcolor-undecided ms-2 me-2"></span>
        <span>Unklar: {{ votes[VOTE_UNDECIDED] }}</span>
      </div>
      <div>
        <div class="vote-diagram">
          <div class="bar bcolor-yes" :style="{width: widthPerVoteCount(votes[VOTE_YES])}"></div>
          <div class="bar bcolor-present" :style="{width: widthPerVoteCount(votes[VOTE_PRESENT])}"></div>
          <div class="bar bcolor-no" :style="{width: widthPerVoteCount(votes[VOTE_NO])}"></div>
          <div class="bar bcolor-undecided" :style="{width: widthPerVoteCount(votes[VOTE_UNDECIDED])}"></div>
          <div class="majority-marker"></div>
        </div>
      </div>
      <div class="text-center">
        Der Antrag ist
        <span :class="isVoteSuccess() ? 'color-yes' : 'color-no'">
          {{ isVoteSuccess() ? "angenommen" : "nicht angenommen" }}
        </span>.
      </div>
      <div class="d-flex justify-content-center align-items-center mt-4">
        <div class="input-group w-sm">
          <span class="input-group-text">Share</span>
          <input type="text" class="form-control" readonly="true" :value="getVoteLink()">
          <button class="btn btn-secondary" type="button" @click="copyVoteLink()">Link kopieren</button>
        </div>
      </div>
    </div>
    <footer class="mt-4">
      <a class="fs-7" href="https://github.com/MisterMX/bundesratstimme" target="_blank">Source Code</a>
    </footer>
  </div>
</template>

<style>
label {
  margin-left: 1em;
  margin-right: 0.5em;
}
.vote-diagram {
  position: relative;
  height: 40px;
  width: 100%;
  border: 1px solid #000;
}
.vote-diagram .bar {
  display: inline-block;
  height: 100%;
  transition: width 0.1s linear;
}
.bcolor-yes {
  background-color: var(--bs-success);
}
.bcolor-no {
  background-color: var(--bs-danger);
}
.bcolor-present {
  background-color: var(--bs-warning);
}
.bcolor-undecided {
  background-color: var(--bs-gray);
}
.color-yes {
  color: var(--bs-success);
}
.color-no {
  color: var(--bs-danger);
}
.vote-diagram .majority-marker {
  position: absolute;
  left: 50%;
  top: 0%;
  height: 100%;
  width: 3px;
  background-color: var(--bs-dark);
}

.square-box {
  display: inline-flex;
  height: 1em;
  width: 1em;
  border: 1px solid #000;
}

.w-sm {
  width: var(--bs-breakpoint-sm);
}
.maxw-sm {
  max-width: var(--bs-breakpoint-sm);
}
.minw-xs {
  min-width: var(--bs-breakpoint-xs);
}

.fs-7 {
  font-size: 0.7rem;
}
</style>
