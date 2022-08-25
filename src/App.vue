<template>
  <div class="min-h-full font-Poppins box-border">
    <Navigation></Navigation>
    <section class="container">
      
      <router-view class="app-main" />
    </section>
  </div>
</template>

<script setup>
import { onMounted } from "vue";
import { storeToRefs } from "pinia";
import { useRouter, useRoute } from "vue-router";
import { useUserStore } from "./store/user.js";
import Navigation from "./components/Navigation.vue";

const router = useRouter();
const route = useRoute();

const userStore = useUserStore();
const { user } = storeToRefs(userStore);


onMounted(async () => {

  const isValidRoute = (window.location.pathname === '/register' || window.location.pathname === '/auth')

  try {
    console.log(window.location.pathname)

  console.log(router.currentRoute.value)
    await userStore.fetchUser(); // here we call fetch user
    
    if (!user.value && !isValidRoute) {
      // redirect them to logout if the user is not there
      router.push({ path: "/auth" });
    } else if (user.value) {
      // continue to dashboard
      console.log("user loged")
      console.log(user);
      router.push({ path: "/" });
    }
  } catch (e) {
    console.log(e);
  }
});
</script>
