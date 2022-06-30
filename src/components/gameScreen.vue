<script>
import { ref, unref, watch } from "vue";
import moment from "moment-timezone";
import PlayerEntity from "./playerEntity.vue";
import lodash from "lodash";
import ConfirmButton from "./confirmButton.vue";

export default {
  name: "gameScreen",
  components: {
    PlayerEntity,
    ConfirmButton,
  },
  props: {
    participantName: String,
    languageString: Object,
    gameSettings: Object,
  },
  emits: ["gameOver"],
  setup(props, context) {
    console.log(props.gameSettings);
    const minWaitingSeconds = props.gameSettings.delayEachTurnSecondsMin;
    const maxWaitingSeconds = props.gameSettings.delayEachTurnSecondsMax;
    const maxTrialTime = props.gameSettings.trialTimeTotal;
    const allocationOccursEach = Math.floor(maxTrialTime / 4) ?? 100;

    console.log("minWaitingSeconds: " + minWaitingSeconds);
    console.log("maxWaitingSeconds: " + maxWaitingSeconds);
    console.log("maxTrialTime: " + maxTrialTime);
    console.log("allocationOccursEach: " + allocationOccursEach);

    /**
     * @constant autoPlay
     * @type {boolean}
     * @default false
     * @description enable autoplay (playerSelf will automatically pass the ball
     * to the next player)
     */
    const autoPlay = props.gameSettings.shouldAutoPlay ?? false;

    /**
     * @constant possibilities
     * @type {{rewarder: (number|string|*), neutral: number, punisher: (number|string|*)}}
     * @description Possibilities of each role to throw the ball
     * back to the player.
     * @example
     * {
     *   "rewarder": 0.7,
     *   "punisher": 0.1,
     *   "neutral": 1/3,
     * }
     */
    const possibilities = unref({
      rewarder: props.gameSettings.rewarderPossibility ?? 0.7,
      punisher: props.gameSettings.punisherPossibility ?? 0.1,
      neutral: 1 / 3,
    });

    console.log(possibilities);

    /**
     * @constant playLog
     * @type {Ref<UnwrapRef<*[]>>}
     * @default []
     * @description All generated play log.
     * @example
     * [{
     *   "order"       : Number,
     *   "time"        : String,
     *   "from"        : String,
     *   "to"          : String,
     *   "playerTurn"  : Boolean,
     *   "rewarderID"  : String,
     *   "punisherID"  : String,
     *   "neutralID"   : String,
     * }, ...]
     */
    const playLog = ref([]);
    const trialOrder = ref(1);
    const ballHolder = ref("playerSelf");
    const rewarderID = ref("");
    const punisherID = ref("");
    const neutralID = ref("");
    const gameOver = ref(false);
    const gameRoles = ref({
      playerTwo: "",
      playerThree: "",
      playerFour: "",
    });

    const ballTransformX = ref(0);
    const ballTransformY = ref(0);
    const isPlayerTurn = ref(true);
    const finishTime = ref("");

    /**
     * @function writeLog
     * @description Write log to memory variable playLog.
     * @param {Number} order Order of trial
     * @param {String} from From whom the ball is thrown
     * @param {String} to To whom the ball is thrown
     * @param {UnwrapRef<boolean>} playerTurn Whether it is player's turn or not
     * @param {String} rewarderID Rewarder's ID
     * @param {String} punisherID Punisher's ID
     * @param {String} neutralID Neutral's ID
     * @param {Number} rewarderPossibility Rewarder's possibility to throw the ball back to the player
     * @param {Number} punisherPossibility Punisher's possibility to throw the ball back to the player
     * @example
     * writeLog(
     *         trialOrder.value, ballHolder.value, "playerThree",
     *         isPlayerTurn.value, rewarderID.value, punisherID.value,
     *         neutralID.value, rewarderPossibility, punisherPossibility
     *       );
     */
    const writeLog = (
      order,
      from,
      to,
      playerTurn,
      rewarderID,
      punisherID,
      neutralID,
      rewarderPossibility,
      punisherPossibility
    ) => {
      const logItem = {
        order: order,
        time: moment().format("YYYY-MM-DD HH:mm:ss"),
        from: from,
        to: to,
        playerTurn: playerTurn,
        rewarderID: rewarderID,
        punisherID: punisherID,
        neutralID: neutralID,
        rewarderPossibility: rewarderPossibility,
        punisherPossibility: punisherPossibility,
      };
      playLog.value.push(logItem);
      console.log(logItem);
    };

    /**
     * @constant
     * @type {Array}
     * @description Stores position of all ball possibilities
     *
     * - self: 0,0
     * - player2: -170,-160
     * - player3: -10,-270
     * - player4: 140,-160
     */
    const ballLocations = [
      [0, 0],
      [-170, -160],
      [-10, -270],
      [140, -160],
    ];

    /**
     * @function allocateRole
     * @description Allocate roles to computer players:
     *
     * - rewarder
     * - punisher
     * - neutral
     * @returns {Array} [rewarder, punisher, neutral]
     */
    const allocateRole = () => {
      const roles = ["playerTwo", "playerThree", "playerFour"];
      const selectAndPop = (array) => {
        const index = Math.floor(Math.random() * array.length);
        return array.splice(index, 1)[0];
      };
      const rewarder = selectAndPop(roles);
      const punisher = selectAndPop(roles);
      const neutral = selectAndPop(roles);
      rewarderID.value = rewarder;
      punisherID.value = punisher;
      neutralID.value = neutral;
      const roleList = {
        rewarder: rewarder,
        punisher: punisher,
        neutral: neutral,
      };
      gameRoles.value = lodash.invert(roleList);
      console.log(
        `Role changed.\nRewarder: ${rewarder}\nPunisher: ${punisher}\nNeutral: ${neutral}`
      );
      return [rewarder, punisher, neutral];
    };

    allocateRole();

    /**
     * @function throwToOne
     * @description Throw ball to playerOne (self) and push log.
     * This should never be activated by user.
     * @param {Boolean} playerActivated user activated this event or not.
     * In this case, it is always false.
     */
    const throwToOne = (playerActivated = false) => {
      [ballTransformX.value, ballTransformY.value] = ballLocations[0];
      writeLog(
        trialOrder.value,
        ballHolder.value,
        "playerSelf",
        isPlayerTurn.value,
        rewarderID.value,
        punisherID.value,
        neutralID.value,
        possibilities.rewarder,
        possibilities.punisher
      );
      trialOrder.value++;
      isPlayerTurn.value = true;
      ballHolder.value = "playerSelf";
    };

    /**
     * @function throwToTwo
     * @description Throw ball to playerTwo and push log.
     * Actions will be evaluated to decide whether this is a valid call or not.
     * @param {Boolean} playerActivated user activated this event or not.
     * @default false
     */
    const throwToTwo = (playerActivated = false) => {
      if (true === playerActivated && false === isPlayerTurn.value) {
        return;
      }
      [ballTransformX.value, ballTransformY.value] = ballLocations[1];
      writeLog(
        trialOrder.value,
        ballHolder.value,
        "playerTwo",
        isPlayerTurn.value,
        rewarderID.value,
        punisherID.value,
        neutralID.value,
        possibilities.rewarder,
        possibilities.punisher
      );
      trialOrder.value++;
      isPlayerTurn.value = false;
      ballHolder.value = "playerTwo";
    };

    /**
     * @function throwToThree
     * @description Throw ball to playerThree and push log.
     * Actions will be evaluated to decide whether this is a valid call or not.
     * @param {Boolean} playerActivated user activated this event or not.
     * @default false
     */
    const throwToThree = (playerActivated = false) => {
      if (true === playerActivated && false === isPlayerTurn.value) {
        return;
      }
      [ballTransformX.value, ballTransformY.value] = ballLocations[2];
      writeLog(
        trialOrder.value,
        ballHolder.value,
        "playerThree",
        isPlayerTurn.value,
        rewarderID.value,
        punisherID.value,
        neutralID.value,
        possibilities.rewarder,
        possibilities.punisher
      );
      trialOrder.value++;
      isPlayerTurn.value = false;
      ballHolder.value = "playerThree";
    };

    /**
     * @function throwToFour
     * @description Throw ball to playerFour and push log.
     * Actions will be evaluated to decide whether this is a valid call or not.
     * @param {Boolean} playerActivated user activated this event or not.
     * @default false
     */
    const throwToFour = (playerActivated = false) => {
      if (true === playerActivated && false === isPlayerTurn.value) {
        return;
      }
      [ballTransformX.value, ballTransformY.value] = ballLocations[3];
      writeLog(
        trialOrder.value,
        ballHolder.value,
        "playerFour",
        isPlayerTurn.value,
        rewarderID.value,
        punisherID.value,
        neutralID.value,
        possibilities.rewarder,
        possibilities.punisher
      );
      trialOrder.value++;
      isPlayerTurn.value = false;
      ballHolder.value = "playerFour";
    };

    /**
     * @function userThrow
     * @description activate user throw event.
     * @param {String} destination the destination (to whom) will the ball be thrown to.
     */
    const userThrow = (destination) => {
      if (false === isPlayerTurn.value) {
        return;
      }
      switch (destination) {
        case "playerTwo":
          throwToTwo(true);
          break;
        case "playerThree":
          throwToThree(true);
          break;
        case "playerFour":
          throwToFour(true);
          break;
        default:
          throwToOne(true);
          break;
      }
    };

    /**
     * @function trialOrder.value watcher.
     * @description monitors the game process and
     * decide when to switch schedule.
     */
    watch(
      () => trialOrder.value,
      (newValue) => {
        if (1 === newValue % allocationOccursEach) {
          allocateRole();
        } else if (maxTrialTime < newValue) {
          gameOver.value = true;
          finishTime.value = moment().format("YYYY-MM-DD_HH:mm:ss");
        }
      }
    );

    /**
     * @function randomThrow
     * @param availableDestArray
     * @example randomThrow(["playerTwo", "playerThree"])
     * @description throw ball to random player EXCEPT playerSelf (user).
     */
    const randomThrow = (availableDestArray) => {
      const randomIndex = Math.floor(Math.random() * availableDestArray.length);
      const randomDestination = availableDestArray[randomIndex];
      switch (randomDestination) {
        case "playerTwo":
          throwToTwo();
          break;
        case "playerThree":
          throwToThree();
          break;
        case "playerFour":
          throwToFour();
          break;
      }
    };

    /**
     * @function systemThrow
     * @description Let the system decide to whom to throw the ball
     * based on the current ball holder's role.
     * @param {String} ballHolderID the current ball holder.
     * @default ballHolder.value
     * @param {String} ballHolderRole the current ball holder.
     * @default gameRoles.value[ballHolder.value]
     */
    const systemThrow = (ballHolderID, ballHolderRole) => {
      // falsey value check
      if (
        [undefined, null, 0, ""].includes(ballHolderID) ||
        [undefined, null, 0, ""].includes(ballHolderRole)
      ) {
        ballHolderID = ballHolder.value;
        ballHolderRole = gameRoles.value[ballHolderID];
      }

      const availableDestinations = [
        "playerSelf",
        "playerTwo",
        "playerThree",
        "playerFour",
      ].filter(
        (destination) =>
          ballHolderID !== destination && "playerSelf" !== destination
      );

      const randomWaitSeconds =
        Math.floor(
          Math.random() * Math.abs(maxWaitingSeconds - minWaitingSeconds)
        ) + minWaitingSeconds;
      const rollPoint = Math.random();

      switch (ballHolderRole) {
        case "rewarder":
          console.log("rewarder schema");
          if (rollPoint < possibilities.rewarder) {
            setTimeout(() => {
              throwToOne(false);
              console.log("rewarder throw to one");
            }, randomWaitSeconds * 1000);
          } else {
            setTimeout(() => {
              randomThrow(availableDestinations);
              console.log("rewarder throw to random");
            }, randomWaitSeconds * 1000);
          }
          break;
        case "punisher":
          console.log("punisher schema");
          if (rollPoint < possibilities.punisher) {
            setTimeout(() => {
              throwToOne(false);
              console.log("punisher throw to one");
            }, randomWaitSeconds * 1000);
          } else {
            setTimeout(() => {
              randomThrow(availableDestinations);
              console.log("punisher throw to random");
            }, randomWaitSeconds * 1000);
          }
          break;
        case "neutral":
          console.log("neutral schema");
          if (rollPoint < possibilities.neutral) {
            setTimeout(() => {
              throwToOne(false);
              console.log("neutral throw to one");
            }, randomWaitSeconds * 1000);
          } else {
            setTimeout(() => {
              randomThrow(availableDestinations);
              console.log("neutral throw to random");
            }, randomWaitSeconds * 1000);
          }
          break;
        default:
          console.error("unknown role!");
          break;
      }
    };

    /**
     * @function ballHolder.value watcher.
     * @description monitors who holds the ball and
     * if is not player's turn, then call systemThrow to decide whom to throw to.
     */
    watch(
      () => ballHolder.value,
      (newValue) => {
        if ("playerSelf" !== newValue && false === gameOver.value) {
          console.log(newValue, gameRoles.value[newValue]);
          const ballHolderID = newValue;
          const ballHolderRole = gameRoles.value[ballHolderID];
          systemThrow(ballHolderID, ballHolderRole);
        } else if (
          "playerSelf" === newValue &&
          false === gameOver.value &&
          true === autoPlay &&
          1 !== trialOrder.value
        ) {
          setTimeout(() => {
            randomThrow(["playerTwo", "playerThree", "playerFour"]);
          }, 1000);
        }
      }
    );

    const saveFile = (blob, filename) => {
      const a = document.createElement("a");
      document.body.appendChild(a);
      const url = window.URL.createObjectURL(blob);
      a.href = url;
      a.download = filename;
      a.click();
      setTimeout(() => {
        window.URL.revokeObjectURL(url);
        document.body.removeChild(a);
      }, 0);
    };

    const hasLogSaved = ref(false);

    const resetState = () => {
      hasLogSaved.value = false;
      gameOver.value = false;
      trialOrder.value = 1;
      ballHolder.value = "playerSelf";
      isPlayerTurn.value = false;
      rewarderID.value = "";
      punisherID.value = "";
      neutralID.value = "";
      gameRoles.value = {
        playerTwo: "",
        playerThree: "",
        playerFour: "",
      };
    };

    const emitGameOverSignal = () => {
      if (true === hasLogSaved.value) {
        context.emit("gameOver");
        // saveFile(
        //   new Blob([JSON.stringify(unref(playLog))], { type: "plain/text" }),
        //   `log_${unref(finishTime.value)}.json`
        // );
        resetState();
      }
    };

    const JSONtoCsv = (jsonObject) => {
      const jsonKeys = Object.keys(jsonObject[0]);
      const csvHeader = `${jsonKeys.join(",")}`;
      const jsonBody = jsonObject.map((row) => {
        return Object.keys(row)
          .map((key) => {
            return row[key];
          })
          .join(",");
      });

      return [csvHeader, ...jsonBody].join("\n");
    };

    const savePlayLog = () => {
      saveFile(
        new Blob([JSONtoCsv(unref(playLog))], { type: "plain/text" }),
        `log_${unref(finishTime.value)}.csv`
      );
      hasLogSaved.value = true;
    };

    return {
      playLog,
      trialOrder,
      ballTransformX,
      ballTransformY,
      isPlayerTurn,
      userThrow,
      gameOver,
      emitGameOverSignal,
      savePlayLog,
      hasLogSaved,
    };
  },
};
</script>

