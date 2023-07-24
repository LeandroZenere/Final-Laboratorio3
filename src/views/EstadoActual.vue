<template>
  <div class="layout">
    <h1>Análisis de Inversiones</h1>
      <div v-if="resumenCriptomonedas.length === 0">
        <p>No tienes criptomonedas.</p>
      </div>

      <div v-else>
        <table>
          <thead>
            <tr>
              <th>Criptomoneda</th>
              <th>Compra</th>
              <th>Cantidad comprada</th>
              <th>Venta</th>
              <th>Cantidad vendida</th>
              <th>Resultado</th>
              <th>Criptomonedas Disponibles</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="resumen in resumenCriptomonedas" :key="resumen.idCriptomoneda">
              <td>{{ obtenerNombre(resumen.idCriptomoneda) }}</td>
              <td>{{ formatearNumero(resumen.comprado) }}</td>
              <td>{{ resumen.cantidadComprada }}</td>
              <td>{{ formatearNumero(resumen.vendido) }}</td>
              <td>{{ resumen.cantidadVendida }}</td>
              <td>{{ formatearNumero(resumen.resultado) }}</td>
              <td>{{ resumen.cantidadDisponible }}</td>
            </tr>
          </tbody>
        </table>
              <h3>Resultado total de sus inversiones</h3>
              <p>{{ formatearNumero(resumenTotal) }}</p>
              
      </div>
  </div>
</template>


<script>
import axios from 'axios';

export default {
  data() {
    return {
      resumenCriptomonedas: [],
      resumenTotal: 0,
      criptomonedas: {},
    };
  },

  created() {
    this.obtenerEstadoActual();
  },

  methods: {
    listaCriptomonedas() {
      return [
        { id: 'btc', name: 'Bitcoin' },
        { id: 'eth', name: 'Ethereum' },
        { id: 'usdt', name: 'USDT' },
        { id: 'dai', name: 'DAI' },
      ];
    },

    formatearNumero(num) {
      return Number(num).toLocaleString('es-AR', { style: 'currency', currency: 'ARS' });
    },

    obtenerNombre(id) {
      const crypto = this.listaCriptomonedas().find(c => c.id === id);
      return crypto ? crypto.name : id;
    },

    obtenerEstadoActual() {
      const idDeUsuario = localStorage.getItem('idDeUsuario');
      const url = `https://laboratorio3-5fc7.restdb.io/rest/transactions?q={"user_id":"${idDeUsuario}"}`;
  
      axios.get(url, {
        headers: {
          'x-apikey': '64bdbc3386d8c5613ded91e7',
        },
      }).then((response) => {
        const transacciones = response.data;
        this.criptomonedas = {};

        for (const transaccion of transacciones) {
          const idCriptomoneda = transaccion.crypto_code;

          if (!this.criptomonedas[idCriptomoneda]) {
            this.criptomonedas[idCriptomoneda] = {
              idCriptomoneda: idCriptomoneda,
              comprado: 0,
              vendido: 0,
              cantidadComprada: 0,
              cantidadVendida: 0,
              
            };
          }
          
          if (transaccion.action === "purchase") {
            this.criptomonedas[idCriptomoneda].comprado += parseFloat(transaccion.money);
            this.criptomonedas[idCriptomoneda].cantidadComprada += transaccion.action === "purchase" ? parseFloat(transaccion.crypto_amount) : 0;
            this.criptomonedas[idCriptomoneda].cantidadDisponible = this.criptomonedas[idCriptomoneda].cantidadComprada - this.criptomonedas[idCriptomoneda].cantidadVendida;
          } else if (transaccion.action === "sale") {
            this.criptomonedas[idCriptomoneda].vendido += parseFloat(transaccion.money);
            this.criptomonedas[idCriptomoneda].cantidadVendida += transaccion.action === "sale" ? parseFloat(transaccion.crypto_amount) : 0;
            this.criptomonedas[idCriptomoneda].cantidadDisponible = this.criptomonedas[idCriptomoneda].cantidadComprada - this.criptomonedas[idCriptomoneda].cantidadVendida;
          }
        }

        this.resumenCriptomonedas = Object.values(this.criptomonedas).map(crip => {
          crip.resultado = crip.vendido - crip.comprado;
          this.resumenTotal += crip.resultado;
          return crip;
        });
      });
    },
  },
};
</script>
<style scoped>

table {
  width: 100%;
  border-collapse: collapse;
}

th,
td {
  padding: 8px;
  border: 1px solid #ddd;
}

th {
  background-color: #f2f2f2;
}
</style>