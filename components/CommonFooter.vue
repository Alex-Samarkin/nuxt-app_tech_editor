<script setup lang="ts"></script>

<template>
  <div>
    <q-footer reveal elevated class="bg-primary text-white">
      <q-toolbar>
        <q-toolbar-title shrink>
          Технологический редактор
        </q-toolbar-title>
        <q-tabs>
          <q-route-tab to="/" label="Index" />
          <q-route-tab to="/about" label="About" />
          <q-route-tab to="/contact" label="Contact" />
        </q-tabs>
      </q-toolbar>
    </q-footer>
  </div>
</template>

<style scoped></style>