<template>
  <div style="display: contents" v-if="!gameOver">
    <div id="game-display">
      <player-entity
        id="player-self"
        @click="userThrow"
        :player-name="participantName"
        :player-id="'playerSelf'"
        :click-enabled="false"
      />
      <player-entity
        id="player-two"
        @click="userThrow('playerTwo')"
        :player-name="'Player'"
        :player-id="'playerTwo'"
        :click-enabled="isPlayerTurn"
      />
      <player-entity
        id="player-three"
        @click="userThrow('playerThree')"
        :player-name="'Player'"
        :player-id="'playerThree'"
        :click-enabled="isPlayerTurn"
      />
      <player-entity
        id="player-four"
        @click="userThrow('playerFour')"
        :player-name="'Player'"
        :player-id="'playerFour'"
        :click-enabled="isPlayerTurn"
      />
    </div>
    <img
      alt="ball"
      id="ball"
      src="../assets/ball.png"
      :style="{
        transform:
          'translate(' +
          ballTransformX +
          'px, ' +
          ballTransformY +
          'px) ' +
          'rotate(' +
          Math.random() * 360 +
          'deg)',
      }"
    />
  </div>
  <div v-if="gameOver" style="display: contents">
    <h1 id="endgame-display">{{ languageString.endgame }}</h1>
    <div class="container-horizontal">
      <confirm-button
        :text="languageString.return"
        @click="emitGameOverSignal"
        :type="hasLogSaved ? 'danger' : 'disabled'"
      />
      <confirm-button
        :text="languageString.saveLog"
        @click="savePlayLog"
        :type="'confirm'"
      />
    </div>
  </div>
</template>

<style scoped>
#game-display {
  display: grid;
  grid-template-areas:
    ". player-three ."
    "player-two . player-four"
    ". player-self .";
  grid-template-columns: 100px 100px 100px;
  grid-template-rows: 100px 100px 100px;
  grid-gap: 100px;
}

#player-self {
  grid-area: player-self;
}

#player-two {
  grid-area: player-two;
}

#player-three {
  grid-area: player-three;
}

#player-four {
  grid-area: player-four;
}

#ball {
  position: absolute;
  width: 32px;
  height: 32px;
  transform: translate(-50%, -50%);
  top: 440px;
  left: 50vw;
  transition: all 1s ease-in-out;
  filter: brightness(0.95) drop-shadow(0 2px 4px #cccccc);
}

.container-horizontal {
  display: grid;
  grid-auto-flow: column;
  grid-gap: 20px;
  place-items: center;
}
</style>
