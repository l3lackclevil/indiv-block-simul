<template>
  <div class="peer-manager">
    <v-alert :value="alert" type="success" transition="scale-transition">This is a success alert.</v-alert>
    <v-card width="100%" height="auto">
      <v-card-title class="card-title">
        <span class="title">Peer Manager</span>
      </v-card-title>
      <div class="card-content">
        <div class="item">
          <span class="topic">Peer Name :</span>
          <v-chip>
            <v-avatar>
              <v-icon :color="peer.color">account_circle</v-icon>
            </v-avatar>
            Peer {{peer.id}} (Group : {{peer.group}})
          </v-chip>
        </div>
        <v-divider/>
        <div class="item">
          <span class="topic">Connect :</span>
          <div class="select">
            <v-select
              v-model="selected"
              :items="items"
              item-text="text"
              item-value="value"
              label="Peer"
            ></v-select>
          </div>
          <v-btn
            small
            dark
            depressed
            color="#1E88E5"
            v-on:click="connect(peer.id, selected)"
          >Connect</v-btn>
        </div>
        <v-divider/>
        <div class="item">
          <span class="topic">Connected To (Directly):</span>
          <div class="neighbor-chips">
            <v-chip
              close
              v-for="(neighbor_id,idx) in peer.connected"
              :key="idx"
              @input="removeedge(peer.id,neighbor_id)"
            >
              <v-avatar>
                <v-icon :color="peercolor(neighbor_id)">account_circle</v-icon>
              </v-avatar>
              Peer {{neighbor_id}}
            </v-chip>
          </div>
        </div>
        <v-divider/>
        <div class="item">
          <span class="topic">Connected To (Indirect) :</span>
          <div class="neighbor-chips">
            <v-chip v-for="(neighbor_id,idx) in indirect_connected" :key="idx">
              <v-avatar>
                <v-icon :color="peercolor(neighbor_id)">account_circle</v-icon>
              </v-avatar>
              Peer {{neighbor_id}}
            </v-chip>
          </div>
        </div>
        <v-divider/>
        <div class="item">
          <span class="topic">Usable/Balance :</span>
          {{peer.usable}}/{{peer.balance}}
        </div>
        <v-divider/>

        <div class="item-block">
          <div class="block-panel">
            <span class="topic">Spend :</span>
            <v-text-field placeholder="Input money" v-model="peer.input" :change="store_money()"/>
            <span class="to">To</span>
            <div class="select">
              <v-select
                v-model="to_peer_id"
                :items="items"
                item-text="text"
                item-value="value"
                label="Peer"
              ></v-select>
            </div>
            <v-btn
              small
              dark
              depressed
              :color="peer.color"
              v-on:click="addblock(peer.id,peer.input,to_peer_id)"
            >Spend</v-btn>
          </div>
          <div class="blocks">
            <div
              class="block"
              v-for="(block, idx) of peer.blocks"
              :key="idx"
              :style="{'border-color' : block.color}"
            >
              BlockID : {{block.blockid}}
              <br>
              From : Peer {{block.from}}
              <br>
              To : Peer {{block.to}}
              <br>
              Value : {{block.value}}
            </div>
          </div>
        </div>
        <v-divider/>
      </div>
    </v-card>
  </div>
</template>
<script>
import _ from 'lodash'
export default {
  name: 'PeerManager',
  props: {
    peer: Object,
    peers: Array,
    connect: Function,
    removeedge: Function,
    addblock: Function,
    group_stake: Array,
    total_peer: Number
  },
  data() {
    return { selected: null, alert: false, to_peer_id: 0 }
  },
  computed: {
    items: function() {
      const filtered = this.peers
        .filter(peer => peer.id !== this.peer.id && _.findIndex(this.peer.connected, id => id === peer.id) == -1)
        .map(peer => ({ text: `Peer ${peer.id}`, value: peer.id }))
      return filtered
    },
    indirect_connected: function() {
      const peers = this.peers.filter(peer => {
        const isSameGroup = peer.group === this.peer.group
        let isDirected = false
        for (const peer_id of this.peer.connected) {
          if (peer_id == peer.id) isDirected = true
        }
        return isSameGroup && !isDirected && peer.id !== this.peer.id
      })
      return peers.map(peer => peer.id)
    }
  },
  methods: {
    peercolor: function(peer_id) {
      const peer_idx = this.peers.findIndex(obj => obj.id === peer_id)
      return this.peers[peer_idx].color || 'black'
    },
    store_money: function() {
      const input = this.peer.input || 0
      this.peer.input = parseInt(input)
    }
  }
}
</script>
<style scoped>
.peer-manager {
  width: 950px;
  text-align: center;
}
.item {
  display: flex;
  align-items: center;
  width: 100%;
  text-align: left;
  padding-left: 10px;
  margin-top: 10px;
  margin-bottom: 10px;
}
.topic {
  font-size: 16px;
  font-weight: 600;
  margin-right: 20px;
  width: 130px;
}
.select {
  width: 350px;
}
.margin-bot {
  margin-bottom: 10px;
}
.item-block {
  display: flex;
  flex-direction: column;
  align-items: flex-start;
  width: 100%;
  padding-left: 10px;
  margin-top: 10px;
  margin-bottom: 10px;
}
.item-block .topic {
  margin-bottom: 10px;
}
.blocks {
  width: 100%;
  display: flex;
  flex-wrap: wrap;
  margin-top: 15px;
}
.block {
  width: 120px;
  height: auto;
  border: solid;
  margin-right: 5px;
  margin-bottom: 10px;
  padding: 10px 5px 5px 5px;
}
.block-panel {
  display: flex;
  align-items: center;
  justify-content: space-around;
  width: 600px;
}
.to {
  margin-left: 10px;
  margin-right: 20px;
}
</style>
