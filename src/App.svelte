<script defer>
  import words from "random-words";
  import { onMount } from "svelte";

  let wordsArr = [];
  let displayedLetters = [];
  let typedLetters = [];
  $: typedLetters && createDynamicKeyWord();
  let definitionArr = [];
  let keyWord = "";
  let keyWordIndexArr = [];
  let dynamicKeyWord = [];
  let displayedCells = [];
  let currentWord = 0;
  let currentCell = 0;
  let startTime = null;
  let finalTime = null;
  let finished = false;

  const getKeyWord = () => {
      return words({ exactly: 1 })[0];
  };

  onMount(async function () {
      wordsArr = words({ min: 6, max: 8 });

      for (const word of wordsArr) {
          const def = await getDefinition(word);
          definitionArr.push(def);
      }

      definitionArr = definitionArr;
      console.log(wordsArr);

      while (keyWord.length < 7) {
          keyWord = getKeyWord();
      }

      hideLetters();

      pickKeyWordLetters();
      console.log(keyWord);

      createDynamicKeyWord();
      setTimeout(() => {
          addDisplayedCells();
      }, 0);
      startTime = Date.now();
  });

  function setCurrentCell() {
      let active = document.activeElement;
      for (let i = 0; i < displayedCells.length; i++) {
          let index = displayedCells[i].indexOf(active);
          if (index != -1) {
              currentCell = index;
              currentWord = i;
          }
      }
  }

  function cursorMove(row, col, event) {
      event = event ? event : window.event;
      var keyCode = event.code;
      if (keyCode == "ArrowRight") {
          let len = displayedCells[currentWord].length;
          if (len == currentCell + 1) {
              currentCell = 0;
          } else {
              currentCell += 1;
          }
          displayedCells[currentWord][currentCell].focus();
      }
      if (keyCode == "ArrowLeft") {
          let len = displayedCells[currentWord].length;
          if (currentCell == 0) {
              currentCell = len - 1;
          } else {
              currentCell -= 1;
          }
          displayedCells[currentWord][currentCell].focus();
      }
      if (keyCode == "Enter") {
          let len = wordsArr.length;
          currentCell = 0;
          if (currentWord == len - 1) {
              currentWord = 0;
          } else {
              currentWord += 1;
          }
          while (
              typedLetters[currentWord].join("") == wordsArr[currentWord]
          ) {
              if (currentWord == wordsArr.length - 1) {
                  currentWord = 0;
              } else {
                  currentWord += 1;
              }
          }
          displayedCells[currentWord][currentCell].focus();
      }
      if (keyCode == "Backspace") {
          displayedCells[currentWord][currentCell].value = null;
          typedLetters[row][col] = null;
      }
  }

  function afterClick(row, col, event) {
    event = event ? event : window.event;
    var keyCode = event.code;
    if(keyCode != "Backspace" && keyCode != "Enter" && keyCode != "ArrowLeft" && keyCode != "ArrowRight") {
        let len = displayedCells[currentWord].length;
          if (len == currentCell + 1) {
              currentCell = currentCell;
          } else {
              currentCell += 1;
          }
        displayedCells[currentWord][currentCell].focus();
      }
  }

  function addDisplayedCells() {
      const crossword = document.getElementById("crossword");
      let arr = [];
      let word = crossword.firstElementChild;
      for (let i = 0; i < wordsArr.length; i++) {
          arr.push([]);
          let letter = word.firstElementChild;
          letter = letter.nextElementSibling;
          for (let j = 0; j < wordsArr[i].length; j++) {
              let isInputText = letter instanceof HTMLInputElement;
              if (isInputText) {
                  if (displayedLetters[i][j] == null) {
                      arr[i].push(letter);
                  }
              } else {
                  letter = letter.nextElementSibling;
                  if (displayedLetters[i][j] == null) {
                      arr[i].push(letter);
                  }
              }
              letter = letter.nextElementSibling;
          }
          word = word.nextElementSibling;
      }
      displayedCells = arr;
  }

  function checkWord(index) {
      checkEnd()
      if (!finished) {
          if (wordsArr[index] == typedLetters[index].join("")) {
              if (currentWord == wordsArr.length - 1) {
                  currentWord = 0;
              } else {
                  currentWord += 1;
              }
              while (
                  typedLetters[currentWord].join("") == wordsArr[currentWord]
              ) {
                  currentWord += 1;
              }
              setTimeout(() => {
                currentCell = 0;
              displayedCells[currentWord][currentCell].focus();
            }, 100);
          }
      } else {
          finalTime = Date.now() - startTime;
          console.log(finalTime / 1000);
      }
  }

  function checkEnd() {
      if (
          JSON.stringify(typedLetters.map((e) => e.join(""))) ==
              JSON.stringify(wordsArr) &&
          wordsArr.length != 0
      ) {
          finished = true;
      }
      else {
          finished = false;
      }
  }

  function createDynamicKeyWord() {
      dynamicKeyWord = [...Array(keyWord.length)].map((e) => "");
      for (const arr of keyWordIndexArr) {
          const letter = typedLetters[arr[0]][arr[1]];
          const index = getKeyWordIndex(arr);
          dynamicKeyWord[index] = letter;
      }
  }

  function getKeyWordIndex(arr) {
      for (let i = 0; i < keyWordIndexArr.length; i++) {
          if (JSON.stringify(keyWordIndexArr[i]) == JSON.stringify(arr)) {
              return i;
          }
      }
      return -1;
  }

  function checkKeyWordLetter(arr) {
      for (let i = 0; i < keyWordIndexArr.length; i++) {
          if (JSON.stringify(keyWordIndexArr[i]) == JSON.stringify(arr)) {
              return true;
          }
      }
      return false;
  }

  function pickKeyWordLetters() {
      keyWordIndexArr = [];
      for (let l = 0; l < keyWord.length; l++) {
          keyWordIndexArr.push([]);
          for (let i = 0; i < wordsArr.length; i++) {
              for (let j = 0; j < wordsArr.length; j++) {
                  if (wordsArr[i][j] == keyWord[l]) {
                      if (!checkKeyWordLetter([i, j])) {
                          if (
                              []
                                  .concat(...keyWordIndexArr.map((x) => x[0]))
                                  .filter((x) => x == i).length < 2
                          ) {
                              keyWordIndexArr[l] = [i, j].slice();
                              break;
                          }
                      }
                  }
              }
              if (keyWordIndexArr[l].length != 0) {
                  break;
              }
          }
          if (keyWordIndexArr[l].length == 0) {
              keyWord = getKeyWord();
              while (keyWord.length < 7) {
                  keyWord = getKeyWord();
              }
              pickKeyWordLetters();
              break;
          }
          let temp = keyWordIndexArr[0][0];
          keyWordIndexArr[0][0] = -1;
          keyWordIndexArr[0][0] = temp;
      }
  }

  function hideLetters() {
      for (let i = 0; i < wordsArr.length; i++) {
          let indexArr = [];
          while (
              indexArr.length !=
              Math.floor(
                  wordsArr[i].length / 3 > 3 ? 3 : wordsArr[i].length / 3
              )
          ) {
              const random = Math.floor(
                  Math.random() * (wordsArr[i].length - 1)
              );

              if (!indexArr.includes(random)) {
                  indexArr.push(random);
              }
          }
          displayedLetters.push([]);
          for (let j = 0; j < wordsArr[i].length; j++) {
              if (indexArr.includes(j)) {
                  displayedLetters[i].push(wordsArr[i][j]);
              } else {
                  displayedLetters[i].push(null);
              }
          }
      }

      displayedLetters.forEach((word, index) => {
          typedLetters.push([]);
          word.forEach((letter) => {
              typedLetters[index].push(letter);
          });
      });

      //force svelte to display letters
      let temp = displayedLetters[0][0];
      displayedLetters[0][0] = "1";
      displayedLetters[0][0] = temp;
  }
  async function getJSON(word) {
      try {
          return await fetch(
              `https://api.dictionaryapi.dev/api/v2/entries/en/${word}`
          )
              .then((response) => response.json())
              .then((data) => {
                  return data[0]["meanings"][0]["definitions"][0][
                      "definition"
                  ];
              });
      } catch {
          document.location.reload();
      }
  }
  async function getDefinition(word) {
      return await getJSON(word);
  }
