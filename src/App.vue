<template>
  <div id="app">
    <section class="hero is-dark">
      <div class="hero-body">
        <div class="container">
          <h1 class="title is-marginless">Pixela Dashboard</h1>

          <div class="level is-marginless">
            <div class="level-left">
              <div class="level-item">
                <h2 class="subtitle is-marginless">&ensp;managing multiple <a 
                  href="https://pixe.la/" 
                  class="has-text-primary"><u>Pixela</u></a> graphs</h2>
              </div>
            </div>
            <div class="level-right">
              <div class="level-item">
                <div class="field">
                  <input 
                    id="switchExample" 
                    v-model="shortMode"
                    type="checkbox" 
                    name="switchExample" 
                    class="switch is-rtl is-small is-rounded">
                  <label 
                    for="switchExample"
                    class="is-small">Short mode {{ shortMode ? 'ON' : 'OFF' }}</label>
                </div>
              </div>
            </div>
          </div>
          
          <div class="level">
            <div class="level-left"/>
            <div class="level-right">
              <div class="level-item">
                <label class="label is-small has-text-white is-marginless">USER-ID:&ensp;&ensp;</label>
                <label class="label is-small has-text-white is-marginless">{{ user }}</label>
                <label class="label is-small has-text-white is-marginless">&ensp;/&ensp;</label>
                <label class="label is-small has-text-white is-marginless">USER-TOKEN:&ensp;&ensp;</label>
                <div class="control">
                  <input 
                    v-model="token" 
                    type="text" 
                    class="input is-small">
                </div>
              </div>
            </div>
          </div>

        </div>
      </div>
    </section>

    <div
      v-if="!shortMode"
      class="container graph-container">
      <Pixela
        v-for="(g, key, index) in graphs"
        :key="index"
        :user= "user"
        :token= "token"
        :graph-name="g" />
    </div>

    <div
      v-if="shortMode"
      class="container graph-container">
      <div class="columns is-centered is-multiline">
        <PixelaShortMode
          v-for="(g, key, index) in graphs"
          :key="index"
          :user= "user"
          :token= "token"
          :graph-name="g" />
      </div>
    </div>

  </div>
</template>

<script>
import Pixela from './components/Pixela.vue'
import PixelaShortMode from './components/PixelaShortMode.vue'
import config from './pixela-config.json'
import { isMobile } from 'mobile-device-detect';

export default {
  name: 'App',
  components: {
    Pixela,
    PixelaShortMode
  },
  data() {
    return {
      user: config.user,
      token: '',
      graphs: config.graphs,
      shortMode: isMobile
    }
  }
}
</script>

<style lang="scss">
@import "../node_modules/bulma/bulma.sass";
@import '../node_modules/bulma-extensions/bulma-switch/dist/css/bulma-switch.min.css';
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #2c3e50;
  margin-top: 20px;
}
body { overflow-x: hidden; }
.graph-container {
  padding-top: 1.5rem;
  padding-bottom: 1.5rem;
  padding-left: 0.5rem;
  padding-right: 0.5rem;
}
</style>
