<link rel="preconnect" href="https://fonts.gstatic.com">
<link href="https://fonts.googleapis.com/css2?family=Montserrat:wght@300;700&display=swap" rel="stylesheet">

<script>
  import {createSmartappDebugger, createAssistant} from '@sberdevices/assistant-client'
  import { onMount } from 'svelte'

  let message = '';
  const backendUrl = 'https://tolmatch-backend.herokuapp.com/'
  let wordsState = []
  let idState
  let token = 'eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiI3NmI4MjA4OWJiNzJkMTA3NGMzMWIwNmQ1YWVmMjE0NDg4NjhjYjZkZTUxZDM4NTE0ZDY1ZjZlYTJmOTgxOTlhNTM5YmU5MjcwMDQyNjI5OCIsImF1ZCI6IlZQUyIsImV4cCI6MTYyMjAzNDY2MiwiaWF0IjoxNjIxOTQ4MjUyLCJpc3MiOiJLRVlNQVNURVIiLCJ0eXBlIjoiQmVhcmVyIiwianRpIjoiODdiOTczYjktZWQ3Ni00ZTc3LWE0NzMtNmQ4YWFkMTQ3MmE0Iiwic2lkIjoiZTRkZGY0YWQtZGZmNy00N2I3LWE4OWItNzZjZjVmM2Q2YmEwIn0.eLLr8w6FCKNKt3gjRtDlbFxg0XIIaTeV3KpXGFKz3zrXEjWSmxqcxYsao-MJzYYrVPwRm6srllzLlXJfJjGNunL9xJ6f-AxOgsGhwj9FRuSZB0VZhb67S6GTVfDNKmVXkjJKOn6HBKBo9w4zcNyQxs9Sb5uTrJh-ncz7y2rZTg3bjIr62mxCB8Pjo5nyySCmUfJgnahKI0cyFlD3epfXBu3E_4sNVV533v7uD0DGfM7MHJAZcosCpUSsVG4Cuge1Pi9_rus_qw8sD8rSSoBls3w0eVoIjOf9yli5Aeovaa5e2OFA7vnebvoKQXx1ZUMRkcC85NWvPTBfz09LB51TAEiyOcqp2LGaiJw2c04jssVTUu4zWtrrVf4EheWb86E4R_Znqb7OpLcuvF-tWwJYtscBH41Je6FUwOwJr6iupAGAQRvZpfRdDxr6RPy3EBvgdCGYzGcW3_vAE-qi1nf6Kr1bkvvYeWKgOI5VfT-WPm_TkubbJtW8bQgR8EbReJBTcnLCeKmdafCmyWcgLLk8GGHr_CEtpl47sd3K-kzlLE2yyR7zWDoCHB05y9GErieG-4vCtTmwjlpGXx_cZMJM-o3Kujgn4hELX_nNCPXlWHOaWvkFgXbYy4gMQOrUKiIQfV1QfQtB0EvPzEQ2Bum2Ij65_xBOFqQtX5Szn7x0pEY';
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
      return createSmartappDebugger({
        token,
        initPhrase,
        getState,
        settings: {debugging: false}
      })
      // return createAssistant({getState});
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