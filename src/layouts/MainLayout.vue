<template>
  <q-layout view="lHh Lpr lFf">
    <q-header elevated>
      <q-toolbar>
        <q-toolbar-title>
          健保特約醫療院所名冊
        </q-toolbar-title>

        <div>Quasar v{{ $q.version }}</div>
        <q-btn
          round
          flat
          icon="info"
          @click="about"
        />
      </q-toolbar>
    </q-header>

    <q-page-container>
      <router-view />
    </q-page-container>
  </q-layout>
</template>

<script>
export default {
  name: 'MainLayout',
  data () {
    return {
      leftDrawerOpen: false
    }
  },
  methods: {
    about () {
      this.$q.dialog({
        title: '關於',
        message: '有任何建議和問題，都歡迎和我聯繫 :)'
      })
    }
  }
}
</script>
