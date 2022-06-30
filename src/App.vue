<script setup>
import { ref, unref, watch } from "vue";
import ConfirmButton from "./components/confirmButton.vue";
import WelcomeScreen from "./components/welcomeScreen.vue";
import WaitingScreen from "./components/waitingScreen.vue";
import GameScreen from "./components/gameScreen.vue";
import SettingsMenu from "./components/settingsMenu.vue";
import LangSelectionMenu from "./components/langSelectionMenu.vue";
import langString from "./assets/i18n/translation.js";

const correctSettingsPassword = "123456";

const showWelcome = ref(true);
const showNameEntrance = ref(false);
const participantName = ref("");
const showWaiting = ref(false);
const showGameScreen = ref(false);
const showSettingsMenu = ref(false);
const showLanguageMenu = ref(false);

const locale = ref("en");
const lang = ref(langString.filter((lang) => lang.locale === locale.value)[0]);

const gameSettings = ref({
  trialTimeTotal: 400,
  enterGameDelaySecondsMin: 2,
  enterGameDelaySecondsMax: 5,
  shouldAutoPlay: false,
  delayEachTurnSecondsMin: 1,
  delayEachTurnSecondsMax: 2,
  rewarderPossibility: 0.7,
  punisherPossibility: 0.1,
});

const updateSettings = (newSettingsObject) => {
  const newSettings = unref(newSettingsObject);
  gameSettings.value = {
    trialTimeTotal:
      newSettings.trialTimeTotal >= 4 ? newSettings.trialTimeTotal : 4,
    enterGameDelaySecondsMin:
      newSettings.enterGameDelaySecondsMin >= 0
        ? newSettings.enterGameDelaySecondsMin
        : 0,
    enterGameDelaySecondsMax:
      newSettings.enterGameDelaySecondsMax >= 0
        ? newSettings.enterGameDelaySecondsMax
        : 0,
    shouldAutoPlay: newSettings.shouldAutoPlay,
    delayEachTurnSecondsMin:
      newSettings.delayEachTurnSecondsMin >= 1
        ? newSettings.delayEachTurnSecondsMin
        : 1,
    delayEachTurnSecondsMax:
      newSettings.delayEachTurnSecondsMax >= 1
        ? newSettings.delayEachTurnSecondsMax
        : 1,
    rewarderPossibility:
      newSettings.rewarderPossibility >= 0 &&
      newSettings.rewarderPossibility <= 1
        ? newSettings.rewarderPossibility
        : 0.7,
    punisherPossibility:
      newSettings.punisherPossibility >= 0 &&
      newSettings.punisherPossibility <= 1
        ? newSettings.punisherPossibility
        : 0.1,
  };
};

const showNameEntranceScreen = () => {
  showWelcome.value = false;
  showNameEntrance.value = true;
  showSettingsMenu.value = false;
  showLanguageMenu.value = false;
};

/**
 * @function enterReadyState
 * @description Continue to the name input screen
 */
const enterReadyState = () => {
  const minWaitingSeconds = gameSettings.value.delayEachTurnSecondsMin ?? 2;
  const maxWaitingSeconds = gameSettings.value.delayEachTurnSecondsMax ?? 5;
  showSettingsMenu.value = false;
  showLanguageMenu.value = false;

  const randomTimeout =
    Math.floor(
      Math.random() * Math.abs(maxWaitingSeconds - minWaitingSeconds) * 1000
    ) +
    minWaitingSeconds * 1000;
  showNameEntrance.value = false;
  showWaiting.value = true;
  if ("" === participantName.value) {
    participantName.value = "Player";
  }

  setTimeout(() => {
    showWaiting.value = false;
    showGameScreen.value = true;
  }, randomTimeout);
};

const namePlaceholder = ref("Player");

/**
 * @function resetOverallStatus
 * @description Called after previous game has ended and return button pressed.
 *            Reset all status to initial state.
 */
const resetOverallStatus = () => {
  showWelcome.value = true;
  showNameEntrance.value = false;
  showWaiting.value = false;
  showGameScreen.value = false;
  participantName.value = "";
  settingsPassword.value = "";
  showSettingsMenu.value = false;
  showLanguageMenu.value = false;
};

const settingsPassword = ref("");

