<template>
  <div id="settings-panel">
    <ul>
      <li>
        <span>{{ settingsText.trialTimeTotal }}</span>
        <input type="number" min="4" v-model="newSettings.trialTimeTotal" />
      </li>
      <li>
        <span>{{ settingsText.enterGameDelaySecondsMin }}</span>
        <input
          type="number"
          min="0"
          v-model="newSettings.enterGameDelaySecondsMin"
        />
      </li>
      <li>
        <span>{{ settingsText.enterGameDelaySecondsMax }}</span>
        <input
          type="number"
          min="0"
          v-model="newSettings.enterGameDelaySecondsMax"
        />
      </li>
      <li>
        <span>{{ settingsText.delayEachTurnSecondsMin }}</span>
        <input
          type="number"
          min="1"
          v-model="newSettings.delayEachTurnSecondsMin"
        />
      </li>
      <li>
        <span>{{ settingsText.delayEachTurnSecondsMax }}</span>
        <input
          type="number"
          min="1"
          v-model="newSettings.delayEachTurnSecondsMax"
        />
      </li>
      <li>
        <span>{{ settingsText.shouldAutoPlay }}</span>
        <toggle-switch
          :is-checked="newSettings.shouldAutoPlay"
          @update-click-value="toggleAutoPlay"
        />
      </li>
      <li>
        <span>{{ settingsText.rewarderPossibility }}</span>
        <input
          type="number"
          step="0.1"
          min="0"
          max="1"
          v-model="newSettings.rewarderPossibility"
        />
      </li>
      <li>
        <span>{{ settingsText.punisherPossibility }}</span>
        <input
          type="number"
          step="0.1"
          min="0"
          max="1"
          v-model="newSettings.punisherPossibility"
        />
      </li>
    </ul>
  </div>
</template>

<script>
import { watch, ref, unref } from "vue";
import ToggleSwitch from "./toggleSwitch.vue";

export default {
  name: "settingsMenu",
  components: { ToggleSwitch },
  emits: ["updateSetting"],
  props: {
    currentSetting: {
      type: Object,
    },
    settingsText: {
      type: Object,
      default: {
        trialTimeTotal: Number,
        enterGameDelaySecondsMin: Number,
        enterGameDelaySecondsMax: Number,
        delayEachTurnSecondsMin: Number,
        delayEachTurnSecondsMax: Number,
        shouldAutoPlay: Boolean,
        rewarderPossibility: Number,
        punisherPossibility: Number,
      },
    },
  },
  setup(props, context) {
    const newSettings = ref(props.currentSetting);

    const toggleAutoPlay = (newValue) => {
      newSettings.value.shouldAutoPlay = newValue;
    };

    watch(
      () => newSettings,
      (newSettings) => {
        context.emit("updateSetting", newSettings);
      },
      {
        deep: true,
      }
    );

    return {
      toggleAutoPlay,
      newSettings,
    };
  },
};
</script>

<style scoped>
#settings-panel {
  grid-area: content;
}

li {
  line-height: 2;
  display: flex;
  justify-content: space-between;
  align-items: center;
}

span {
  user-select: none;
}

input {
  max-width: 3rem;
  text-align: center;
  margin-left: 0.5rem;
  background: #f2f2f2;
  border-radius: 0.25rem;
  border: 1px solid #ccc;
}

li:nth-child(even) {
  background-color: #eeeeeee5;
}

li:last-child {
  padding-bottom: 0.3rem;
}

li:first-child {
  padding-top: 0.3rem;
}
</style>
