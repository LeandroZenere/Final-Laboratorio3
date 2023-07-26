<template>
  <div class = "layout">
    <div class="recuadroLogin">
      <h1>Ingrese su ID alfanumérico de 10 dígitos.</h1>
      <h2>Una vez su sesión sea iniciada, podrá realizar transacciones.</h2>
      <input v-model="idDeUsuario" type="text" placeholder="ID de usuario" @input="verificarId" class="idDeUsuario">
        <p v-if="debeMostrarError" class="alerta">ID no válido. No se puede iniciar sesión.</p>
    </div>
          <div v-if="sesionIniciada" class="recuadroBienvenida">
              <h2>Bienvenido, {{ obtenerNombreApellido() }}</h2>
              <h3>¿Qué operación desea realizar?</h3>
            <div class="seccionComprar">
                <label class="texto">Tipo de operación:</label>
                <select v-model="tipoDeOperacion">
                <option value="comprar">Comprar</option>
                <option value="vender">Vender</option>
                </select>
            </div>
  
            <div class="seccionComprar">
              <label class="texto">Criptomoneda:</label>
              <select v-model="criptomonedaSeleccionada">
              <option v-for="criptomoneda in criptomonedas" :key="criptomoneda.id" :value="criptomoneda.id">{{ criptomoneda.name }}</option>
              </select>
            </div>
            <div class="seccionComprar">
              <label class="texto">Cantidad:</label>
              <input v-model="cantidad" type="number" step="0.1" min="0.000001" @change="calcularMonto">
            </div>
            <div class="seccionComprar">
              <label class="texto">Monto a pagar:</label>
              <input type="number" readonly :value="montoActualizado">         
            </div>
            <div class="centrarBoton">
            <button @click="guardarTransaccion" :disabled="!datosValidos || (tipoDeOperacion === 'vender' && !puedeVender) || bloquearBoton">Guardar Transacción</button>
            </div>
          </div>
      
  
  <!-- Apartado para Mostrar los datos de nuestras compras o ventas. -->
