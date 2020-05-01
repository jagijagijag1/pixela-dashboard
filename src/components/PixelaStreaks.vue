<template>
  <div class="pixela-streak">
    <div class="label">Streaks</div>
    <div class="field is-size-6 is-vertical-center is-pulled-right">
      <div class="small-label">Goal: </div>
      <input 
        v-model="goal" 
        type="number" 
        class="input is-small"
        min="1"
        max="7"
        style="width: 2.5rem; text-align: center;"
        @change="calcStreaks">
      <div class="small-label">/ week</div>
    </div>

    <div class="has-text-right has-text-primary has-text-weight-bold is-size-5">
      <small>current:</small> <big>{{ current }}</big> <small>streaks</small>
    </div>

    <div class="has-text-right has-text-primary has-text-weight-bold is-size-5">
      <small>best:</small> <big>{{ longest }}</big> <small>streaks</small>
    </div>
  </div>
</template>


<script>
export default {
  props: {
    svg: {
      type: String,
      required: true
    }
  },
  data() {
    return {
      goal: 7,
      current: -1,
      longest: -1 
    }
  },
  watch: {
    svg: function() {
      this.calcStreaks()
    }
  },
  mounted() {
    this.calcStreaks()
  },
  methods: {
    calcStreaks() {
      // init
      this.current = -1
      this.longest = -1

      // parse svg to dom
      const domParser = new DOMParser();
      const parsedSVGDoc = domParser.parseFromString(this.svg, 'image/svg+xml')
      const parsedSVG = parsedSVGDoc.childNodes[1]

      // get rooot element of tip erctangles
      const rootg = parsedSVG.childNodes[3] // g tag
      const children = [].slice.call(rootg.childNodes, 0).reverse()
      var tmpStreak = 0
      children.forEach(e => {
        // for each "g" tag element
        if (e.tagName == "g"){
          const sampleRect = e.getElementsByTagName("rect").item(0)
          // if g's child "rect" is exists & has pixel data
          if (sampleRect != null && sampleRect.hasAttribute("data-count")) {
            // collect pixel data
            const counts = Array.from(e.children, r => r.getAttribute("data-count"))

            // count streak
            if (this.goal == 7) {
              tmpStreak = this.countStreakEveryday(tmpStreak, counts)
            } else {
              tmpStreak = this.countSterakPerWeek(tmpStreak, this.goal, counts)
            }
          }
        }
      }, this);
    },
    countStreakEveryday(tmpStreak, counts) {
      counts.reverse().forEach(e => {
        if (e != 0) {
          // if data exists, count-up
          tmpStreak += 1
        } else {
          // if no data, record to current & longest
          this.current = (this.current == -1) ? tmpStreak : this.current
          this.longest = (this.longest < tmpStreak) ? tmpStreak : this.longest
          tmpStreak = 0
        }
      })
      return tmpStreak
    },
    countSterakPerWeek(tmpStreak, goal, counts) {
      // calc # of streak
      const streak = counts.reduce((prev, cur) => {
        return (cur != 0) ? prev + 1 : prev
      }, 0)

      if (streak >= goal) {
        // if counts contains non-zero data more than goal count
        tmpStreak += streak
      } else {
        // if streak is not continued, record current & longest
        this.current = (this.current == -1) ? tmpStreak : this.current
        this.longest = (this.longest < tmpStreak) ? tmpStreak : this.longest
        tmpStreak = 0
      }
      return tmpStreak
    }
  }
}
</script>

<style scoped>
.is-vertical-center {
  display: flex;
  align-items: center;
}

.small-label {
  padding-left: 5%;
  padding-right: 5%;
}
</style>