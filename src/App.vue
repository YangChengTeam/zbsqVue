<template>
  <div id="app">
    <router-view/>
  </div>
</template>

<script>
export default {
  name: 'App'
}
</script>

<style>
@import "../static/css/minirefresh.min.css";
* { margin: 0; }
body {
  background:#f1f1f1;
}
.nav {
   background: #ff6600;
   height: 60px;
   position: fixed;
   left: 0;
   top: 0;
   z-index: 1000000;
   width: 100%;
   display: flex;
   align-items: center;
}

.nav > img {
   margin-left: 10px;
}

.nav > h2 {
   font-size: 20px;
   color: #fff;
   font-weight: normal;
}

</style>
