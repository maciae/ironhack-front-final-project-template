<template>
  <div>
    <div v-if="dataLoaded && dataCompleted.length === 0 && dataPending.length === 0" class="container mt-10 px-4">
        <!-- NO DATA -->
        <div class="w-full flex flex-col items-center">
            <h1 class="text-2xl">Looks empty here...</h1>
            <router-link class="text-sm mt-6 text-center" :to="{name: 'Create'}">
                <span class="text-at-light-green">Create a task</span>
            </router-link>
        </div>
    </div>
    <!-- <div v-else class="grid grid-cols-1 sm:grid-cols-2 md:grid-cols-3 lg:grid-cols-4 gap-6">
        <router-link 
            class="flex flex-col items-center bg-light-grey p-8 shadow-md cursor-pointer"
            :to="{ name: ''}"
            v-for="(task, index) in data"
            :key="index"
        >

        <h1>{{task.title}}</h1>
        </router-link>
    </div>
    -->
    <div v-else class="grid gap-x-8 gap-y-4 grid-cols-4">
        <div class="col-span-2">
            <h2>To-Do Tasks</h2>
            <div 
                class="to-do-list [&>a]:bg-sky-200 bg-sky-100 h-full p-5"
                @drop="onDrop($event, 1)"
                @dragenter.prevent
                @dragover.prevent
                >
                <router-link 
                    class="flex flex-col items-center mb-5 bg-light-grey p-5 shadow-md cursor-pointer"
                    :to="{ name: 'ViewTask', params: { taskId: task.id } }"
                    v-for="(task, index) in dataPending"
                    v-bind:id="task.id"
                    :key="index"
                    draggable="true"
                    @dragstart="startDrag($event, task, 1)"
                >
                <h1>{{task.title}}</h1>
                </router-link>
            </div>
        </div>
        <div class="col-span-2">
            <h2>Done Tasks</h2>
            <div 
                class="done-list [&>a]:bg-lime-200 bg-lime-100 h-full p-5"
                @drop="onDrop($event, 2)"
                @dragenter.prevent
                @dragover.prevent
                >
                <router-link
                    class="flex flex-col items-center mb-5 bg-light-grey p-5 shadow-md cursor-pointer"
                    :to="{ name: 'ViewTask', params: { taskId: task.id } }"
                    v-for="(task, index) in dataCompleted"
                    v-bind:id="task.id"
                    :key="index"
                    draggable="true"
                    @dragstart="startDrag($event, task, 2)"
                >
                <h1>{{task.title}}</h1>
                </router-link>
            </div>
        </div>
    </div>
  </div>
</template>

<script>
import { ref } from "vue";
import { supabase } from '../supabase';
import { useUserStore } from "./../store/user";
import { storeToRefs } from "pinia";


export default {
    name: "home",
    setup() {
        const dataCompleted = ref([]);
        const dataPending = ref([]);
        const dataLoaded = ref(null);

        const userStore = useUserStore();
        const { user } = storeToRefs(userStore);

        const getData = async() => {
            console.log("getadata")
            try{
                let {data: tasks, error} = await supabase
                    .from('tasks')
                    .select('*')
                    .eq('user_id', user._object.user.id)
                    .eq('is_complete', true)
                if (error) throw error;
                dataCompleted.value = tasks;
                console.log("tasks")
                console.log(tasks)

                let {data: tasks2, error2} = await supabase
                    .from('tasks')
                    .select('*')
                    .eq('user_id', user._object.user.id)
                    .eq('is_complete', false)
                if (error2) throw error2;
                console.log("tasks2")
                console.log(tasks2)
                dataPending.value = tasks2;
                
                dataLoaded.value = true;
            } catch (error) {
                console.warn(error.message);
            }
        }

        getData();

        const startDrag = (event, item, originList) => {
            console.log(item)
            event.dataTransfer.dropEffect = 'move'
            event.dataTransfer.effectAllowed = 'move'
            event.dataTransfer.setData('itemID', item.id)
            event.dataTransfer.setData('originList', originList)
        }

        const onDrop = (event, list) => {
            const itemID = event.dataTransfer.getData('itemID');
            const originList = event.dataTransfer.getData('originList');
            if (originList != list){
                if (list == 2){ // DONE LIST
                    
                } else { // TODO LIST

                }
            }
            const item = document.getElementById(itemID);
            console.log(list);
            
        }

        return {
            dataCompleted, 
            dataPending, 
            dataLoaded,
            startDrag,
            onDrop
        }

    }
}
</script>

<style>

</style>