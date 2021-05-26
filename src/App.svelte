<link rel="preconnect" href="https://fonts.gstatic.com">
<link href="https://fonts.googleapis.com/css2?family=Montserrat:wght@300;700&display=swap" rel="stylesheet">

<script>
  import {createSmartappDebugger, createAssistant} from '@sberdevices/assistant-client'
  import { onMount } from 'svelte'

  let message = '';
  const backendUrl = 'https://tolmatch-backend.herokuapp.com/'
  let wordsState = []
  let idState
  let token = 'eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiI3NmI4MjA4OWJiNzJkMTA3NGMzMWIwNmQ1YWVmMjE0NDg4NjhjYjZkZTUxZDM4NTE0ZDY1ZjZlYTJmOTgxOTlhNTM5YmU5MjcwMDQyNjI5OCIsImF1ZCI6IlZQUyIsImV4cCI6MTYyMjEyNjA1MywiaWF0IjoxNjIyMDM5NjQzLCJpc3MiOiJLRVlNQVNURVIiLCJ0eXBlIjoiQmVhcmVyIiwianRpIjoiZDYxNDMxMzYtNWRiNy00MzdiLWI1YTktZmFlY2QyNTMyMWI4Iiwic2lkIjoiZjkzOTVlNDgtOTk2Yi00YmU0LWE1ZDktM2NiMDZlNGUwY2ZiIn0.i6D0w8SMDVsiO-2vMJsMRGDqoyfHICFqyd0thF5uqupR1rfIqFQvwwsOJCgj6wPszfRS_U_MbObefNyRpVIbI6VunfA3YW0xBgBc8WtKalRpJPQdalF0GtmabvVJ5H4YVNq2d7tNwyqz2y8eZLyeoTRx8WgtsxHs_i53vloTU0lAopSch6WqxVqXAtjH4uMQbCdAN9SvhpXWIJsMu-Jn53EbNxLJ7tnavkkN1wAG6LHAaRxYtK8Y730xUb7ITsAWrWX7Jdjeo5HrbglwQ-AE-AdeDAiGAoo-phNKUbqrqnAsHHlWFqVVishjZT8H63I5ttAjNyqSNUWL_GiE6VfB8mijHfIYITd25zSILfGbNP0tzN1cPDSNfxUYSKL4blIdaqaMJDzjh9Js7QtAc6-cZWGmvIaWvsaBF6Yly1EU9bTV8iW82qeiP5YpBegoJSPWmUG2gaMI_Hb_rBE1gbriND6RHG_6k0lPaCmfzJSpFyIAWmwUvff9OXqJ_ziJTp9_POGSTbaLZwUCKX2QGjPXhUvmtcMPYFB3O1wNvtY0z3rc8CP9hVGCeuhHghoY5QKvgvN1VUA3I44CU74iWHY7TC0vigHpMbjXUPhEeKIUBY_-rBnN1T-fi2FCQdjh8aVHTw3fxC51VIJjexz6Ki_A0N8I1k5T0zLyxHrT9rudUjE';
  //let token = 'eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiJhMzM2YmEwMjU3MzEyN2RkMDY1MmYxNDRmZDI5NmQwNWViNmIyZTk5MTJhMmVlMDQ5NzAyMWE4ZGE0NDVjM2E5NTM5YmU5MjcwMDQyNjI5OCIsImF1ZCI6IlZQUyIsImV4cCI6MTYyMjAzNDIzNCwiaWF0IjoxNjIxOTQ3ODI0LCJpc3MiOiJLRVlNQVNURVIiLCJ0eXBlIjoiQmVhcmVyIiwianRpIjoiNDkxZTRjMjUtOGE0Ni00MTIxLTllMDktOGNkYWVmYjg0NTU0Iiwic2lkIjoiYjI1MmRmMzktY2JkNy00NmY5LThhZWEtYjAyNTNjMzA4MTI2In0.UJ-GIOJ0JqNw1kBDe_9T42YXBZL6LrmQ9dWMOKiRG6qWBxSPrwYZD1LS32KENvLv7NvCURs_nTM6J622vooQ-EM366yj1UiQQ_sMmw8XPKlq1q46GIfM9nF0ngu_Lzny4GDfxytsA_T-DdQby9g1HdqH1TWnbOhUA3gz7S2wL-esWF2IENDJI01Yx_VR141fi9fc8eM0PM_gHWJOsL7v8ZHFMBcDhT4zRsHPhgXxKJCApL98xzu9akpXjrrJ_5F8gvaZ-mKSVTsd3aAZyzR4Eh9JuVmhyCekD3BohwF61Tn6wse7FjTYkM5yzqM8BY8CyIchAQUPg3yBS3z9E1K5jn4WiDBFpUVI7vrkZOCZspa7l64jagUyCASoVHLni-PArZBFsn9ztPKA0rh4znTh5Hz8JMDktzT6bG7QYe911DrWtTL0dQkmz73KHEzaH1_TWt2C3Qbaht_i78qGK7Oo5oG4WPWI_wv9Keyg8_SCsk6I9oMSRVd-5onWO1cqfR8QUo4wuSJmUd84lhdfw_OEyWb8wFDmuGJemj_0zYRBnXdG_XV8AR1OLe1GZ9Ulv4sV3Ib0KSxMGYrGecj3Dy2BdQubUmOl7Zo6GY9X7_xgDSHqEw4h41lqtRRQQxew9q_aRACbdWJfrAPjxGOPf-PDmcis8PYsH9URfbGyJNbXYu8';
  let initPhrase = 'Включи игру переводчик'; 

  function getState() {
    console.log("State was get");
    const state = {
      item_selector: {
        items: [
          {rightId: idState},
          {
            words: wordsState
          }
        ],
      }
    }
    console.log(state)
    return state;
  }

  let assistant;
  onMount(() => {
    const init = () => {
      // return createSmartappDebugger({
      //   token,
      //   initPhrase,
      //   getState,
      //   settings: {debugging: false}
      // })
      return createAssistant({getState});
    }
    assistant = init();

    assistant.on("start", (event) => {
      console.log(`assistant.on(start)`, event);
    });

    assistant.on("data", (event) => {
      console.log('EVENT!!!', event);
      switch (event.action.type) {
        case 'answer':
          if (wordsState[idState] === event.action.word) {
            message = '';
            promise = newGame();
          } else {
            message = 'Неверно, подумай еще раз.'
          }
        break
      }
    });
  })

  function check(ind, rightInd) {
    if (ind === rightInd) {
      message = '';
      promise = newGame();
    } else {
      message = 'Неверно, подумай еще раз.'
    }
  }

  const newGame = async () => {
    const response = await fetch(backendUrl+'game');
    if (response.ok) {
      const json = await response.json();
      wordsState = json.words;
      idState = json.trueIndex;
      return json;
    } else {
      return {trueWord: 'Ошибка сервера'}
    }
  }

  let promise = newGame();
