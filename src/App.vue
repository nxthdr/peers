<template>
  <header>
  </header>

  <main>
    <h1>Peers</h1>
    <p>Number of peers: {{ peers.length }}</p>
    <input v-model="filterInput" placeholder="Filter peers" />
    <p><br /></p>
    <table class="table">
    <tbody>
      <tr v-for="peer in filteredPeers">
        <td>{{ peer }}</td>
      </tr>
    </tbody>
  </table>
  </main>
</template>

<script setup lang="ts">
  import { ref, computed } from 'vue'
  const peers = ref<string[]>([]);
  const filterInput = ref<string>('');

  fetch('https://clickhouse.nxthdr.dev?user=read&password=read', {
        method: 'POST',
        body: `
          SELECT DISTINCT path[1] AS peer
          FROM nxthdr.bgp_updates
          WHERE peer != 0 AND timestamp >= subtractHours(now(), 24)
          ORDER BY peer
          FORMAT Json
        `,
      })
      .then(response => response.json())
      .then(response => { response.data.forEach((peer: { peer: string }) => {  peers.value.push(peer.peer) }) });

  const filteredPeers = computed(() => {
    return peers.value.filter((peer) => {
      if (!filterInput.value) return true;
      return peer == filterInput.value;
    } )
  })
</script>