<template>
  <v-container>
    <v-row>
      <v-col cols="12" md="6">
        <h2 class="title">{{ originalProvider ? 'Editar Proveedor' : 'Registrar Proveedor' }}</h2>
        <v-form @submit.prevent="handleSubmit" ref="providerForm" v-model="isValid">
          <v-text-field
            label="Nombre"
            v-model="provider.name"
            required
          ></v-text-field>
          <v-text-field
            label="Usuario"
            v-model="provider.user"
            required
          ></v-text-field>
          <v-text-field
            label="Contraseña"
            v-model="provider.password"
            type="password"
            required
          ></v-text-field>
          <v-text-field
            label="Correo Electrónico"
            v-model="provider.email"
            required
            :rules="[rules.email]"
          ></v-text-field>
          <v-text-field
            label="Teléfono"
            v-model="provider.phone"
            required
            :rules="[rules.phone]"
          ></v-text-field>
          <v-btn color="primary" type="submit">{{ originalProvider ? 'Actualizar' : 'Registrar' }}</v-btn>
          <v-btn color="error" @click="$refs.providerForm.reset()">Resetear</v-btn>
        </v-form>
      </v-col>

      <v-col cols="12" md="6">
        <h2 class="title">Lista de Proveedores</h2>
        <v-data-table
          :headers="headers"
          :items="providers"
          item-value="id"
          class="elevation-1"
          dense
        >
          <template v-slot:top>
            <v-toolbar flat>
              <v-toolbar-title>Proveedores</v-toolbar-title>
              <v-spacer></v-spacer>
            </v-toolbar>
            <v-text-field
              v-model="search"
              class="pa-2 search-field"
              label="Buscar..."
              hide-details
              append-icon="mdi-magnify"
              solo
            ></v-text-field>
          </template>
          <!-- eslint-disable-next-line vue/valid-v-slot -->
          <template #item.actions="{ item }">
            <v-btn icon color="primary" @click="editProvider(item)">
              <v-icon>mdi-pencil-outline</v-icon>
            </v-btn>
            <v-btn icon color="error" @click="deleteProvider(item)">
              <v-icon>mdi-trash-can-outline</v-icon>
            </v-btn>
          </template>
        </v-data-table>
      </v-col>
    </v-row>
  </v-container>
</template>

<script>
export default {
  data() {
    return {
      isValid: false,
      provider: {
        name: '',
        user: '',
        password: '',
        email: '',
        phone: '',
        products: [],
      },
      search: '',
      providers: [],
      headers: [
        { title: 'Nombre', key: 'name' },
        { title: 'Usuario', key: 'user' },
        { title: 'Correo Electrónico', key: 'email' },
        { title: 'Teléfono', key: 'phone' },
        { title: 'Acciones', key: 'actions', sortable: false },
      ],
      rules: {
        email: [(v) => /.+@.+\..+/.test(v) || 'El correo no es válido'],
        phone: [
          (v) =>
            /^[6-9]\d{8}$/.test(v) || 'El número de teléfono no es válido para España',
        ],
      },
      originalProvider: null,
    };
  },
  methods: {
    async handleSubmit() {
      const providerData = { ...this.provider };
      delete providerData._id; 
      delete providerData.__v;
      try {
        if (this.originalProvider) {
          if (this.isProviderModified()) {
            const route = `http://127.0.0.1:3000/providers/${this.originalProvider._id}`;
            const response = await fetch(route, {
              method: 'PUT',
              headers: {
                'Content-Type': 'application/json',
              },
              body: JSON.stringify(providerData),
            });

            if (response.ok) {
              const updatedProvider = await response.json();
              const index = this.providers.findIndex((p) => p._id === updatedProvider._id);
              if (index !== -1) {
                this.providers.splice(index, 1, updatedProvider);
              }
              console.log('Proveedor actualizado con éxito');
              this.resetForm();
            }
          } else {
            console.log('No se hicieron cambios.');
          }
        } else {
          const route = 'http://127.0.0.1:3000/providers';
          const response = await fetch(route, {
            method: 'POST',
            headers: {
              'Content-Type': 'application/json',
            },
            body: JSON.stringify(providerData),
          });

          if (response.ok) {
            console.log('Proveedor registrado con éxito.');
            this.fetchProviders(); 
            this.$refs.providerForm.reset(); 
          } else {
            const errorData = await response.json();
            console.log('Error al registrar el proveedor:', errorData.message);
          }
        }
      } catch (error) {
        console.log('Error:', error);
      }
    },

    async fetchProviders() {
      try {
        const route = 'http://127.0.0.1:3000/providers';
        const response = await fetch(route);
        if (response.ok) {
          this.providers = await response.json();
        }
      } catch (error) {
        console.log('Error al obtener la lista de proveedores.', error);
      }
    },

    editProvider(provider) {
      this.provider = { ...provider };
      this.originalProvider = { ...provider }; 
    },

    async deleteProvider(provider) {
      try {
        const route = `http://127.0.0.1:3000/providers/${provider._id}`;
        const response = await fetch(route, {
          method: 'DELETE',
        });

        if (response.ok) {
          console.log('Proveedor eliminado con éxito');
          this.fetchProviders();
        }
      } catch (error) {
        console.log('Error al eliminar el proveedor', error);
      }
    },

    isProviderModified() {
      return JSON.stringify(this.provider) !== JSON.stringify(this.originalProvider);
    },

    resetForm() {
      this.provider = {
        name: '',
        user: '',
        password: '',
        email: '',
        phone: '',
      };
      this.originalProvider = null; 
    },
  },
  mounted() {
    this.fetchProviders();
  },
};
</script>

<style scoped>
.provider-form {
  max-width: 600px;
  margin: 0 auto;
}
.title {
  color: #003459;
}

.v-data-table {
  background-color: white;
}
.v-toolbar {
  color: white;
  background-color: #003459;
}
</style>
