<script setup lang="ts">
    import { Users } from '../../core/Clients/Users';
    import { Tasks } from '../../core/Clients/Tasks';
    import { Ref,onMounted,ref } from 'vue';

    const client = new Users();
    const taskClient = new Tasks();
    const assistants :Ref<any> = ref();
    const tasks : Ref<Array<any>> = ref([])
    const statuts = ['en attente','fini','annulé'];
    const task :Ref<any> = ref({})

    const getTasks = async () =>{
        tasks.value = await taskClient.getAll()
    }

    const addTask = async () =>{
        await taskClient.add(task.value)
        await getTasks()
        task.value = {}
    }
	
	const updateTask = async (value) =>{
		await taskClient.update(value)
        await getTasks()
        task.value = {}
	}
		

    onMounted(async ()=>{
        assistants.value = await client.getAll()
        await getTasks()
    })
</script>
<template>
    <div>
        <div class="my-4">
            <el-form label-position="top">
                <el-row :gutter="10">
                    <el-col :span="6">
                        <el-form-item label="Libellé"> 
                            <el-input v-model="task.libelle" />
                        </el-form-item>
                    </el-col>
                    <el-col :span="6">
                        <el-form-item label="Date d'echeance"> 
                            <el-date-picker
                                v-model="task.expiration_date"
                                type="date"
                                placeholder="Cliquez pour selectionner"
                                format="DD/MM/YYYY"
                                value-format="DD/MM/YYYY"
                                class="w-full"
                                style="width:100vw"
                            />
                        </el-form-item>
                    </el-col>
                    <el-col :span="6">
                        <el-form-item label="Asistant"> 
                            <el-select v-model="task.user_id" class="w-full">
                                <el-option v-for="a in assistants" :label="a.name" :value="a.id" />
                            </el-select>
                        </el-form-item>
                    </el-col>
                    <el-col :span="6">
                        <el-form-item label="&nbsp"> 
                            <button class="btn background-clickdoc btn-sm btn-block" type="button" @click="addTask()" > Enregistrer </button>
                        </el-form-item>
                    </el-col>
                </el-row>
            </el-form>
        </div>
        <el-table :data="tasks">
            <el-table-column prop="libelle" label="Libellé" />
            <el-table-column prop="assistant" label="Assistant" />
            <el-table-column prop="expiration_date" label="Date d'expiration" />
            <el-table-column >
                <template #default="scope">
                    <el-select class="w-full" v-model="scope.row.status" @change(scope.row)>
                        <el-option v-for="status in statuts" :label="status" :value="status" />
                    </el-select>
                </template>
            </el-table-column>
        </el-table>
    </div>
</template>