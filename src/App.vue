<template>
  <main class="container py-4">
    <div class="mb-4">
      <h1 class="mb-3">Vue.js Example</h1>
      <div class="mb-3">
        <button class="btn btn-primary me-2" @click="fetchUsers">Fetch Users</button>
        <select v-model="sortOrder" class="form-select d-inline-block w-auto me-2">
          <option value="asc">Trier: Plus jeune au plus âgé</option>
          <option value="desc">Trier: Plus âgé au plus jeune</option>
        </select>
        <select v-model="selectedGender" class="form-select d-inline-block w-auto me-2">
          <option value="all">Tous</option>
          <option value="male">Homme</option>
          <option value="female">Femme</option>
        </select>
        <input type="text" v-model="searchQuery" class="form-control d-inline-block w-auto" placeholder="Rechercher..." />
      </div>
      <p class="text-muted">Nombre d'utilisateurs affichés : {{ filteredUsers.length }}</p>
    </div>
    <div class="table-responsive">
      <table class="table table-striped table-bordered">
        <thead class="table-dark">
          <tr>
            <th>Photo</th>
            <th>Nom</th>
            <th>Email</th>
            <th>Tel</th>
            <th>Âge</th>
            <th>Genre</th>
          </tr>
        </thead>
        <tbody>
          <tr v-if="!filteredUsers.length">
            <td colspan="6" class="text-center">Appuyez sur "Fetch Users" pour afficher les utilisateurs</td>
          </tr>
          <tr v-for="user in filteredUsers" :key="user.email">
            <td><img :src="user.picture.thumbnail" :alt="user.name.first" class="rounded-circle" /></td>
            <td>{{ user.name.first }} {{ user.name.last }}</td>
            <td>{{ user.email }}</td>
            <td>{{ user.phone }}</td>
            <td>{{ user.dob.age }}</td>
            <td>{{ user.gender === 'male' ? '♂️' : '♀️' }}</td>
          </tr>
        </tbody>
      </table>
    </div>
  </main>
</template>

<script setup>
import { ref, computed } from 'vue';

const users = ref([]), searchQuery = ref(''), selectedGender = ref('all'), sortOrder = ref('asc');

const fetchUsers = async () => {
  try {
    const response = await fetch('https://randomuser.me/api/?results=20');
    users.value = (await response.json()).results;
  } catch (error) {
    console.error('Erreur lors de la récupération des utilisateurs :', error);
  }
};

const normalizeString = str => str.normalize('NFD').replace(/[̀-ͯ]/g, '').replace(/\s+/g, '').toLowerCase();

const filteredUsers = computed(() => {
  let filtered = users.value;
  if (selectedGender.value !== 'all') filtered = filtered.filter(user => user.gender === selectedGender.value);
  if (searchQuery.value) {
    const query = normalizeString(searchQuery.value.trim());
    filtered = filtered.filter(user =>
      normalizeString(user.name.first).includes(query) ||
      normalizeString(user.name.last).includes(query) ||
      normalizeString(user.email).includes(query) ||
      normalizeString(user.phone).includes(query)
    );
  }
  return filtered.sort((a, b) => sortOrder.value === 'asc' ? a.dob.age - b.dob.age : b.dob.age - a.dob.age);
});
</script>

<style>
main { padding: 2rem; }
table img { width: 50px; height: 50px; object-fit: cover; }
</style>