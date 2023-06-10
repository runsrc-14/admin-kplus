<template>
    <p v-for="user in users" :key="user.id">
      ID: {{ user.id }} 👉 {{ user.name }}
    </p>
    <v-btn color="success" varaint="outlined">add</v-btn>
</template>

<script>
definePageMeta({
  layout: "blank",
});

export default {
  data() {
    return {
      users: "",
    };
  },
  async mounted() {
    const useRuntimeConfig = this.$config.public.apiBase;
    this.users = await $fetch(`${useRuntimeConfig}/category/getcategory`).then(
      (res) => {
        return res.data.map((item) => {
          return {
            id: item.ctId,
            name: item.ctName,
          };
        });
      }
    );
  },
};
</script>
