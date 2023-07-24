<template>
  <div class="layout">
  <div class="MostrarUsuariosRegistrados">
    <h2>Usuarios Registrados:</h2>
    <ul>
      <li v-for="usuario in usuarios" :key="usuario.idDeUsuario">
        {{ usuario.Nombre }} {{ usuario.Apellido }} (ID: {{ usuario.idDeUsuario }})
      </li>
    </ul>
  </div>
</div>
</template>

<script>
import { mapState } from 'vuex';

export default {
  computed: {
    ...mapState(["usuarios"]),
  },
};
</script>