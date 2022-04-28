<script lang="ts">
	import { onDestroy, onMount } from 'svelte';
  import { browser } from '$app/env';
	import { writable } from 'svelte/store';

	const CACHE_KEY = 'notes-cache-key';

	let text = '';

	let textbox: HTMLDivElement;
	const focusTextbox = () => textbox.focus();

  let initStack: string[] = [];
  try {
    if (browser) {
      initStack = JSON.parse(window.localStorage.getItem(CACHE_KEY) || "[]");
    }
  } catch(err) {
    console.error(err);
  }

  const stack = writable<string[]>(initStack);
  onMount(() => {
    stack.subscribe((value) => {
      console.log({ value, browser })
      if (!browser) return;

      window.localStorage.setItem(CACHE_KEY, JSON.stringify(value));
    });
  });
</script>

<svelte:head>
	<title>Stack</title>
</svelte:head>

<div class="container">
	<h1>Stack.</h1>
	<div class="card" on:click={focusTextbox}>
		<div
			bind:this={textbox}
			class="editable-text"
			role="textbox"
			contenteditable="true"
			bind:textContent={text}
		/>
	</div>
	<div class="toolbar">
		<img
			class="icon"
			src="/icons/tick.svg"
			alt="complete-note"
			role="button"
			on:click={() => {
        if ($stack.length == 0) return;

				text = $stack.pop() ?? '';
        $stack = $stack.slice(0, -1);

				focusTextbox();
			}}
		/>
		<img
			class="icon"
			src="/icons/add.svg"
			alt="add-note"
			role="button"
			on:click={() => {
				if (!text) return;

				$stack = [...$stack, text];
				text = '';

				focusTextbox();
			}}
		/>
	</div>
</div>

<style>
	@font-face {
		font-family: Poppins;
		src: url('/Poppins.ttf') format('truetype'), local('Helvetica'), local('Arial');
		font-display: swap;
		font-weight: normal;
	}

	@font-face {
		font-family: Poppins;
		src: url('/Poppins-Light.ttf') format('truetype'), local('Helvetica'), local('Arial');
		font-display: swap;
		font-weight: 300;
	}

	:global(body) {
		box-sizing: border-box;
		margin: 0;

		background-color: #49525b;
		color: #ebfff9;
	}

	* {
		font-family: Poppins, sans-serif;
	}

	h1 {
		font-weight: 300;
		text-shadow: 1px 3px #24292c44;
	}

	.container {
		height: 100vh;
		width: 100vw;

		display: flex;
		flex-direction: column;
		justify-content: center;
		align-items: center;
	}

	.card {
		background-color: #dea47e;
		height: 17.5rem;
		width: 17.5rem;
		box-shadow: 4px 4px 2px #24292c44;
		font-size: 1.25rem;

		color: #380517;

		padding: 1.25rem;
		display: flex;
		align-items: center;
		justify-content: center;

		overflow-y: scroll;

		scrollbar-gutter: stable;
	}

	.editable-text {
		outline: none;
		min-height: 1rem;
		min-width: 1rem;
	}

	.toolbar {
		display: flex;
		justify-content: center;
		padding: 1.5rem 0;
		gap: 1rem;
		width: 17.5rem;
	}

	.icon {
		transition: none;
		-webkit-transition: -webkit-filter 100ms ease-in;

		cursor: pointer;

		filter: invert(89%) sepia(13%) saturate(216%) hue-rotate(96deg) brightness(109%) contrast(102%)
			drop-shadow(1px 1px 1px #24292c);
		height: 2rem;
		width: 2rem;

		user-select: none;
	}

	.icon:hover {
		filter: invert(89%) sepia(13%) saturate(216%) hue-rotate(96deg) brightness(109%) contrast(102%)
			drop-shadow(1px 1px 2px #16191b);
	}
</style>