const toggleSettingsMenuScreen = () => {
  showSettingsMenu.value = !showSettingsMenu.value;
  settingsPassword.value = "";
  showLanguageMenu.value = false;
};

const toggleLanguageMenuScreen = () => {
  showSettingsMenu.value = false;
  showLanguageMenu.value = !showLanguageMenu.value;
};

const updateLanguage = (newLanguage) => {
  toggleLanguageMenuScreen();
  locale.value = unref(newLanguage.locale);
  lang.value = unref(newLanguage);
};

const clickEventHandler = (e) => {
  if (showSettingsMenu.value && (e.clientX < 517 || e.clientY > 356)) {
    toggleSettingsMenuScreen();
  } else if (showLanguageMenu.value && (e.clientX < 660 || e.clientY > 162)) {
    toggleLanguageMenuScreen();
  }
};

watch(
  () => [showSettingsMenu.value, showLanguageMenu.value],
  (newValue) => {
    if (newValue[0] || newValue[1]) {
      window.addEventListener("click", clickEventHandler);
    } else {
      window.removeEventListener("click", clickEventHandler, {
        capture: false,
      });
    }
  }
);
</script>

<template>
  <div id="container-settings">
    <div id="container-icons">
      <svg
        id="settings-icon"
        @click="toggleSettingsMenuScreen"
        v-if="showWelcome"
        :class="showSettingsMenu ? 'active' : ''"
        viewBox="0 0 512 512"
      >
        <path
          d="M465.6,187.4c-6.1-0.6-12.2-2-17.9-4.4c-31.9-13-47.2-49.5-34.1-81.4c4.7-12.2,0.9-26-9.3-34.1
	c-19.8-15.7-41.9-28.1-65.6-36.8c-12-4.5-25.5-0.7-33.4,9.3c-3.1,4-6.7,7.6-10.7,10.7c-11.5,9-25,13.3-38.5,13.3
	c-13.5,0-27.1-4.3-38.5-13.3c-4-3.1-7.6-6.7-10.7-10.7c-8-10-21.5-13.7-33.4-9.3c-23.7,8.7-45.8,21.1-65.6,36.8
	c-10.2,8.1-14,21.9-9.3,34.1c13,31.9-2.2,68.3-34.1,81.4c-5.7,2.3-11.7,3.8-17.9,4.4c-12.9,1.4-23.5,11.1-25.9,23.8
	C17.5,225.9,16,241,16,256s1.5,30.1,4.5,44.8c2.5,12.8,13,22.5,25.9,23.8c6.1,0.6,12.2,2,17.9,4.4c31.9,13,47.2,49.5,34.1,81.4
	c-4.7,12.2-0.9,26,9.3,34.1c19.8,15.7,41.9,28.1,65.6,36.8c12,4.5,25.5,0.7,33.4-9.3c3.1-4,6.7-7.6,10.7-10.7
	c11.5-9,25-13.3,38.5-13.3c13.5,0,27.1,4.3,38.5,13.3c4,3.1,7.6,6.7,10.7,10.7c8,10,21.5,13.7,33.4,9.3
	c23.7-8.7,45.8-21.1,65.6-36.8c10.2-8.1,14-21.9,9.3-34.1c-13-31.9,2.2-68.3,34.1-81.4c5.7-2.3,11.7-3.8,17.9-4.4
	c12.9-1.4,23.5-11.1,25.9-23.8c3-14.7,4.5-29.8,4.5-44.8s-1.5-30.1-4.5-44.8C489.1,198.4,478.5,188.7,465.6,187.4z M256,336L256,336
	L256,336c-44.2,0-80-35.8-80-80s35.8-80,80-80h0h0c44.2,0,80,35.8,80,80S300.2,336,256,336z"
        />
      </svg>
      <svg
        id="languages-icon"
        @click="toggleLanguageMenuScreen"
        viewBox="0 0 512 512"
      >
        <g>
          <g>
            <g>
              <polygon
                points="138.71,137 132.29,137 120.293,197 150.707,197 		"
              />
            </g>
          </g>
          <g>
            <g>
              <path
                d="M381.374,257c6.477,17.399,15.092,31.483,24.626,43.467c9.534-11.984,19.149-26.069,25.626-43.467H381.374z"
              />
            </g>
          </g>
          <g>
            <g>
              <path
                d="M467,91H280.717l38.842,311.679c0.687,12.748-2.798,24.75-11.118,34.146L242.663,512H467c24.814,0,45-20.186,45-45V137
			C512,112.186,491.814,91,467,91z M467,257h-4.006c-8.535,27.383-22.07,48.81-36.136,65.702
			c11.019,10.074,22.802,18.338,34.517,27.594c6.46,5.171,7.515,14.604,2.329,21.079c-5.162,6.465-14.632,7.513-21.079,2.329
			c-12.729-10.047-24.677-18.457-36.625-29.421c-11.948,10.964-22.896,19.374-35.625,29.421c-6.447,5.184-15.917,4.136-21.079-2.329
			c-5.186-6.475-4.131-15.908,2.329-21.079c11.715-9.256,22.498-17.52,33.517-27.594c-14.066-16.891-26.602-38.318-35.136-65.702
			H346c-8.291,0-15-6.709-15-15s6.709-15,15-15h45v-15c0-8.291,6.709-15,15-15c8.291,0,15,6.709,15,15v15h46c8.291,0,15,6.709,15,15
			S475.291,257,467,257z"
              />
            </g>
          </g>
          <g>
            <g>
              <path
                d="M244.164,39.419C241.366,16.948,222.162,0,199.516,0H45C20.186,0,0,20.186,0,45v332c0,24.814,20.186,45,45,45
			c89.67,0,154.177,0,236.551,0c4.376-5.002,8.044-8.134,8.199-14.663C289.788,405.7,244.367,41.043,244.164,39.419z
			 M183.944,286.707c-7.954,1.637-16.011-3.527-17.651-11.763L156.706,227h-42.411l-9.587,47.944
			c-1.611,8.115-9.434,13.447-17.651,11.763c-8.115-1.626-13.389-9.521-11.763-17.651l29.999-150
			C106.699,112.054,112.852,107,120,107h31c7.148,0,13.301,5.054,14.707,12.056l30,150
			C197.333,277.186,192.06,285.081,183.944,286.707z"
              />
            </g>
          </g>
          <g>
            <g>
              <path
                d="M175.261,452l2.574,20.581c1.716,13.783,10.874,27.838,25.938,34.856c28.428-31.294,11.229-12.362,50.359-55.437H175.261z
			"
              />
            </g>
          </g>
        </g>
      </svg>
    </div>
    <transition name="menu-animation">
      <div
        id="container-settings-panel"
        v-if="showLanguageMenu || showSettingsMenu"
      >
        <lang-selection-menu
          v-if="showLanguageMenu"
          :available-lang="langString"
          @updateLang="updateLanguage"
        />
        <div
          id="password-protection"
          v-if="
            settingsPassword !== correctSettingsPassword && showSettingsMenu
          "
        >
          <svg id="lock" viewBox="0 0 512 512">
            <g>
              <g>
                <path
                  d="M400,188h-36.037v-82.23c0-58.322-48.449-105.77-108-105.77c-59.551,0-108,47.448-108,105.77V188H112
			c-33.084,0-60,26.916-60,60v204c0,33.084,26.916,60,60,60h288c33.084,0,60-26.916,60-60V248C460,214.916,433.084,188,400,188z
			 M187.963,105.77c0-36.266,30.505-65.77,68-65.77s68,29.504,68,65.77V188h-136V105.77z M420,452c0,11.028-8.972,20-20,20H112
			c-11.028,0-20-8.972-20-20V248c0-11.028,8.972-20,20-20h288c11.028,0,20,8.972,20,20V452z"
                />
              </g>
              <g>
                <path
                  d="M256,286c-20.435,0-37,16.565-37,37c0,13.048,6.76,24.51,16.963,31.098V398c0,11.045,8.954,20,20,20
			c11.045,0,20-8.955,20-20v-43.855C286.207,347.565,293,336.08,293,323C293,302.565,276.435,286,256,286z"
                />
              </g>
            </g>
          </svg>
          <h3 style="margin-bottom: 1rem">Password Required</h3>
          <input
            id="password"
            v-focus
            type="password"
            v-model="settingsPassword"
          />
        </div>
        <settings-menu
          v-show="showSettingsMenu"
          :settings-text="lang.strings.settings"
          :current-setting="gameSettings"
          @updateSetting="updateSettings"
        />
      </div>
    </transition>
  </div>
  <div class="container-center" v-if="showWelcome">
    <welcome-screen
      :title-string="lang.title"
      :briefing-string="lang.strings.welcomeText"
    />
    <confirm-button
      @click="showNameEntranceScreen"
      :text="lang.strings.start"
      :type="'confirm'"
    />
  </div>

  <div class="container-center" v-if="showNameEntrance">
    <h1>{{ lang.strings.inputName }}</h1>
    <p v-html="lang.strings.inputNameHint"></p>
    <input
      type="text"
      id="input-name"
      v-focus
      v-model="participantName"
      :placeholder="namePlaceholder"
      @focus="namePlaceholder = ''"
      @blur="namePlaceholder = 'Player'"
      @keydown.enter="enterReadyState"
    />
    <confirm-button
      @click="enterReadyState"
      :text="lang.strings.next"
      :type="'confirm'"
    />
  </div>

  <div class="container-center" v-if="showWaiting">
    <waiting-screen
      :participant-name="participantName"
      :welcome-string="lang.strings.welcome"
      :matching-string="lang.strings.matching"
    />
  </div>

  <div class="container-center" v-if="showGameScreen">
    <game-screen
      :participant-name="participantName"
      :language-string="lang.strings"
      :game-settings="gameSettings"
      @gameOver="resetOverallStatus"
    />
  </div>
