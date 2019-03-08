<template>
  <div class="pixela">
    <h2 class="title is-5">{{ graphName }}</h2>

    <div class="level">
      <div class="level-left">
        <div class="level-item">
          <div
            v-if="loaded"
            style="width:720px;"
            v-html="svg"/>
        </div>

        <div class="level-item">
          <div>
            <div class="field is-horizontal">
              <div class="field-label is-small">
                <label class="label">Date&ensp;&ensp;&ensp;&ensp;</label>
              </div>
              <div class="field-body">
                <div class="field">
                  <div class="control">
                    <input 
                      v-model="date" 
                      type="text" 
                      class="input is-small">
                  </div>
                </div>
              </div>
            </div>

            <div class="field is-horizontal">
              <div class="field-label is-small">
                <label class="label">Quantity</label>
              </div>
              <div class="field-body">
                <div class="field">
                  <div class="control">
                    <input 
                      v-model="quantity" 
                      type="text" 
                      class="input is-small">
                  </div>
                </div>
              </div>
            </div>

            <div class="field is-horizontal">
              <div class="field-label is-small"/>
              <div class="field-body">
                <div class="field">
                  <div class="control">
                    <button 
                      :class="{ 'is-loading': isRecording }"
                      class="button is-primary is-small" 
                      @click="recordData">Record</button>
                  </div>
                </div>
                <div class="field">
                  <div class="control">
                    <button 
                      :class="{ 'is-loading': isUpdating }"
                      class="button is-link is-small" 
                      @click="updateData">Update</button>
                  </div>
                </div>
              </div>
            </div>
            <p 
              v-if="isSuccess"
              class="help is-success is-center">{{ msg }}</p>
            <p 
              v-if="!isSuccess" 
              class="help is-danger is-center">{{ msg }}</p>
          </div>
        </div>
      </div>
      <div class="level-right"/>
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
      isRecording: false,
      isUpdating: false,
      svg: ''
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
    });
  },
  updated() {
    tippy('.each-day', { arrow: true });
  },
  methods: {
    recordData() {
      // set target URL
      var postUrl = this.graphUrl
      this.initForm('record')

      if (this.token === '' || this.date === '' | this.quantity === '') {
        this.updateForm('record', 'fail')
        this.msg = 'fill all input (USER-TOKEN, date, quantity)'
        return
      }

      // POST this.date & this. quantity to this.graphName
      axios({
        method: 'post',
        url: postUrl,
        headers: { 'X-USER-TOKEN': this.token },
        data: { date: this.date, quantity: this.quantity }
      })
      .then(response => {
        console.log("record " + response.data.message);
        this.updateForm('record', 'success')
      }).catch(error => {
        console.log(error);
        this.updateForm('record', 'fail')
      });
    },
    updateData() {
      // set target URL
      var putUrl = this.graphUrl + '/' + this.date
      this.initForm('update')

      if (this.token === '' || this.date === '' | this.quantity === '') {
        this.updateForm('record', 'fail')
        this.msg = 'fill all input (USER-TOKEN, date, quantity)'
        return
      }

      // PUT (update) quantity of this.date to this. quantity on this.graphName
      axios({
        method: 'put',
        url: putUrl,
        headers: { 'X-USER-TOKEN': this.token },
        data: { quantity: this.quantity }
      })
      .then(response => {
        console.log("update " + response.data.message);
        this.updateForm('update', 'success')
      }).catch(error => {
        console.log(error);
        this.updateForm('update', 'fail')
      });
    },
    initForm(action) {
      // initialize record/update-related variables
      this.loaded = false
      this.msg = ''
      if (action === 'record') {
        this.isRecording = true
      } else if (action === 'update') {
        this.isUpdating = true
      }
    },
    updateForm(action, isSuccess) {
      // update record/update-related variables
      this.loaded = true
      this.isRecording = false
      this.isUpdating = false

      if (isSuccess === 'success') {
        // if action sucessed
        this.isSuccess = true

        // update greaph svg
        axios.get(this.graphUrl).then(response => {
          this.svg = response.data
        });

        if (action === 'record') {
          this.msg = 'Successfully recorded'
        } else if (action === 'update') {
          this.msg = 'Successfully updated'
        }
      } else {
        // if action failed
        this.isSuccess = false
        this.msg = 'Error: something wrong'
      }
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.pixela { margin-bottom:  1.5rem; }
</style>
