<template>
  <div class="layout">
    <div class="titulo">
      <h1>Para continuar, debe registrarse en Corn.</h1>
      <h2>Completando el siguiente formulario se generará una ID única, la cual utilizará para realizar las transacciones.</h2>
    </div>
  <div class="registrarUsuario">
    <input v-model="Nombre" type="text" placeholder="Nombre" @input="validarCampoPorLetras('Nombre')">
    <p v-if="!esCampoValido('Nombre')" class="mensaje-validacion">Solo debe contener letras.</p>
    <input v-model="Apellido" type="text" placeholder="Apellido" @input="validarCampoPorLetras('Apellido')">
    <p v-if="!esCampoValido('Apellido')" class="mensaje-validacion">Solo debe contener letras.</p>
    <button @click="generarIdDeUsuario" :disabled="idCopiada || !ValidacionNombre" :class="{ 'copiado': idCopiada }" >
      {{ idCopiada ? '✔' : 'Generar ID de usuario' }}
    </button>
    <p v-if="idDeUsuario">ID de usuario generado: {{ idDeUsuario }}</p> 
  </div>
</div>

</template>

<script>
import { mapMutations, mapState } from 'vuex';
//Para dar acceso a los usuarios, usamos MapState
export default {
  data() {
    return {
      Nombre: "",
      Apellido: "",
      idDeUsuario: "",
      idCopiada: false, //Indicará si la ID fue copiada o no.
      idsGeneradas: new Set(), //Con esto validamos no repetir 2 veces un mismo ID.
      camposValidos: {
      Nombre: true,
      Apellido: true,
      },
    };
  },

  methods: {
    validarCampoPorLetras(campoAValidar) { //Utilizamos para verificar que el nombre/apellido sea exclusivamente texto.
      const texto = this[campoAValidar].trim();
      const valoresValidos = /^[A-Za-z]+$/;
      this.camposValidos[campoAValidar] = texto !== '' && valoresValidos.test(texto);
    },

    esCampoValido(campo) {
      return this.camposValidos[campo];
    },

    generarIdRandom() {
      const caracteres = "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789¡?!¿";
      let idGenerada = "";
      for (let i = 0; i < 10; i++) {
        idGenerada += caracteres.charAt(Math.floor(Math.random() * caracteres.length));
      }
      return idGenerada;
    },

    generarIdUnica() {
      let idNueva = this.generarIdRandom();
      while (this.idsGeneradas.has(idNueva)) {
        idNueva = this.generarIdRandom();
      }
      return idNueva;
    },

    generarIdDeUsuario() {
  const idNueva = this.generarIdUnica();
  localStorage.setItem('idDeUsuario', idNueva);
  this.idsGeneradas.add(idNueva);
  this.idDeUsuario = idNueva;
  const nuevoUsuario = {
    Nombre: this.Nombre,
    Apellido: this.Apellido,
    idDeUsuario: this.idDeUsuario,
  };
  this.copiarId();
  this.idCopiada = true;
  this.agregarUsuario(nuevoUsuario);
  
},

    copiarId() {
  const idACopiar = this.idDeUsuario;
  navigator.clipboard.writeText(idACopiar) //Uso de API propia de los navegadores (navigator) > Usamos la propiedad clipboard y el método ya integrado writeText para copiar el ID.
    .catch((error) => {
      console.error('Error al copiar el ID:', error);
    });
},
    ...mapMutations(['agregarUsuario']), //Llamamos al método creado en Store.
    
  },
  computed: {
    ...mapState(["usuarios"]),
    ValidacionNombre() { //Corroboramos que no hayan quedado espacios en blanco y que el nombre solo contenga letras.
      return this.esCampoValido('Nombre') && this.esCampoValido('Apellido') && this.Nombre.trim() !== '' && this.Apellido.trim() !== '';
    },
  },
};
</script>

<style>
.layout {
  width: 1366px;
  height: 768px;
  display: grid;
  grid:
    "header" auto
    "main" 1fr
    / 1fr;
  gap: 0px;
  justify-items: center;
  align-items: center;
  background-image: linear-gradient(45deg, #A48754, #E8B174, #916d31 );
}
</style>
<style scoped>
.mensaje-validacion {
  color: rgb(245, 9, 9);
  font-size: 18px;
  margin-top: 2px;
  font-weight: bold;
}
.registrarUsuario{
width: 67%;
height: 70%;
padding: 0%;
display: flex;
flex-direction: column; 
align-items: center;
border-radius: 15px;
grid-area: main;
}

button {
  width: 20%; 
  height: 15%;
  border-radius: 5%;
  margin-top: 10px;
  font-size: 14px;
  line-height: 1; 
  cursor: pointer; /* Cambia el cursor al pasar sobre el botón */
  transition: transform 0.3s ease;
  color: black;
}

button:active,
button:hover {
  transform: scale(1.1);
}

button.copiado {
  background-color: rgb(0, 255, 136);
  font-size: 24px;
  color: white;
  width: 8%; 
  height: 12%;
  border-radius: 10px;
  border: 1px solid black;
}

input {
  outline: none;
  width: 70%; 
  height: 10%;
  margin-bottom: 15px;
  border-radius: 15px;
  appearance: none;
  border-style: none;
  font-family: "Montserrat", sans-serif;
  font-size: 15px;
  padding: 1%;
  border: 2px solid transparent; /* Agrega un borde transparente */
  box-shadow: 0 0 0 2px rgb(255, 123, 0); /* Inicialmente, establece un borde negro */
  animation: borderAnimation 2.8s linear infinite; /* Agrega la animación al borde */
}

@keyframes borderAnimation {
  0%, 100% {
    box-shadow: 0 0 0 3px rgb(255, 235, 57); /* Borde negro */
  }
  50% {
    box-shadow: 0 0 0 3px transparent, 0 0 0 6px rgb(255, 123, 0); /* Borde negro más grande */
  }
}
.titulo {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  font-family: "Roboto Condensed", sans-serif;
  font-size: 100%;
  color: #000000;
  padding-top: 3%;
  width: 60%;
  border-radius: 20px;
  grid-area: header;
}
p {
  color: rgb(0, 0, 0);
  display: flex;
  justify-content: center;
}

</style>