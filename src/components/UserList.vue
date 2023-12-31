<template>
  <div class="user-list">
    <h2>User List</h2>
    <div class="search-container">
      <input v-model="searchText" @input="filterUsers" @keydown.enter="searchUsers"
        placeholder="Buscar por nombre o correo electrónico" />
      <button @click="searchUsers">Search</button>
    </div>

    <div v-if="error" class="error-message">{{ error }}</div>

    <div v-if="loading" class="spinner"></div>
    <ul v-else-if="displayedUsers.length > 0">
      <li v-for="user in displayedUsers" :key="user.id">
        <router-link :to="'/user/' + user.id">
          <div class="user-item">
            <img :src="user.avatar" :alt="user.first_name" />
            <div>
              <h3>{{ user.first_name }} {{ user.last_name }}</h3>
              <p>{{ user.email }}</p>
            </div>
          </div>
        </router-link>
      </li>
    </ul>
    <div v-else-if="searchText.length > 0">
      <p>There are not results</p>
    </div>
    <div v-else>
      <p>Enter a search term to find users.</p>
    </div>
  </div>
</template>
  
<script>
import { defineComponent, onMounted, ref, computed } from 'vue';
import { useUserStore } from '@/store';

export default defineComponent({
  name: 'UserList',
  setup() {
    const userStore = useUserStore();
    const loading = ref(true);
    const userList = ref([]);
    const searchText = ref('');
    const requestError = ref(null);

    onMounted(async () => {
      try {
        await userStore.fetchUsers();
        setTimeout(() => {
          userList.value = userStore.userList;
          loading.value = false;
        }, 500);
      } catch (error) {
        console.error('Error fetching users:', error);
        loading.value = false;
        requestError.value = error.message;
      }
    });

    // Filter users by name or e-mail
    const filteredUsers = computed(() => {
      const query = searchText.value.toLowerCase();
      return userList.value.filter(
        (user) =>
          user.first_name.toLowerCase().includes(query) ||
          user.last_name.toLowerCase().includes(query) ||
          user.email.toLowerCase().includes(query)
      );
    });

    // Function to filter users when clicking on the search button
    const filterUsers = () => {
      if (searchText.value) {
        userList.value = filteredUsers.value;
      } else {
        userList.value = userStore.userList;
      }
    };

    
    const searchUsers = () => {
      if (searchText.value) {
        filterUsers();
      } else {
        userList.value = [];
      }
    };

    // Shw users based on search text
    const displayedUsers = computed(() => {
      if (searchText.value) {
        return filteredUsers.value.length > 0 ? filteredUsers.value : [];
      } else {
        return userList.value;
      }
    });

    return {
      loading,
      searchText,
      displayedUsers,
      filterUsers,
      searchUsers,
    };
  },
});
</script>
  
<style scoped>
.user-list {
  margin: 20px;
}

.user-item {
  display: flex;
  align-items: center;
  margin: 10px 0;
  padding: 10px;
  border: 1px solid #ccc;
  border-radius: 5px;
}

.user-item img {
  width: 50px;
  height: 50px;
  margin-right: 10px;
  border-radius: 50%;
}

.spinner {
  border: 4px solid rgba(255, 255, 255, 0.3);
  border-radius: 50%;
  border-top: 4px solid #3498db;
  width: 30px;
  height: 30px;
  animation: spin 2s linear infinite;
  margin: 20px auto;
}

@keyframes spin {
  0% {
    transform: rotate(0deg);
  }

  100% {
    transform: rotate(360deg);
  }
}
</style>