<div class = "mostrarCompra">
      <div v-if="compraRealizada" class="recuadroCompraRealizada">
          <h3>Compra Realizada</h3>
          <p>Fecha y Hora: {{ compraActual.fechaHora }}</p>
          <p>Criptomoneda: {{ compraActual.criptomoneda }}</p>
          <p>Cantidad: {{ compraActual.cantidad }}</p>
          <p>Monto: ${{ compraActual.monto }}</p>
      </div>
      <div v-if="ventaRealizada" class="recuadroCompraRealizada">
        <h3>Venta Realizada</h3>
        <p>Fecha y Hora: {{ ventaActual.fechaHora }}</p>
        <p>Criptomoneda: {{ ventaActual.criptomoneda }}</p>
        <p>Cantidad: {{ ventaActual.cantidad }}</p>
        <p>Monto: ${{ ventaActual.monto }}</p>
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
        idDeUsuario: '',
        debeMostrarError: false,
        criptomonedaSeleccionada: '',
        cantidad: 0,
        fechaHora: '',
        monto: 0,
        precios: {},
  
        compraRealizada: false,
        ventaRealizada: false,
        tipoDeOperacion: 'comprar',
        compraActual: {
          fechaHora: '',
          criptomoneda: '',
          cantidad: '',
          monto: '',
          tipoDeOperacion: 'comprar',
        },
        ventaActual: {
          fechaHora: '',
          criptomoneda: '',
          cantidad: '',
          monto: '',
          tipoDeOperacion: 'vender',
        },
        puedeVender: false,
        bloquearBoton: false,
        contadorCriptomonedas: {},
      };
    },
  
    methods: {
      mounted() {
    // Cargar el contador desde localStorage
    if (localStorage.getItem('contadorCriptomonedas')) {
      this.contadorCriptomonedas = JSON.parse(localStorage.getItem('contadorCriptomonedas'));
    }
  },

  ApiClient() {
        return axios.create({
          baseURL: 'https://laboratorio3-f36a.restdb.io/rest/',
          headers: {
            'x-apikey': '60eb09146661365596af552f',
          },
        });
      },

      obtenerNombreApellido() {
        const usuario = this.usuarios.find((usuario) => usuario.idDeUsuario === this.idDeUsuario);
        return usuario ? `${usuario.Nombre} ${usuario.Apellido}` : '';
      },
  
      calcularMonto() {
        const criptomoneda = this.criptomonedas.find((c) => c.id === this.criptomonedaSeleccionada);
        if (criptomoneda) {
          const url = `https://criptoya.com/api/argenbtc/${criptomoneda.id}/ars/${this.cantidad}`;
          const action = this.tipoDeOperacion === 'comprar' ? 'ask' : 'bid';
  
          axios
            .get(url)
            .then((response) => {
              const precioARS = response.data[action];
              this.precios[criptomoneda.id] = precioARS;
              this.monto = (this.cantidad * precioARS).toFixed(2);
            })
            .catch((error) => {
              console.error('Error al obtener el monto en pesos (ARG): ', error);
            });
        }
      },
  
      verificarId() {
        const idValida = this.usuarios.some((usuario) => usuario.idDeUsuario === this.idDeUsuario);
        const tieneDiezDigitos = this.idDeUsuario.length === 10;
        this.debeMostrarError = !idValida && tieneDiezDigitos;
        if (this.sesionIniciada) {
          this.verificarCantidadCriptomonedas();
        }
      },
  
      guardarTransaccion() {
        const nuevaTransaccion = {
          user_id: this.idDeUsuario,
          action: this.tipoDeOperacion === 'comprar' ? 'purchase' : 'sale',
          crypto_code: this.criptomonedaSeleccionada,
          crypto_amount: this.cantidad.toString(),
          money: this.monto.toString(),
          datetime: this.fechaHoraActual(),
        };
  
        axios
          .post('https://laboratorio3-f36a.restdb.io/rest/transactions', nuevaTransaccion, {
            headers: {
              'x-apikey': '60eb09146661365596af552f',
            },
          })
          .then((response) => {
            console.log('Operación exitosa:', response.data);
  
            if (nuevaTransaccion.action === 'purchase') {
              // Operación de compra
              this.compraActual.fechaHora = this.fechaHoraActual();
              this.compraActual.criptomoneda = this.obtenerNombreCriptomoneda();
              this.compraActual.cantidad = this.cantidad;
              this.compraActual.monto = this.monto;
              this.compraRealizada = true;
              if (!this.contadorCriptomonedas[this.criptomonedaSeleccionada]) {
                this.contadorCriptomonedas[this.criptomonedaSeleccionada] = 0;
              }
              this.contadorCriptomonedas[this.criptomonedaSeleccionada] += this.cantidad;
            } else if (nuevaTransaccion.action === 'sale') {
              // Operación de venta
              this.ventaActual.fechaHora = this.fechaHoraActual();
              this.ventaActual.criptomoneda = this.obtenerNombreCriptomoneda();
              this.ventaActual.cantidad = this.cantidad;
              this.ventaActual.monto = this.monto;
              this.ventaRealizada = true;
  
              //Restar la cantidad vendida de la cantidad comprada para reflejar la venta
              this.contadorCriptomonedas[this.criptomonedaSeleccionada] -= this.cantidad; 
            }
            this.bloquearBoton = true;
            this.verificarCantidadCriptomonedas();
            this.resetearFormulario();
          })
          .catch((error) => {
            console.error('Error al guardar la transacción:', error);
          });
      },


      resetearFormulario() {
        this.criptomonedaSeleccionada = '';
        this.cantidad = 0;
        this.fechaHora = this.fechaHoraActual();
        this.monto = 0;
      },
  
      obtenerNombreCriptomoneda() {
        const criptomoneda = this.criptomonedas.find((c) => c.id === this.criptomonedaSeleccionada);
        return criptomoneda ? criptomoneda.name : '';
      },
  
      fechaHoraActual() {
        const now = new Date();
        const year = now.getFullYear();
        const month = `${now.getMonth() + 1}`.padStart(2, '0');
        const day = `${now.getDate()}`.padStart(2, '0');
        const hour = `${now.getHours()}`.padStart(2, '0');
        const minute = `${now.getMinutes()}`.padStart(2, '0');
        return `${year}-${month}-${day}  ${hour}:${minute}`;
      },
  
      verificarCantidadCriptomonedas() {
    const idDeUsuario = this.idDeUsuario;
    const codigoCriptomoneda = this.criptomonedaSeleccionada;

    const url = `https://laboratorio3-f36a.restdb.io/rest/transactions?q={"user_id":"${idDeUsuario}","crypto_code":"${codigoCriptomoneda}"}`;
    const apiClient = this.ApiClient();
  
    apiClient
      .get(url)
      .then((response) => {
        const movimientosCriptomoneda = response.data;
        let cantidadComprada = 0;
        let cantidadVendida = 0;
        let cantidadDisponible = 0;
  
        for (const movimiento of movimientosCriptomoneda) {
          if (movimiento.action === 'purchase') {
            cantidadComprada += parseFloat(movimiento.crypto_amount);
          } else if (movimiento.action === 'sale') {
            cantidadVendida += parseFloat(movimiento.crypto_amount);
          }
        }
  
        const contadorCriptomonedas = this.contadorCriptomonedas[this.criptomonedaSeleccionada] || 0;
        cantidadComprada += contadorCriptomonedas;
        cantidadDisponible = cantidadComprada - cantidadVendida;
  
        this.puedeVender = cantidadDisponible >= 0 && this.cantidad > 0 && this.cantidad <= cantidadDisponible;
        if (this.tipoDeOperacion === 'comprar') {
          // Para comprar, no hay restricciones, siempre se puede comprar.
          this.puedeVender = true;
        } else if (this.tipoDeOperacion === 'vender' && this.cantidad > this.contadorCriptomonedas[this.criptomonedaSeleccionada]) {
          this.puedeVender = this.cantidad <= cantidadDisponible;
          this.puedeVender ? null : console.log("No puede realizar la venta. No tiene las criptomonedas")
        }

        localStorage.setItem('contadorCriptomonedas', JSON.stringify(this.contadorCriptomonedas));
        this.bloquearBoton = false; // Desbloquear el botón después de la validación
      })
      .catch((error) => {
        console.error('Error al obtener la cantidad de criptomonedas:', error);
      });
  },
},

  
    computed: {
      ...mapState(['usuarios']),
      sesionIniciada() {
        return this.idDeUsuario !== '' && this.esIdValida;
      },
  
      esIdValida() {
        return this.usuarios.some((usuario) => usuario.idDeUsuario === this.idDeUsuario);
      },
  
      criptomonedas() {
        return [
          { id: 'btc', name: 'Bitcoin' },
          { id: 'eth', name: 'Ethereum' },
          { id: 'usdt', name: 'USDT' },
          { id: 'dai', name: 'DAI' },
        ];
      },
  
      datosValidos() {
        return (
          this.criptomonedaSeleccionada !== '' &&
          this.cantidad > 0 &&
          this.monto >= 0
        );
      },
  
      montoActualizado() {
        const criptomoneda = this.criptomonedas.find((c) => c.id === this.criptomonedaSeleccionada);
        if (criptomoneda) {
          const precioARS = this.precios[criptomoneda.id];
          return (this.cantidad * precioARS).toFixed(2);
        } else {
          return 0;
        }
      },
    },
  
    watch: {
      cantidad: {
        handler() {
          this.calcularMonto();
          this.verificarCantidadCriptomonedas();
        },
        immediate: true,
      },
    },
  };
  </script>
  
  <style scoped>
  .alerta {
    color: rgb(245, 9, 9);
    font-size: 18px;
    margin-top: 2px;
    font-weight: bold;
  }
  .recuadroLogin {
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
  .idDeUsuario {
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
  
  .recuadroBienvenida {
    background-color: #e0e0e0;
    border: 1px solid rgb(0, 0, 0);
    width: 80%; 
    height: 80%;
    display: flex;
    flex-direction: column;
    justify-content: flex-start;
    font-family: "Roboto Condensed", sans-serif;
    color: #000000;
    padding: 2%;
    border-radius: 20px;
    grid-area: main;
    margin: 5px;
    position: relative;
  }
  
  .seccionComprar {
    display: flex;
    margin-bottom: 10px;
  }
  
  .seccionComprar .texto{
    margin-right: 10px;
    font-size: 18px;
    width: 15%; 
  }
  
  
  .seccionComprar p {
    margin-right: 10px; 
    font-size: 18px;
    width: 100%;
  }
  
  .seccionComprar input,
  .seccionComprar select {
    flex: 1;
    height: 40px;
    border: 1px solid #ccc;
    border-radius: 5px;
    padding: 5px;
    font-size: 16px;
  }
  
  .centrarBoton {
    display: flex;
    justify-content: center; 
    margin-bottom: 10px;
    height: 60%;
  }
  
  
  button {
    align-items: center;
    appearance: none;
    background-color: #FCFCFD;
    border-radius: 4px;
    border-width: 0;
    box-shadow: rgba(45, 35, 66, 0.4) 0 2px 4px,rgba(45, 35, 66, 0.3) 0 7px 13px -3px,#D6D6E7 0 -3px 0 inset;
    box-sizing: border-box;
    color: #36395A;
    cursor: pointer;
    display: inline-flex;
    font-family: "JetBrains Mono", monospace;
    height: 48px;
    line-height: 1;
    list-style: none;
    overflow: hidden;
    padding-left: 16px;
    padding-right: 16px;
    position: relative;
    text-align: left;
    text-decoration: none;
    transition: box-shadow .15s,transform .15s;
    user-select: none;
    -webkit-user-select: none;
    touch-action: manipulation;
    white-space: nowrap;
    will-change: box-shadow,transform;
    font-size: 18px;
  }
  
  button:focus {
    box-shadow: #D6D6E7 0 0 0 1.5px inset, rgba(45, 35, 66, 0.4) 0 2px 4px, rgba(45, 35, 66, 0.3) 0 7px 13px -3px, #D6D6E7 0 -3px 0 inset;
  }
  
  button:hover {
    box-shadow: rgba(45, 35, 66, 0.4) 0 4px 8px, rgba(45, 35, 66, 0.3) 0 7px 13px -3px, #D6D6E7 0 -3px 0 inset;
    transform: translateY(-2px);
  }
  
  button:active {
    box-shadow: #D6D6E7 0 3px 7px inset;
    transform: translateY(2px);
  }
  .mostrarCompra {
    background-image: linear-gradient(45deg, #A48754, #E8B174, #916d31 );
    height: 80%;
    width: 90%;
    border: 1px inset black;
    border-radius: 5px;
    padding: 5%;
    font-size: 16px;
    margin-top: 50px;
  }
  </style>