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
        <td>{{ countryFlag(peer.country) }}</td>
        <td>{{ peer.asn }}</td>
        <td>{{ peer.asname }}</td>
      </tr>
    </tbody>
  </table>
  </main>
</template>

<script setup lang="ts">
  import { ref, computed } from 'vue'
  interface Peer {
    asn: string;
    asname: string;
    country: string;
  }

  function countryFlag(code: string): string {
    if (!code || code.length !== 2) return '';
    return String.fromCodePoint(...[...code.toUpperCase()].map(c => 0x1F1E6 + c.charCodeAt(0) - 65));
  }
  const peers = ref<Peer[]>([]);
  const filterInput = ref<string>('');

  fetch('https://clickhouse.nxthdr.dev?user=read&password=read', {
        method: 'POST',
        body: `
          WITH as_path[1] as asn
          SELECT
            asn,
            dictGet('ipinfo.asn_to_name', 'as_name', asn) AS asname,
            any(country_asn(peer_addr, 'country')) AS country
          FROM bmp.updates
          WHERE asn != 0 AND time_received_ns >= subtractHours(now(), 24)
          GROUP BY asn
          ORDER BY asn
          FORMAT Json
        `,
      })
      .then(response => response.json())
      .then(response => { response.data.forEach((peer: { asn: string, asname: string, country: string }) => {
        peers.value.push(peer) })
      });

  const filteredPeers = computed(() => {
    return peers.value.filter((peer) => {
      if (!filterInput.value) return true;
      return peer.asn == filterInput.value || peer.asname.toLowerCase().includes(filterInput.value.toLowerCase());
    } )
  })
</script>
