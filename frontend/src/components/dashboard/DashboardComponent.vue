<template>
  <v-app>
    <div class="dashboard-container">
      <!-- Barra de aplicaciones con el ícono de menú para pantallas pequeñas -->
      <v-app-bar class="app-bar">
        <v-app-bar-nav-icon @click="toggleMenu" color="black" />
        <v-toolbar-title class="title">Panel de Control</v-toolbar-title>

        <v-breadcrumbs class="breadcrumbs" :items="breadcrumbs" @click:breadcrumb="goToHome" />

        <!-- Campo de búsqueda -->
        <v-spacer></v-spacer>
        <v-text-field
          v-model="searchQuery"
          append-icon="mdi-magnify"
          label="Buscar..."
          single-line
          hide-details
          @input="onSearch"
          class="search-bar"
        />
        <!-- Mostrar usuario -->
        <v-spacer></v-spacer>
        <div class="user-info" v-if="user">
          <span class="user-name">Bienvenido, {{ user }}</span>
          <img src="https://cdn-icons-png.flaticon.com/512/3135/3135715.png" alt="">
        </div>
      </v-app-bar>

      <!-- Menú lateral con VNavigationDrawer -->
      <v-navigation-drawer
        v-model="isMenuOpen"
        app
        class="menu"
        dark
        temporary
      >
        <v-list class="menu-list">
          <!-- Opciones de menú basadas en el tipo de usuario -->
          <v-list-item v-if="userType === 'admin'" class="menu-item">
            <v-list-item-title @click="switchComponent('Analitics')">
              Analíticas
            </v-list-item-title>
          </v-list-item>
          <v-list-item v-if="userType === 'admin'" class="menu-item">
            <v-list-item-title @click="switchComponent('AddPurchaser')">
              Compradores
            </v-list-item-title>
          </v-list-item>
          <v-list-item v-if="userType === 'admin'" class="menu-item">
            <v-list-item-title @click="switchComponent('AddPet')">
              Mascotas
            </v-list-item-title>
          </v-list-item>
          <v-list-item v-if="userType === 'purchaser'" class="menu-item">
            <v-list-item-title @click="switchComponent('AddPet')">
              Mascotas
            </v-list-item-title>
          </v-list-item>
          <v-list-item v-if="userType === 'admin'" class="menu-item">
            <v-list-item-title @click="switchComponent('AddProvider')">
              Proveedores
            </v-list-item-title>
          </v-list-item>
          <v-list-item v-if="userType === 'provider'" class="menu-item">
            <v-list-item-title @click="switchComponent('AddProduct')">
              Productos
            </v-list-item-title>
          </v-list-item>
          <v-list-item v-if="userType === 'purchaser'" class="menu-item">
            <v-list-item-title @click="switchComponent('WishList')">
              Lista de deseados
            </v-list-item-title>
          </v-list-item>
          <v-list-item @click="signOut" class="menu-item">
            <v-list-item-title>Cerrar sesión</v-list-item-title>
          </v-list-item>
        </v-list>
      </v-navigation-drawer>

      <!-- Contenido principal -->
      <v-main>
        <div class="main-content">
          <component :is="currentComponent" />
        </div>
      </v-main>
    </div>
  </v-app>
</template>

<script>
import { useUserStore } from "@/stores/userStore"; // Imports the user store
import AddPurchaser from "@/components/dashboard/AddPurchaser.vue";
import Analitics from "@/components/dashboard/AnaliticsData.vue";
import AddPet from "@/components/dashboard/AddPet.vue";
import AddProvider from "@/components/dashboard/AddProvider.vue";
import AddProduct from "@/components/dashboard/AddProduct.vue";
import WishList from "@/components/dashboard/WishList.vue";

export default {
  data() {
    return {
      currentComponent: "", // Actual component to display
      isMenuOpen: false,
      searchQuery: "",
      breadcrumbs: [
        { title: "Inicio", disabled: false, to: "/" },
        { title: "Panel de Control", disabled: true },
      ],
    };
  },
  computed: {
    // Access the user type from the user store
    userType() {
      return useUserStore().userType;
    },
    user() {
      return useUserStore().user;
    },
  },
  created() {
    // Configure the initial component based on the user type
    this.setInitialComponent();
  },
  methods: {
    toggleMenu() {
      this.isMenuOpen = !this.isMenuOpen;
    },
    onSearch() {
      console.log("Search query:", this.searchQuery);
    },
    switchComponent(componentName) {
      this.currentComponent = componentName;
    },
    setInitialComponent() {
      if (this.userType === "admin") {
        this.currentComponent = "Analitics";
      } else if (this.userType === "purchaser") {
        this.currentComponent = "AddPet";
      } else if (this.userType === "provider") {
        this.currentComponent = "AddProduct";
      } else {
        this.currentComponent = "";
      }
    },
    signOut() {
      useUserStore().clearUser();
      this.$router.push("/");
    },
  },
  components: {
    AddPurchaser,
    Analitics,
    AddPet,
    AddProvider,
    AddProduct,
    WishList,
  },
};
</script>

<style scoped>
.dashboard-container {
  display: flex;
  flex-direction: column;
  height: 100%;
}

.app-bar {
  background-color: #003459;
  color: white;
}

.title {
  font-size: 24px;
  font-weight: bold;
}

.search-bar {
  max-width: 300px;
}

.menu {
  background-color: #002642;
}

.menu-list .menu-item {
  color: white;
}

.user-info {
  display: flex;
  align-items: center;
  gap: 8px;
}

.user-name {
  font-size: 16px;
}
.v-list-item {
  cursor: pointer;
}

.dashboard-container {
  display: flex;
  height: 100vh;
  background-color: white;
}

.app-bar {
  background-color: #003459;
}

.title {
  color: white;
}

.search-bar {
  background-color: white;
  color: #003459;
}

.user-info {
  display: flex;
  align-items: center;
  gap: 10px;
  color: white;
}

.user-name {
  font-weight: bold;
  color: white;
}

.user-info img {
  width: 30px;
  height: 30px;
  margin-right: 10px;
}

.menu {
  background-color: #003459;
}

.menu-list .menu-item {
  color: white;
}

.menu-list .menu-item:hover {
  background-color: white;
  color: #003459;
}

.main-content {
  padding: 20px;
  flex-grow: 1;
  background-color: white;
}

.v-breadcrumbs {
  color: white;
}
</style>

  