</script>

<h1 class="title">Crossword</h1>

<div id="crossword">
  {#each displayedLetters as word, index}
      <div class="word">
          <p>{index + 1}</p>
          {#each word as letter, index2}
              {#if checkKeyWordLetter([index, index2])}
                  <label for="">{getKeyWordIndex([index, index2]) + 1}</label>
              {/if}
              {#if letter == null}
                  <input
                      class="letter"
                      type="text"
                      maxlength="1"
                      bind:value={typedLetters[index][index2]}
                      on:click={setCurrentCell}
                      on:keyup={() => afterClick(index, index2)}
                      on:keydown={() => cursorMove(index, index2)}
                      on:input={() => checkWord(index)}
                      disabled={wordsArr[index] ==
                      typedLetters[index].join("")
                          ? true
                          : false}
                  />
              {:else}
                  <input class="letter" type="text" value={letter} disabled />
              {/if}
          {/each}
          {#if wordsArr[index] == typedLetters[index].join("")}
              <svg
                  xmlns="http://www.w3.org/2000/svg"
                  width="50"
                  height="50"
                  fill="currentColor"
                  class="bi bi-check"
                  viewBox="0 0 16 16"
              >
                  <path
                      d="M10.97 4.97a.75.75 0 0 1 1.07 1.05l-3.99 4.99a.75.75 0 0 1-1.08.02L4.324 8.384a.75.75 0 1 1 1.06-1.06l2.094 2.093 3.473-4.425a.267.267 0 0 1 .02-.022z"
                  />
              </svg>
          {/if}
      </div>
  {/each}
</div>

<h2 class="keyTitle">Keyword</h2>
<div class="keyWord">
  {#if dynamicKeyWord.length == 0}
      <p>Loading...</p>
  {:else}
      {#each dynamicKeyWord as letter, index}
          <label for="">{index + 1}</label>
          <input class="letter" type="text" bind:value={letter} disabled />
      {/each}
      {#if finished}
          <svg
              xmlns="http://www.w3.org/2000/svg"
              width="60"
              height="60"
              fill="currentColor"
              class="bi bi-check"
              viewBox="0 0 16 16"
          >
              <path
                  d="M10.97 4.97a.75.75 0 0 1 1.07 1.05l-3.99 4.99a.75.75 0 0 1-1.08.02L4.324 8.384a.75.75 0 1 1 1.06-1.06l2.094 2.093 3.473-4.425a.267.267 0 0 1 .02-.022z"
              />
          </svg>
      {/if}
  {/if}
</div>
{#if finished}
  <p>Congrats! Solving time: {finalTime/1000}s</p>
{/if}

<div class="definitions">
  <h2>Definitions</h2>
  <ol>
      {#if definitionArr.length == 0}
          <li>Loading...<br>If it won't load please refresh the page.</li>
      {:else}
          {#each definitionArr as definition}
              <li>{definition}</li>
          {/each}
      {/if}
  </ol>
</div>

<style>
  .title {
      font-size: 4em;
      margin-bottom: 30px;
      font-weight: 100;
  }
  #crossword {
      width: min-content;
      margin: auto;
      display: flex;
      flex-direction: column;
      flex-wrap: nowrap;
      gap: 1px;
  }
  .word,
  .keyWord {
      display: flex;
      gap: 1px;
      position: relative;
      width: min-content;
  }
  .word p {
      font-weight: 300;
      font-size: 1.5em;
      align-self: center;
      margin-right: 2px;
  }
  .word label,
  .keyWord label {
      margin-top: -4px;
      font-size: 0.7em;
      width: 13px;
      text-align: center;
      z-index: 1;
  }
  .word label + .letter,
  .keyWord .letter {
      margin-left: -14px;
  }
  .word svg {
      position: absolute;
      right: -50px;
  }
  .letter {
      width: 50px;
      height: 50px;
      border: 1px solid black;
      border-radius: 5px;
      text-align: center;
      font-weight: 900;
      font-size: 1.5em;
      text-transform: capitalize;
  }
  .keyWord {
      width: min-content;
      margin: auto;
  }
  .keyTitle {
      margin-top: 40px;
      margin-bottom: 5px;
      font-size: 1.5em;
      font-weight: 100;
  }
  .keyWord .letter {
      width: 60px;
      height: 60px;
      margin-left: -21px;
  }
  .keyWord label {
      font-size: 0.9em;
      margin-top: -2px;
      width: 25px;
  }
  .keyWord svg {
      position: absolute;
      right: -55px;
  }
  .definitions {
      margin: 50px;
      text-align: initial;
  }
  .definitions h2 {
      font-size: 2em;
      margin-bottom: 10px;
  }
  .definitions ol {
      margin-left: 30px;
      list-style: decimal;
  }
</style>
