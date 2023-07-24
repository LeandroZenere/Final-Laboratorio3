<template>
  <div class="header">
    <img src="./assets/CORN.png" class="logo">
    <nav>
          <router-link class= "encabezado" to="/">Registrarse</router-link>
          <router-link class= "encabezado" to="/nuevacompra">Comprar y Vender criptomonedas</router-link> 
          <router-link class= "encabezado" to="/historialdemovimientos">Historial de Movimientos</router-link>
          <router-link class= "encabezado" to="/estadoactual">Estado Actual</router-link> 
          <router-link class= "encabezado" to="/usuarios">Usuarios Registrados</router-link>
        </nav>
</div>
  <router-view />
</template>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
}

nav {
  padding: 0px;
  padding-right: 100px;
  display: flex; 
  align-items: center; 
}

nav a {
  font-weight: bold;
  color: #ffffff;
  margin-right: 10px;
}

nav a.router-link-exact-active {
  color: #E8B174;
}
nav a:last-child {
  margin-right: 0;
}

.header { 
grid-area: header; 
background-color: black;
width: 1366px;
display: flex; 
justify-content: space-between; 
align-items: center; 
}

.logo {
  width: 7%;
  margin-left: 4%;
}

/* .encabezado:active, */
.encabezado:hover {
  transform: scale(1.03);
}
</style>
