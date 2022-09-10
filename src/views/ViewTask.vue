<template>
  <div>
    <div v-if="errorMsg" class="mb-10 p-4 rounded-md bg-light-grey shadow-lg">
        <p class="text-red-500">
            {{ errorMsg }}
        </p>
    </div>

    <div v-if="successMsg" class="mb-10 p-4 rounded-md bg-light-grey shadow-lg">
        <p class="text-red-500">
            {{ successMsg }}
        </p>
    </div>

    <div v-if="dataLoaded">
        <form 
            @submit.prevent="createTask" 
            class="p-8 flex flex-col bg-light-grey rounded-md shadow-lg"
            >
            <h1 class="text-3xl text-at-light-green mb-4">
                Edit task {{ data.title }}
            </h1>
            <div class="flex flex-col mb-2">
                <label for="name" class="mb-1 text-sm text-at-light-green">Name</label>
                <input 
                    type="text"
                    required
                    class="p-2 text-gray-500 focus:outline-none" 
                    id="name"
                    v-model="data.title"
                />
            </div>

            <div class="flex flex-col mb-2">

                <label for="name" class="mb-1 text-sm text-at-light-green">Is completed?</label>
                <p class="onoff">
                    <input type="checkbox"
                    required
                    class="p-2 text-gray-500 focus:outline-none hidden" 
                    id="completed"
                    v-model="data.is_complete"
                    >
                    <label for="completed"></label>
                </p>

            </div>

            <button 
                type="submit" 
                @click="update"
                class="mt-6 py-2 px-6 rounded-sm self-start text-sm text-white bg-at-light-green 
                    duration-200 border-solid border-2 border-transparent hover:border-at-light-green hover:bg-white hover:text-at-light-green"
                >
                Update
            </button>
        </form>
        <div v-if="successMsg" class="mb-10 p-4 rounded-md bg-light-grey shadow-lg bg-orange-200">
            <p class="text-red-500 font-bold">
                {{ errorMsg }}
            </p>
        </div>
    </div>

  </div>
</template>

<script>
import { ref } from "vue";
import { supabase } from '../supabase';
import { useUserStore } from "./../store/user";
import { storeToRefs } from "pinia";
import { useRouter, useRoute } from 'vue-router';



export default {
    name: "view-task",
    setup() {
        const name = ref("");
        const errorMsg = ref(null);
        const successMsg = ref(null);
        const userStore = useUserStore();
        const { user } = storeToRefs(userStore);
        const router = useRouter();
        const route = useRoute();

        const data = ref(null);
        const dataLoaded = ref(null);
        const taskId = route.params.taskId;

        const getData = async() => {
            console.log("getadata")
            try{
                let {data: tasks, error} = await supabase
                    .from('tasks')
                    .select('*')
                    .eq('user_id', user._object.user.id)
                    .eq('id', taskId)
                if (error) throw error;
                data.value = tasks[0];
                console.log("data value")
                console.log(data.value)
                dataLoaded.value = true;
            } catch (error) {
                console.warn(error.message);
            }
        }

        getData();

            // Update Workout
        const update = async () => {
            try {
                const { error } = await supabase
                .from('tasks')
                .update({
                    title: data.value.title,
                    is_complete: data.value.is_complete,
                })
                .eq("id", taskId);
                if (error) throw error;
                edit.value = false;
                successMsg.value = "Success: Workout Updated!";
                setTimeout(() => {
                    successMsg.value = false;
                }, 5000);
            } catch (error) {
                console.warn(error.message);
            }
        };
            
        return { name, dataLoaded, data, errorMsg, successMsg, update };
    }
}
</script>

<style>

</style>