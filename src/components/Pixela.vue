<template>
  <div class="pixela">
    <h2 class="title is-5">{{ graphName }}</h2>

    <div class="columns">
      <div
        class="column is-8">
        <div
          v-if="loaded"
          v-html="svg"/>
      </div>

      <div class="column">
        <div class="label">Input</div>

        <div class="field">
          <label class="label is-small">Date</label>
          <div class="control">
            <input 
              v-model="date" 
              type="text" 
              class="input is-small">
          </div>
        </div>

        <div class="field">
          <label class="label is-small">Quantity</label>
          <div class="control">
            <input 
              v-model="quantity" 
              type="text" 
              class="input is-small">
          </div>
        </div>

        <div class="field">
          <div class="control">
            <button 
              :class="{ 'is-loading': isUpdating }"
              class="button is-link is-small" 
              @click="updateData">Record / Update</button>
          </div>
        </div>

        <p 
          v-if="isSuccess"
          class="help is-success is-center">{{ msg }}</p>
        <p 
          v-if="!isSuccess" 
          class="help is-danger is-center">{{ msg }}</p>

      </div>

      <div class="column">
        <div class="label">Streaks</div>
        <div class="has-text-right is-size-6">
          type: 
          <div class="select is-small">
            <select
              v-model="goal"
              @change="calcStreaks">
              <option>7</option>
              <option>6</option>
              <option>5</option>
              <option>4</option>
              <option>3</option>
              <option>2</option>
              <option>1</option>
            </select>
          </div> per week
        </div>
        <br>

        <div class="has-text-right has-text-primary has-text-weight-bold is-size-5">
          <small>current:</small> {{ current }} <small>streaks</small>
        </div>

        <div class="has-text-right has-text-primary has-text-weight-bold is-size-5">
          <small>longest:</small> {{ longest }} <small>streaks</small>
        </div>
      </div>

    </div>

  </div>
</template>

<script>
import axios from 'axios';
import tippy from 'tippy.js';

export default {
  props: {
    user: {
      type: String,
      required: true
    },
    token: {
      type: String,
      required: true
    },
    graphName: {
      type: String,
      required: true
    }
  },
  data() {
    return {
      loaded: true,
      graphUrl: '',
      date: '', 
      quantity: '0',
      msg: '',
      isSuccess: null,
      isUpdating: false,
      svg: '',
      goal: 7,
      current: -1,
      longest: -1 
    }
  },
  created() {
    // set graph URL
    this.graphUrl = "https://pixe.la/v1/users/" + this.user + "/graphs/" + this.graphName

    // set today's date string
    var dateObj = new Date()
    var yyyy = dateObj.getFullYear()
    var MM = ('0' + (dateObj.getMonth() + 1)).slice(-2)
    var dd = ('0' + dateObj.getDate()).slice(-2)
    this.date = yyyy + MM + dd

    // get svg content
    axios.get(this.graphUrl).then(response => {
      this.svg = response.data
      this.calcStreaks()
    });
  },
  updated() {
    tippy('.each-day', { arrow: true });
  },
  methods: {
    updateData() {
      // set target URL
      var putUrl = this.graphUrl + '/' + this.date
      this.initForm()

      if (this.token === '' || this.date === '' | this.quantity === '') {
        this.updateForm('fail')
        this.msg = 'fill all input (USER-TOKEN, date, quantity)'
        return
      }

      // PUT (update) quantity of this.date to this. quantity on this.graphName
      axios({
        method: 'put',
        url: putUrl,
        headers: { 'X-USER-TOKEN': this.token },
        data: { quantity: this.quantity }
      }).then(response => {
        console.log("update " + response.data.message);
        this.updateForm('success')
      }).catch(error => {
        console.log(error);
        this.updateForm('fail')
      });
    },
    initForm() {
      // initialize record/update-related variables
      this.loaded = false
      this.msg = ''
      this.isUpdating = true
    },
    updateForm(isSuccess) {
      // update record/update-related variables
      this.loaded = true
      this.isUpdating = false

      if (isSuccess === 'success') {
        // if action sucessed
        this.isSuccess = true

        // update greaph svg
        axios.get(this.graphUrl).then(response => {
          this.svg = response.data
        });
        this.msg = 'Successfully recorded/updated'
      } else {
        // if action failed
        this.isSuccess = false
        this.msg = 'Error: something wrong'
      }
    },
    calcStreaks() {
      // init
      this.current = -1
      this.longest = -1

      // parse svg to dom
      const domParser = new DOMParser();
      const parsedSVGDoc = domParser.parseFromString(this.svg, 'image/svg+xml')
      const parsedSVG = parsedSVGDoc.childNodes[0]

      // get rooot element of tip erctangles
      const rootg = parsedSVG.childNodes[1] // g tag
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

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.pixela { margin-bottom:  1.5rem; }
</style>
