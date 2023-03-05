<template>
  <q-layout view="lHh Lpr lFf">
    <q-header elevated>
      <div class="text-grey">
        <div class="row no-wrap shadow-1">
          <q-toolbar class="col-8 bg-grey-3">
            <q-btn flat round dense icon="menu" @click="toggleLeftDrawer" />
            <q-toolbar-title>Draggable App</q-toolbar-title>
            <q-btn flat round dense icon="search" />
          </q-toolbar>
          <q-toolbar class="col-4 bg-primary text-white">
            <q-space />
            <q-btn flat round dense icon="mdi-axe" class="q-mr-sm" to="/create" />
            <q-btn flat round dense icon="mdi-anvil" class="q-mr-sm" />
            <q-btn flat round dense icon="more_vert" />
          </q-toolbar>
        </div>
      </div>
    </q-header>

    <q-drawer v-model="leftDrawerOpen" bordered>
      <q-list>
        <q-item-label header> Essential Links </q-item-label>
      </q-list>
    </q-drawer>

    <q-page-container>
      <router-view />
    </q-page-container>
  </q-layout>
</template>

<script>
export default {
  data() {
    return {
      leftDrawerOpen: false,
    };
  },
  methods: {
    toggleLeftDrawer() {
      this.leftDrawerOpen = !this.leftDrawerOpen;
    },
  },
};
</script>
