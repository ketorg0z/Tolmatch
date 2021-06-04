<link rel="preconnect" href="https://fonts.gstatic.com">
<link href="https://fonts.googleapis.com/css2?family=Montserrat:wght@300;700&display=swap" rel="stylesheet">

<script>
  import {createSmartappDebugger, createAssistant, initializeAssistantSDK} from '@sberdevices/assistant-client'
  import { onMount } from 'svelte'

  let message = '';
  const backendUrl = 'https://tolmatch-backend.herokuapp.com/'
  let wordsState = []
  let idState
  let token = 'eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiI3NmI4MjA4OWJiNzJkMTA3NGMzMWIwNmQ1YWVmMjE0NDg4NjhjYjZkZTUxZDM4NTE0ZDY1ZjZlYTJmOTgxOTlhNTM5YmU5MjcwMDQyNjI5OCIsImF1ZCI6IlZQUyIsImV4cCI6MTYyMjkxNjI2NiwiaWF0IjoxNjIyODI5ODU2LCJpc3MiOiJLRVlNQVNURVIiLCJ0eXBlIjoiQmVhcmVyIiwianRpIjoiMTQ5NmQ0NWMtYzg5MC00NDk5LWExNjctZWM5OGU5ODYwYTBkIiwic2lkIjoiMjQzYzJkMTktNTI5Ni00MGJjLTg4M2ItNWFjMWFjMmYzMzk3In0.jH8noU3F1U-O3HRXRmX8pEzlLEitbAY1wsGueGIyA3KPc95vi554tjpmsDVIQ0coZRDja85gk7DJvg8V3IhVwb7KEe-NyhvUDGZdhwg1y214BAjLWpY58zxVqol2mwKTrHXWmg07SpLdHC4TsgpkXwxOK4tnd34OGf3aDyXgJq-aasOl7gusRcVSSj9oL0C7NJ_Q5N28C4yUaSFh_FN56LPdnUIy5__rdVYE-w-Ld8Zi-2slyArxwOrJFQ0EEayAqMR4v_zG9BhpCSMPZGc65MKeGiX0_f9eleSvA22ua-bBE-f2UURYUw_HsF2xySGacaJdgn-dr1mp4wS0iBS_adQ7RueLaFgoBTn53zzfNMaDP5FX-hoFknrs3Sh_57dfcBz37cw_nNLAxzP0sytYhGHpb276YDaZAZ3y2srhB7N5d3D-8QWduLq7l5CrHmXmejeP-230otLPZr5zkZt9iX9Aapy2vbOZ26xqV7S6UIkpCl-c2cBrJVMFuXh4xWeEdPuhV1O4nixmp6p3UZqK6B6QWkVaaJ6qh6rmnNx-_2IdEgdy63OYVKcY1xyENrOBWUckhwWBqYsWQhdEx07jrKQTOycyeQz7Ck-6JIYHiV9yP0xuNgi9hV66qLBvv7WGJyZJlYdcquIc7mH1uHcyZQwrahR-twiLtZP1cN0UZgw';
  let initPhrase = 'Включи квиз-переводчик'; 

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
       return createSmartappDebugger({
        token,
        initPhrase,
        getState,
        settings: {debugging: false}
       })
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