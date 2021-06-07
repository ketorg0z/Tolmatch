<link rel="preconnect" href="https://fonts.gstatic.com">
<link href="https://fonts.googleapis.com/css2?family=Montserrat:wght@300;700&display=swap" rel="stylesheet">

<script>
  import {createSmartappDebugger, createAssistant, initializeAssistantSDK} from '@sberdevices/assistant-client'
  import { onMount } from 'svelte'

  let message = '';
  const backendUrl = 'https://tolmatch-backend.herokuapp.com/'
  let wordsState = []
  let idState
  let token = 'eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiI3NmI4MjA4OWJiNzJkMTA3NGMzMWIwNmQ1YWVmMjE0NDg4NjhjYjZkZTUxZDM4NTE0ZDY1ZjZlYTJmOTgxOTlhNTM5YmU5MjcwMDQyNjI5OCIsImF1ZCI6IlZQUyIsImV4cCI6MTYyMzE2ODgxMSwiaWF0IjoxNjIzMDgyNDAxLCJpc3MiOiJLRVlNQVNURVIiLCJ0eXBlIjoiQmVhcmVyIiwianRpIjoiZDE4MjEwOGYtNzRmMi00ZTZjLTljYjMtNzM5Yjg5MDVlMjcwIiwic2lkIjoiMmYyNjlmYmItZmI2Yi00Y2NjLThkODYtODI0MGFjYjRkY2QyIn0.Uq-3vzElco6imp4AYu_O4wJJqWNrYiN7Mwe7_P_mo8uHx0KBt0EQyEp4kCHRjnMWC3_w2HCE7KcT8pmZrfKxrryaKlF9UcTHhsQTKMFeuc-kCDJqQdlw0evXKvAf_ADWoGiE3x5To9UG7hglxi2XQV-7TdzWrZxbevfX-8m4_YMtD1Awtsf81ptLtZYucw5i6nHkSLdWqjU81WAFcu926kTEtsxtN6iEZs4VGgO74fT7Vr-gNSiUAsBetNEZeHl-40h10WedQQ0jE6jQnnU0PKV_-7UMhIxt3LeJFiyDDmVHOXCjdVaqFgEQgX6zM1b5QLWiGxAjdxQKIFNYWjToCvYnv5w4Bk_hZjE9c7IHmMot0PZa_AtCUh6JdpE18LNi3BEeH4KV-Y_-ItNLJSOGabwFNTu5-2zK16fEGtKS-FS9sMVBrWtqGGBWZ-XiSEaQLgY741GmWL-nO73nudOqPYW07RQc7IkxoQwM3clA7oZ36_Or24fQuzvEwZiaA0IiFQ1dLb1IATOkPu3AqiP89UHK9BiRriZX0bn5JE_wPw4NPOSa9NLeaLDhtZadpCzpxMZ1dxicSImWtLQTwgEd5I9qere5zAxoiOg5104hbGjE4wAXDUb056GsecOEa_OIJJh1C1me0EOi2OaPlXNf3Juidz1DVq-7UMzTCg2A2BY';
  let initPhrase = 'Включи толмач'; 

  function getState() {
    //console.log("State was get");
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
    //console.log(state)
    return state;
  }

  let assistant;
  onMount(() => {
      const init = () => {
      //  return createSmartappDebugger({
      //   token,
      //   initPhrase,
      //   getState,
      //   settings: {debugging: false}
      //  })
       return createAssistant({getState});
      }
    assistant = init();

    assistant.on("start", (event) => {
      //console.log(`assistant.on(start)`, event);
    });

    assistant.on("data", (event) => {
      console.log("event", event)
      switch (event?.action?.type) {
        case 'answer':
          if (wordsState[idState] === event.action.word) {
            message = 'Верно';
            document.getElementById('message').style.color = 'green'
            var step;
            for (step = 0; step < 3; step++) {
              if (wordsState[step] == event.action.word){
                document.getElementById(step.toString()).style.backgroundColor = 'green'
                break
              }
            }
            var delayInMilliseconds = 700;
            setTimeout(function() {
              promise = newGame();
              message = ''
            }, delayInMilliseconds);
          } else {
            let k = 0
            //console.log(wordsState)
            var step;
            for (step = 0; step < 3; step++) {
              if (wordsState[step] == event.action.word){
                document.getElementById(step.toString()).style.backgroundColor = 'red'
                break
              }
            }
            message = 'Неверно'
          }
        break
      } 
    });
  })

  function check(ind, rightInd) {
    if (ind === rightInd) {
      message = 'Верно';
      document.getElementById('message').style.color = 'green'
      document.getElementById(ind.toString()).style.backgroundColor = 'green'

      var delayInMilliseconds = 700;

      setTimeout(function() {
        promise = newGame();
        message = ''
      }, delayInMilliseconds);

    assistant.sendData({
      action: {
        action_id: 'yes',
        parameter: {}
      }}
      )
    } else {
      document.getElementById(ind.toString()).style.backgroundColor = 'red'
      message = 'Неверно'
      assistant.sendData({
        action: {
          action_id: 'no',
          parameter:  {}
        }}
      )
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
            <button id={i.toString()} on:click={() => check(i, game.trueIndex) }>{word}</button>
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

  button {
    width: 250px;
    min-width: 100px;
    cursor: pointer;
    margin: 10px 20px;
    padding: 20px;
    border-radius: 20px;
    transition: background 0.3s linear 0.1s , color 0.3s linear 0s, border 0.3s linear 0.1s;
  }

  button:hover {
    background-color: yellowgreen;
    border-color: yellowgreen;
  }
  button:focus {
    background-color: yellowgreen;
    border-color: yellowgreen;
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