<template>
  <header>
  </header>

  <main>
    <h1>Peers</h1>
    <p>Number of peers: {{ peers.length }}</p>
    <table class="table">
    <thead>
      <tr>
        <th scope="col">ASN</th>
      </tr>
    </thead>
    <tbody>
      <tr v-for="peer in peers">
        <td>{{peer}}</td>
      </tr>
    </tbody>
  </table>
  </main>
</template>

<script setup lang="ts">
  import { ref } from 'vue'
  const peers = ref<string[]>([]);
  fetch('https://clickhouse.nxthdr.dev?user=read&password=read', {
        method: 'POST',
        body: 'SELECT DISTINCT path[1] AS peer FROM nxthdr.bgp_updates WHERE peer != 0 ORDER BY peer FORMAT Json',
      })
      .then(response => response.json())
      .then(response => { response.data.forEach((peer: { peer: string }) => {  peers.value.push(peer.peer) }) });
</script>