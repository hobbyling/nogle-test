<template>
  <div id="app">
    <h1>Order Book</h1>
    <hr />

    <div class="table">
      <div class="tr">
        <div>Price(USD)</div>
        <div>Size</div>
        <div>Total</div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'App'
}
</script>

<style lang="scss">
#app {
  width: 375px;
  margin: 0 auto;
  background: #000;
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #fff;
  margin-top: 60px;
  padding: 10px;
}

h1 {
  text-align: left;
}

.table {
  .tr {
    display: flex;
    justify-content: space-between;
    align-items: center;
  }
}
</style>
