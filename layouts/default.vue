<template>
  
  <q-layout view="hHh lpR fFf">
    
    <q-header reveal elevated class="bg-primary text-white" height-hint="98">
      <q-toolbar>
        <q-btn dense flat round icon="menu" @click="toggleLeftDrawer" />

        <q-toolbar-title>
          <q-avatar>
            <img src="https://cdn.quasar.dev/logo-v2/svg/logo-mono-white.svg">
          </q-avatar>
          Технологический редактор
        </q-toolbar-title>

        <q-btn dense flat round icon="menu" @click="toggleRightDrawer" />
      </q-toolbar>

      <q-tabs align="left">
        <q-route-tab to="/" label="Index" />
        <q-route-tab to="/about" label="About" />
        <q-route-tab to="/contact" label="Contact" />
      </q-tabs>
    </q-header>
    
    <q-drawer v-model="leftDrawerOpen" side="left" overlay elevated>
      <!-- drawer content -->
       <q-list>
        <q-item-label header>Essential Links</q-item-label>
        <q-item clickable v-ripple to="/">
          <q-item-section avatar>
            <q-icon name="home" />
          </q-item-section>
          <q-item-section>Index</q-item-section>
        </q-item>
        <q-item clickable v-ripple to="/about">
          <q-item-section avatar>
            <q-icon name="info" />
          </q-item-section>
          <q-item-section>About</q-item-section>
        </q-item>
        <q-item clickable v-ripple to="/contact">
          <q-item-section avatar>
            <q-icon name="contact_mail" />
          </q-item-section>
          <q-item-section>Contact</q-item-section>
        </q-item>
        <q-item clickable v-ripple to="/mashines">
          <q-item-section avatar>
            <q-icon name="build" />
          </q-item-section>
          <q-item-section>Mashines</q-item-section>
        </q-item>
        <q-item clickable v-ripple to="/machinetools">
          <q-item-section avatar>
            <q-icon name="build" />
          </q-item-section>
          <q-item-section>Mashine-Tools</q-item-section>
        </q-item>
      </q-list>
    </q-drawer>

    <q-drawer v-model="rightDrawerOpen" side="right" overlay elevated>
      <!-- drawer content -->
    </q-drawer>

    <q-page-container>
      <NuxtPage />
    </q-page-container>


    <CommonFooter />
  </q-layout>

</template>

<script>
import { ref } from 'vue'

export default {
  setup () {
    const leftDrawerOpen = ref(false)
    const rightDrawerOpen = ref(false)

    return {
      leftDrawerOpen,
      toggleLeftDrawer () {
        leftDrawerOpen.value = !leftDrawerOpen.value
      },

      rightDrawerOpen,
      toggleRightDrawer () {
        rightDrawerOpen.value = !rightDrawerOpen.value
      }
    }
  }
}

</script>
