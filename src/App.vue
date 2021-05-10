<template>
  <div class="h-screen w-screen flex flex-col">
    <div class="flex flex-row justify-between p-4">
      <div class="flex flex-row space-x-4 text-gray-700">
        <button
          class="font-bold p-3 pb-2 pt-2 bg-green-400 rounded-md hover:bg-green-500"
          @click="selectDifficulty(0)"
          :class="{ 'text-black': difficulty === 0 }"
        >Easy</button>
        <button
          class="font-bold p-3 pb-2 pt-2 rounded-md hover:bg-blue-500 bg-blue-400"
          @click="selectDifficulty(1)"
          :class="{ 'text-black': difficulty === 1 }"
        >Medium</button>
        <button
          class="font-bold p-3 pb-2 pt-2 bg-purple-400 rounded-md hover:bg-purple-500"
          @click="selectDifficulty(2)"
          :class="{ 'text-black': difficulty === 2 }"
        >Expert</button>
      </div>
      <div>
        <div v-if="playing">
          <span>
            {{markedBombs}}
          </span>
          / 
          <span>
            {{totalBombs}}
          </span>
          bombs marked
        </div>
        <button
          v-else
          class="p-3 pt-2 pb-2 bg-red-400 rounded-full font-bold hover:bg-red-500"
          @click="startGame"
        >Start</button>
      </div>
      <div class="bg-gray-800 rounded-md text-center flex items-center p-3">{{ secondsToTimer() }}</div>
    </div>
    <div class="flex flex-col items-center">
      <div class="bg-gray-400 p-1 rounded-md" @contextmenu="$event.preventDefault()">
        <div v-for="row in height" class="flex flex-row">
          <cell
            v-for="col in width"
            :row="row - 1"
            :col="col - 1"
            :state="board[row - 1][col - 1]"
            :value="truth ? truth[row - 1][col - 1] : NaN"
            @click="onClick"
            @flag="onFlag"
          ></cell>
        </div>
      </div>
    </div>
    <div 
    v-if="gameOver" 
    class="z-20 w-screen h-screen absolute bg-opacity-20 flex items-center justify-center"
    :class="[win ? 'bg-green-600' : 'bg-red-600']">
      <div class="flex flex-col bg-black rounded-lg p-4">
        <p class="text-xl font-bold">{{win ? 'YOU WIN!' : 'GAME OVER'}}</p>
        <button class="bg-green-400 rounded-md font-bold text-gray-800 hover:bg-green-500" @click="playAgain">PLAY AGAIN?</button>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, computed, watchEffect, watch } from "vue";
import { floor, values } from 'lodash';
import Cell from "@/components/Cell.vue"
import lodash from 'lodash'

const playing = ref(false);
const difficulty = ref(1); // 0 = easy, 1 = medium, 2 = expert
const totalBombs = ref(40);
const markedBombs = ref(0);
const timerSeconds = ref(0);
const truth = ref(); // Truth array for the values / hidden state. -1: Bomb, [0-8]: Number of bombs near
const board = ref(matrix(16, 16)); // Board state, flagged: 1 / unflagged: 0 / exposed: 2
const height = ref(16);
const width = ref(16);
const gameOver = ref(false);
const win = ref(false);

function matrix(m: number, n: number): Array<Array<number>> {
  return Array.from({
    length: m
  }, () => new Array(n).fill(0));
}

watch(() => difficulty.value, (diff) => {
  // Need to change a bunch of stuff if we change the difficulty.
  if (diff === 0) {
    totalBombs.value = 10;
    height.value = 9;
    width.value = 9;
    // board.value = matrix(height.value, width.value);
  } else if (diff === 1) {
    totalBombs.value = 40;
    height.value = 16;
    width.value = 16;
    // board.value = matrix(height.value, width.value);
  } else {
    // diff === 2
    totalBombs.value = 99
    height.value = 16;
    width.value = 30;
    // board.value = matrix(height.value, width.value);
  }
  playAgain();
})

const selectDifficulty = (diff: number) => {
  // If it's the selected difficulty already, we don't change anything.
  if (difficulty.value === diff)
    return;

  difficulty.value = diff;
}

const playAgain = () => {
  gameOver.value = false;
  playing.value = false;
  board.value = matrix(height.value, width.value);
  truth.value = null;
  timerSeconds.value = 0;
  markedBombs.value = 0;
}
const startGame = () => {
  playing.value = true;
  gameOver.value = false;
}

const secondsToTimer = () => {
  const m = floor(timerSeconds.value / 60);
  const s = timerSeconds.value % 60;

  let minutes = "";
  let seconds = "";
  if (m < 10)
    minutes = `0${m}`;
  else
    minutes = `${m}`;

  if (s < 10)
    seconds = `0${s}`
  else
    seconds = `${s}`

  return `${minutes}:${seconds}`;
}

