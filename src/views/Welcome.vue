<template>

  <div class="welcome">
    <div class="greypath"></div>
    <div class="welcome-content">
      <div>
        <img src="@/assets/suazo.png" width="100" alt="">
      </div>
       <div>
        <img src="@/assets/rickandmorty.png" width="400" alt="">
      </div>
      <div class="welcome-msg">
        <h2>Bienvenido a Rick and Morty</h2>
        <p>En esta prueba, evaluaremos su capacidad para construir la aplicación mediante el análisis de código y la reproducción del siguiente diseño.</p>
      </div>
      <router-link to="/home">
        <b-button rounded type="is-info is-link" >Continuar</b-button>
      </router-link>
    </div>
     
  </div>

</template>

<script>
   export default {
    name: "Welcome"
    }
</script>

<style scoped>
  .welcome {
    background-image: url('../assets/portal.png');
    background-repeat: no-repeat;
    background-color: black;
    background-size: contain;
    background-position: center;
    position: relative;
    height: 100vh;
    padding: 0;
    margin: 0;
    width: 100%;
    display: flex;
    flex-wrap: wrap;
    justify-content: center;
    align-content: center;
  }
  .welcome .welcome-content {
    position: relative;
    display: column;
    z-index: 999 !important;
  }
  .welcome .welcome-msg {
    margin: 0 40px !important;
    color: white;
  }
  .welcome .welcome-msg h2 {
    font-size: 1.2rem;
    font-weight: 600;
  }
  .welcome .welcome-msg p {
    font-size: .8rem;
    max-width: 400px;
    margin: 20px auto !important;
  }
  .greypath {
    position: absolute;
    display: block;
    left: 0;
    top: 0;
    width: 100%;
    height: 100%;
    background-color: rgba(0, 0, 0, 0.603);
    z-index: 994 !important;
  }
</style>