</script>

<main>
  <div id="game">
    {#await promise then game}
      <h1>{game.trueWord}</h1>
      <div id="words-block">
        {#each game.words as word, i}
          <button on:click={() => check(i, game.trueIndex)}>{word}</button>
        {/each}
      </div>
      <div id="message">
        <p>{message}</p>
      </div>
    {/await}
  </div>
</main>

<style>
	main {
    width: 100%;
    height: 100%;
    display: flex;
    justify-content: center;
    align-items: center;
    flex: 1 1;
    font-family: 'Montserrat', sans-serif;
    background-color: #3c3c3c;
    color: #FFFFFF;
	}

	#game {
    position: relative;
    text-align: center;
    padding-bottom: 50px;
  }

  #words-block {
    min-width: 300px;
    display: flex;
    justify-content: center;
    flex: 1 1;
    margin-top: 40px;
    flex-wrap: wrap;
  }

  #words-block button {
    width: 250px;
    min-width: 100px;
    cursor: pointer;
    margin: 10px 20px;
    padding: 20px;
    border-radius: 20px;
  }

  #message {
    width: 100%;
    position: absolute;
    bottom: 0;
    color: red;
    text-align: center;
  }

	@media (min-width: 640px) {
		main {
			max-width: none;
		}
	}
</style>