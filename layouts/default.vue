<template>
  <v-app id="name">
    <v-main>
      <v-container>
        <nuxt />
        <client-only>
          <vue-confirm-dialog />
        </client-only>
      </v-container>
    </v-main>
  </v-app>
</template>

<script>
</script>

<style>
/* #name{
  margin-top: 1rem;
  margin-left: 1rem;
  color: rgb(40, 11, 119);
  background: color #d7d7d7;
  font-weight: 500;
}
#logout{
  margin-left:75rem;
  inline-size: 1%;
  background-color: rgb(204, 169, 169);
} */
</style>