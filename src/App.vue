<template>
  <div id="app">
    <nav class="navbar-nav fixed-top">
      <h1 class="navbar-text pt-5">Five Day Forecast</h1>
    </nav>
    <Weather />
  </div>
</template>

<script>
import Weather from './components/Weather'

export default {
  name: 'App',
  components: {
    Weather
  }
}
</script>
