<script setup lang="ts">
import { computed, ref } from "vue";
import GenePicker from "./GenePicker.vue";
import { asyncComputed } from "@vueuse/core";

const secret = ref<string>("");

const genes = ref<string[]>([]);

function hex(xs: number[]): string {
  return xs.map((b) => b.toString(16).padStart(2, "0")).join("");
}

async function hashOne(txt: string): Promise<string> {
  const data = new TextEncoder().encode(txt);
  const hashBuffer = await window.crypto.subtle.digest("SHA-256", data);
  const hashArray = Array.from(new Uint8Array(hashBuffer)); // convert buffer to byte array
  return hex(hashArray);
}

async function computeHashes(
  salt: string,
  items: string[],
  n: number,
  fuzz: number
): Promise<string[]> {
  const res: string[] = [];
  for (const item of items) {
    for (let i = 0; i < n; ++i) {
      const txt = `${salt} ${item} ${i}`;
      const hash = await hashOne(txt);
      res.push(hash);
    }
  }
  const m = items.length * fuzz;
  for (let j = 0; j < m; ++j) {
    const bytes = new Uint8Array(256);
    window.crypto.getRandomValues(bytes);
    const item = hex(Array.from(bytes));
    for (let i = 0; i < n; ++i) {
      const txt = `${salt} ${item} ${i}`;
      const hash = await hashOne(txt);
      res.push(hash);
    }
  }
  res.sort();
  return res;
}

const hashes = asyncComputed<string[]>(() => {
  return computeHashes(secret.value, genes.value, 2, 1.0);
});
</script>

<style>
.mono {
  font-family: monospace;
}
</style>

<template>
  <v-container class="fill-height">
    <v-responsive class="align-centerfill-height mx-auto" max-width="900">
      <v-card>
      <v-card-title>Parameters</v-card-title>
      <v-card-text>
        <v-text-field label="Secret" v-model="secret"></v-text-field>
        <GenePicker v-model:selected="genes"></GenePicker>
      </v-card-text>
      </v-card>
      <v-card>
      <v-card-title>Results</v-card-title>
      <v-card-text>
        <div class="mono" v-for="hash in hashes" :key="hash">{{ hash }}</div>
      </v-card-text>
    </v-card>
    </v-responsive>
  </v-container>
</template>
