<script setup lang="ts">
import raw_genes from "@/assets/genes.json";
import { computed } from "vue";

interface Entity {
    title: string,
    value: string
}

const selected = defineModel<string[]>("selected")

const genes = computed(() => {
    const res:string[] = []
    raw_genes.forEach((x) => {
        res.push(x[1]);
    }) 
    return res;
});
</script>

<template>
    <div>
    <v-combobox chips multiple clearable label="Gene Entities" :items="genes" v-model="selected">
    </v-combobox>
    </div>
</template>