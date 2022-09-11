<template>
  <div class="min-h-full font-Poppins box-border">
    <Navigation></Navigation>
    <section class="container overflow-hidden mb-20">
      <router-view class="app-main" />
    </section>
    <Footer></Footer>
  </div>
</template>

<script setup>
import { onMounted } from "vue";
import { storeToRefs } from "pinia";
import { useRouter, useRoute } from "vue-router";
import { useUserStore } from "./store/user.js";
import Navigation from "./components/Navigation.vue";
import Footer from "./components/Footer.vue";

const router = useRouter();

const userStore = useUserStore();
const { user } = storeToRefs(userStore);


onMounted(async () => {

  const isValidRoute = (window.location.pathname === '/register' || window.location.pathname === '/auth')

  try {    
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