setInterval(() => {
  if (playing.value)
    timerSeconds.value++;
}, 1000)

function getSurroundingCells(row: number, col: number) {
  const cells = [];
  if (row > 0) {
    // Top
    cells.push([row - 1, col]);
    // Top left
    if (col > 0)
      cells.push([row - 1, col - 1]);
    // Top right
    if (col < width.value - 1)
      cells.push([row - 1, col + 1]);
  }
  if (row < height.value - 1) {
    // Bottom
    cells.push([row + 1, col]);
    // Bottom left
    if (col > 0)
      cells.push([row + 1, col - 1])
    // Bottom right
    if (col < width.value - 1)
      cells.push([row + 1, col + 1])
  }
  // Left
  if (col > 0)
    cells.push([row, col - 1])
  // Right
  if (col < width.value - 1)
    cells.push([row, col + 1])
  return cells;
}

function recursiveExpose(row: number, col: number, checked: Array<{row: number, col: number}>) {
  const toCheck = getSurroundingCells(row, col);
  // For adjacent tile
  for (const [r,c] of toCheck) {
    // If we haven't checked it
    if (!checked.some((val) => val.row === r && val.col === c)) {
      // Add it to the checked tiles
      checked.push({row: r,col: c})
      // If it's also a 0, we expose it and recurse on it.
      board.value[r][c] = 2
      if (truth.value[r][c] === 0) {
        recursiveExpose(r,c,checked);
      }
    }
  }
}

const expose = (row: number, col: number) => {
  if (board.value[row][col] === 2)
    return;
  // Mark it exposed
  board.value[row][col] = 2;
  if (truth.value[row][col] === -1) {
    // Check if it's a bomb, then game over
    gameOver.value = true;
    win.value = false;
    playing.value = false;
  } else if (truth.value[row][col] === 0) {
    // Check if it's not connected to any bombs, then we need to expose nearby "non-bombs"
    recursiveExpose(row,col,[]);
  }
}

const onClick = ({ row, col }: any) => {
  if (gameOver.value)
    return;

  if (!truth.value) {
    // If we click on something and the truth hasn't started, we need to generate it and make sure hte game is started.
    truth.value = matrix(height.value, width.value);
    // Generate a shuffled array of indexes to act as where we'll place bombs
    const bombs = lodash.shuffle(lodash.range(height.value * width.value));
    // In order to start with a small exposed area, we need to set the place we need to remove the bombs from the place we clicked and the surrounding 8.
    const clickIndex = row * width.value + col;
    const surrounding = getSurroundingCells(row, col);
    lodash.remove(bombs, (bombIndex) => {
      if (bombIndex === clickIndex) {
        return true;
      } else if (surrounding.some((val) => val[0] * width.value + val[1] === bombIndex)) {
        return true;
      } 
      return false;
    });
    // Limit to the first N values of the bombs array.
    for (const i in lodash.range(totalBombs.value)) {
      const b = bombs[i];
      const row = lodash.floor(b / width.value);
      const col = b % width.value;
      truth.value[row][col] = -1;
      // Let's also increment every index around our row/col that isn't also equal to -1.
      for (const [r, c] of getSurroundingCells(row, col)) {
        if (truth.value[r][c] !== -1)
          truth.value[r][c]++;
      }
    }
    if (!playing.value)
      startGame();
  }
  expose(row, col);
}

const onFlag = ({ row, col }: { row: number, col: number }) => {
  // Can't set flags if we haven't generated bombs yet & started the game.
  if (!truth.value)
    return;

  // Marking an unmarked cell.
  if (board.value[row][col] === 0) {
    if (markedBombs.value < totalBombs.value) {
      board.value[row][col] = 1;
      markedBombs.value++;
    }
  }
  // Unmarking a marked cell.
  else if (board.value[row][col] === 1) {
    board.value[row][col] = 0;
    markedBombs.value--;
  }
  // Checking if we've ended the game.
  if (markedBombs.value === totalBombs.value) {
    // Check if the user marked all total bombs
    // For each row and column
    for (const r of lodash.range(height.value)) {
      for (const c of lodash.range(width.value)) {
        // If it's marked as a bomb
         if (board.value[r][c]===1) {
           // Check if it's a bomb.
           if (truth.value[r][c]!==-1) {
            return;
           }
         }
      }
    }
    // If we make it here, all bombs marked are correct.
    gameOver.value = true;
    win.value = true;
  }
}
</script>

<style>
#app {
  @apply bg-gray-700;
  @apply text-white;
}
</style>