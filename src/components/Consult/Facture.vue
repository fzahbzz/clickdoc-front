<script setup lang="ts">
import { onBeforeMount, ref,Ref } from 'vue';

import { useConsultStore } from '../../../core/Data/stores/consultation';
import { Facture } from '../../../core/Clients/Facture';
import { Article } from '../../../core/Clients/Article';
import { ActeMedical } from '../../../core/Clients/ActeMedical';

const consult = useConsultStore();

const factureClient = new Facture()
const articleClient = new Article()
const acteClient = new ActeMedical()

const actes :Ref<any> = ref([])

const facture : Ref<any> = ref({
    facture:{},
    doctor_fee:[],
    liste:[],
    autres:[]
})

const article : Ref<any> = ref({
    facture_id:consult.facture_id,
    libelle:"",
    prix:0,
    type:0
})

async function getFacture() {
    facture.value=await factureClient.facture(consult.consult)
}

async function generateFacture() {
    facture.value=await factureClient.add({consultation_id:consult.consult})
}

async function saveFacture() {
    facture.value.facture.amount=total();
    facture.value.facture = await factureClient.update(facture.value.facture)
}

async function addArticle() {
    if(article.value.id==undefined)
    {
        await articleClient.add(article.value)
    }
    else
    {
        await articleClient.update(article.value)
    }
    article.value={
        facture_id:consult.facture_id,
        libelle:"",
        prix:0,
        type:1
    }
    await getFacture();
}
async function removeArticle(x:any) {
    await articleClient.delete(x)
    await getFacture();
}
async function getActes(){
    actes.value = await acteClient.getAll()
}

function total(){
    var t = 0;
    for (var i =0;i< facture.value.liste.length;i++ )
    {
        t=t+parseFloat(facture.value.liste[i].prix)
    }

    return t;
}

async function getPrix (){
    const acte =  await acteClient.getByID(article.value.acte);
    article.value.prix = acte.prix
    article.value.libelle = acte.libelle
}

onBeforeMount(async()=>{
    if(consult.edit==false)
    {
        await getFacture()
    }
    await getActes() 
})

</script>

<template>
    <div class="container">
        <div v-if="facture.facture!=undefined  && facture.facture!=null && facture.facture!=0 ">
            <el-form label-position="top">
                <el-row :gutter="10">
                    <el-col :span="16">
                        <el-form-item label="libellé">
                            <el-select v-model="article.acte" class="w-full" @change="getPrix()">
                                <el-option v-for="a in actes" :key="a.id"
                                    :value="a.id"
                                    :label="a.libelle"
                                />
                            </el-select>
                        </el-form-item>
                    </el-col>
                    <el-col :span="4">
                        <el-form-item label="Prix">
                            <el-input v-model="article.prix" />
                        </el-form-item>
                    </el-col>
                    <el-col :span="4">
                        <el-form-item label="&nbsp">
                            <button class="btn btn-sm btn-block background-clickdoc" type="button" @click="async()=>{await addArticle()}"> Ajouter </button>
                        </el-form-item>
                    </el-col>
                </el-row>
            </el-form>
            <br>
            <el-table :data="facture.liste" >
                <el-table-column prop="libelle" label=" Libellé" />
                <el-table-column prop="prix" label=" Prix" width="150px" />
                <el-table-column width="100px">
                    <template #default="scope">
                        <el-button class="btn btn-sm btn-link background-clickdoc" type="button" @click="()=>{article=scope.row;facture.liste.splice(scope.index,-1)}" ><el-icon><Edit/></el-icon></el-button>
                        <el-button class="btn btn-sm btn-link btn-danger" type="button" @click="async ()=>{ await removeArticle(scope.row.id) }"><el-icon><Delete/></el-icon></el-button>
                    </template>
                </el-table-column>
            </el-table>
            <br>
            <div class="text-right">
                total : {{ total() }}
            </div>
            <hr class="my-3">
            <div class="text-right">
                <el-button class="btn btn-sm btn-block background-clickdoc" type="button" @click="async()=>await saveFacture()">
                    Enregistrer la facture 
                </el-button>
				<a class="btn btn-sm btn-block background-clickdoc" target="_blank" :href="'https://api-cd.clickdoc.ma/cnss/'+consult.consult "> Telecharger la fiche AMO CNSS  </a>
            </div>
        </div>
        <div class="text-center" v-else>
            <el-button class="btn btn-sm btn-block background-clickdoc" type="button" @click="async ()=> await generateFacture()">
                Générer Honoraires
            </el-button>
        </div>
    </div>
</template>