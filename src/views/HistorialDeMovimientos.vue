<template>
<div class = "layout">
  <div>
    <h2>Historial de Movimientos</h2>
        <div v-if="movimientos.length === 0">
          <p>No se encontraron movimientos.</p>
        </div>
        <div v-else>
        <table>
          <thead>
            <tr>
              <th>Fecha y Hora</th>
              <th>Criptomoneda</th>
              <th>Cantidad</th>
              <th>Monto</th>
              <th>Tipo</th>
              <th>Acciones</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="movimiento in movimientos" :key="movimiento._id">
              <td>{{ movimiento.datetime }}</td>
              <td>{{ movimiento.crypto_code }}</td>
              <td>{{ movimiento.crypto_amount }}</td>
              <td>{{ movimiento.money }}</td>
              <td>{{ movimiento.action === 'purchase' ? 'Compra' : 'Venta' }}</td>
              <td>
                <button @click="abrirFormularioEdicion(movimiento)">Editar</button>
                <button @click="borrarMovimiento(movimiento._id)">Borrar</button>
              </td>
            </tr>
          </tbody>
        </table>
      </div>
    </div>
  <div class="otrousuario">
    <input v-model="nuevaId" type="text" placeholder="Ingrese ID de usuario" @input="verificarId">
    <p v-if="debeMostrarError" class="alerta">ID inválido o sin registros por mostrar.</p>
    <button @click="verificarMovimientosRegistrados">Buscar</button>
  </div>

  <div v-if="mostrandoEdicionDeFormulario" class="formularioFlotante">
    <h3>Editar Movimiento</h3>
    <div>
      <label>Dinero Gastado: </label>
      <input v-model="nuevoMonto" type="number" step="0.01">
    </div>
    <div>
      <label>Cantidad de Criptomoneda: </label>
      <input v-model="edicionDelMovimiento.crypto_amount" type="number" step="0.0001">
    </div>
    <div>
      <button @click="guardarEdicionMovimiento">Guardar</button>
      <button @click="cancelarEdicionMovimiento">Cancelar</button>
    </div>
  </div>
</div>
</template>

<script>
import axios from 'axios';
import { mapState } from 'vuex';
export default {
  data() {
    return {
      movimientos: [],
      nuevaId: '',
      debeMostrarError: false,
      mostrandoEdicionDeFormulario: false,
      edicionDelMovimiento: {
        money: '',
        crypto_amount: '',
      },
      id: '',
      nuevoMonto: '',
    };
  },


  created() {
    const idDeUsuario = localStorage.getItem('idDeUsuario');
    if (idDeUsuario) {
      this.verificarMovimientosRegistrados(idDeUsuario);
    }
  },


  computed: {
    ...mapState(['usuarios']),
  },


  methods: {
    
    ApiClient() {
        return axios.create({
          baseURL: 'https://laboratorio3-f36a.restdb.io/rest/',
          headers: {
            'x-apikey': '60eb09146661365596af552f',
          },
        });
      },

    verificarMovimientosRegistrados() {
      const idDeUsuario = this.nuevaId || localStorage.getItem('idDeUsuario');
      const url = `https://laboratorio3-f36a.restdb.io/rest/transactions?q={"user_id":"${idDeUsuario}"}`;
      const apiClient = this.ApiClient();

      apiClient
        .get(url)
        .then(response => {
          this.movimientos = response.data;
          if (this.movimientos.length === 0) {
            console.log('No se encontraron movimientos registrados.');
          } else {
            console.log('Se encontraron movimientos registrados.');
          }
        })
        .catch(error => {
          console.error('Error al obtener los movimientos:', error);
        });
    },

    abrirFormularioEdicion(movimiento) {
      console.log("Formulario de edición abierto")
      this.mostrandoEdicionDeFormulario = true;
      this.edicionDelMovimiento = {...movimiento};
      this.nuevoMonto
    },

    async guardarEdicionMovimiento() {
      if (!this.edicionDelMovimiento || !this.nuevoMonto) return;

      const apiClient = this.ApiClient();
        try {
          console.log(this.edicionDelMovimiento._id);
    const response = await apiClient.patch(`/transactions/${this.edicionDelMovimiento._id}`, {
      money: this.nuevoMonto,
      crypto_amount: this.edicionDelMovimiento.crypto_amount,
    });
    console.log("Movimiento editado y guardado")
    console.log(response);

    this.verificarMovimientosRegistrados();
    this.cancelarEdicionMovimiento();
  } catch (error) {
    console.error("Error al guardar la edición del movimiento: ", error);
  }
},

cancelarEdicionMovimiento() {
  this.mostrandoEdicionDeFormulario = false;
  this.edicionDelMovimiento = null;
  this.nuevoMonto = '';
  console.log("Formulario cerrado")
},

  async borrarMovimiento(id) {
    try {
      const apiClient = this.ApiClient();
      if (!apiClient) return;

      const response = await apiClient.delete(`/transactions/${id}`);
      console.log("Movimiento borrado");  
      this.movimientos = this.movimientos.filter(movimiento => movimiento._id !== id); //Actualizamos el HTML removiendo automáticamente los borrados. 
      return response.data;


    } catch (error) {
      console.error("Error al borrar el movimiento", error);
    }
  },

    ocultarEdicionDeFormulario() {
      this.mostrandoEdicionDeFormulario = false; // Ocultamos el formulario de editarMovimiento
    },
    
  },
};
</script>

<style scoped>
.alerta {
  color: red;
  font-weight: bold;
}

table {
  width: 100%;
  border-collapse: collapse;
}

th,
td {
  padding: 8px;
  border-bottom: 1px solid #ddd;
}

th {
  background-color: #f2f2f2;
}

.formularioFlotante {
  position: fixed;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  background-color: #1f0052;
  padding: 20px;
  border: 1px solid #00ccff;
  border-radius: 4px;
  color: white;
}

.otrousuario input {
  outline: none;
  width: 100%; 
  height: 15%;
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

.otrousuario button {
  width: 80%; 
  height: 60%;
  border-radius: 5%;
  margin-top: 10px;
  font-size: 14px;
  line-height: 1; 
  cursor: pointer; /* Cambia el cursor al pasar sobre el botón */
  transition: transform 0.3s ease;
  color: black;
}

.otrousuario button:active,
.otrousuario button:hover {
  transform: scale(1.1);
}
</style>

