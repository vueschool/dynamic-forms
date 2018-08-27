<template>
  <div id="app" class="container mx-auto py-6">
    <header>
      <div>
        <span id="logo">Liquid Gold Box</span>
      </div>
    </header>

    <div class="content">
      <FormWizard/>
    </div>
  </div>
</template>

<script>
import FormWizard from './components/FormWizard.vue'

export default {
  name: 'app',
  components: {
    FormWizard
  }
}
</script>