</template>

<style>
#container-settings {
  position: absolute;
  top: 1rem;
  right: 1rem;
  display: flex;
  flex-direction: column;
  justify-content: flex-end;
  transition: all 0.375s ease-in-out;
}

#container-icons {
  display: flex;
  flex-direction: row-reverse;
  justify-content: flex-start;
  align-items: center;
}

#settings-icon {
  width: 24px;
  height: 24px;
  fill: #56bd8f;
  margin-bottom: 0.5rem;
  cursor: pointer;
  transition: all 0.375s ease-in-out;
  margin-left: 0.5rem;
}

#languages-icon {
  width: 22px;
  height: 22px;
  fill: #56bd8f;
  margin-bottom: 0.5rem;
  cursor: pointer;
  transition: all 0.375s ease-in-out;
}

#settings-icon.active {
  transform: rotate(90deg);
}

#password-protection {
  grid-area: content;
  width: 100%;
  height: 100%;
  background: #ffffff;
  z-index: 2;
  border-radius: 10px;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
}

#lock {
  width: 1.5rem;
  height: 1.5rem;
  fill: #304050;
  margin-bottom: 0.5rem;
}

input#password {
  text-align: center;
  border-radius: 0.25rem;
  border: 1px solid #cccccc;
}

#container-settings-panel {
  position: relative;
  min-width: 15vw;
  border-radius: 10px;
  background: #ffffffe5;
  backdrop-filter: blur(2px);
  border: solid 0.1px #ccc;
  box-shadow: 0 4px 12px #ccc;
  display: grid;
  grid-template-areas: "content";
  transition: all 0.375s ease-in-out;
  overflow: hidden;
}

#input-name {
  width: 100%;
  padding: 0.5rem;
  border-radius: 10px;
  border: 1px solid #ccc;
  outline: 2px solid transparent;
  margin-top: 60px;
  text-align: center;
  box-sizing: border-box;
  transition: all 0.2s ease-out;
  color: hsl(0deg 0% 13%);
  font-size: 1.5rem;
  margin-bottom: 60px;
  background: transparent;
  backdrop-filter: brightness(0.9)
    drop-shadow(0px 0px 10px hsla(0, 0%, 0%, 0.5));
}

input:focus {
  outline: solid 2px #4fc08d90;
  border: 1px solid transparent;
}

.menu-animation-enter-active,
.menu-animation-leave-active {
  transition: all 0.375s ease;
  transform-origin: top right;
}

.menu-animation-enter-active,
.menu-animation-leave-to {
  opacity: 0;
  transform: translateY(-5%) scale(0.8);
}

.menu-animation-enter-to {
  opacity: 1;
  transform: translateY(0%) scale(1);
}
</style>
