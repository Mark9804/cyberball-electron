<template>
  <div
    class="person-placeholder"
    :class="clickEnabled ? 'click-enabled' : 'click-disabled'"
  >
    <img class="avatar" src="../assets/player.png" alt="player avatar" />
    <div class="player-name" style="display: contents">
      <p>{{ obfuscatePlayerName }}</p>
      <div class="player-indicator" v-if="'playerSelf' === playerId">You</div>
    </div>
  </div>
</template>

<script>
export default {
  name: "playerEntity",
  props: {
    playerName: String,
    playerId: String,
    clickEnabled: Boolean,
  },
  computed: {
    obfuscatePlayerName() {
      return `${this.playerName}_${Math.random().toString(36).substring(2, 8)}`;
    },
  },
};
</script>

<style>
.person-placeholder {
  display: grid;
  grid-template-areas:
    "avatar"
    "name";
  place-items: center;
}

.avatar {
  grid-area: avatar;
  width: 64px;
  height: 64px;
}

.player-indicator {
  font-weight: bold;
  color: #4fc08d;
  z-index: 2;
}

.player-name {
  font-size: 1.2rem;
  font-weight: bold;
  text-align: center;
  user-select: none;
}

.click-enabled {
  cursor: pointer;
}

.click-disabled {
  cursor: not-allowed;
}
</style>
