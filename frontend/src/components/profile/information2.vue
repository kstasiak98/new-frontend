<template>
  <v-container>
    <v-row class="text-center">
      <v-col cols="12">
        <p>Złożono zamówienie</p>
        <br />
      </v-col>
      <v-col>
        <v-btn @click="back()">Powrót</v-btn>
      </v-col>
    </v-row>
  </v-container>
</template>

<script>
export default {
  name: "information2",

  data: () => ({}),
  methods: {
    back() {
      this.$router.push("/");
    }
  }
};
</script>

<style scoped></style>
