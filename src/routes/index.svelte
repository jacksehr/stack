<script lang="ts">
	import { dev } from '$app/env';
	import { writable } from 'svelte/store';
	import { createClient } from '@supabase/supabase-js';
	import { debounce } from 'lodash-es';
	import { onDestroy } from 'svelte';

	let stack: { id: string; text: string }[] = [];
	let currentId = '';

	// Create a single supabase client for interacting with your database
	const supabase = createClient(
		'https://iieojksehfiegxvfdxah.supabase.co',
		'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6ImlpZW9qa3NlaGZpZWd4dmZkeGFoIiwicm9sZSI6ImFub24iLCJpYXQiOjE2NTE0MjkxMjYsImV4cCI6MTk2NzAwNTEyNn0.j2kcCyqZbvChFnz6ZgyTT_R2DPABRcPlgVjigeMaqLg'
	);

	const useSupabase = () => supabase.from<{ user_id: string; text: string; id: string }>('stack');

	let textbox: HTMLDivElement;
	const focusTextbox = () => textbox.focus();

	let user = supabase.auth.user();

	const currentText = writable('');
	const saveText = async (value: string) => {
		await useSupabase().upsert({ id: currentId || undefined, user_id: user?.id, text: value });
	};
	const debouncedSaveText = debounce(saveText, 500);
	let unsubscribe = currentText.subscribe(debouncedSaveText);

	// for wiping the note
	const reset = ({ id, text }: { id?: string; text: string }) => {
		unsubscribe();

		currentId = id ?? '';
		currentText.set(text);

		unsubscribe = currentText.subscribe(debouncedSaveText);
	};

	const refreshStack = async () => {
		if (user) {
			const { data, error } = await useSupabase().select().filter('user_id', 'eq', user.id);

			if (error) {
				throw error;
			}

			stack = [...data];
			const newData = data.pop();
			if (!newData) {
				currentId = '';
				return reset({ text: '' });
			}

			const { id, text } = newData;
			reset({ id, text });
			currentId = id;
		}
	};

	refreshStack();

	supabase.auth.onAuthStateChange(() => {
		user = supabase.auth.user();
		refreshStack();
	});

	onDestroy(unsubscribe);
</script>

<svelte:head>
	<title>Stack</title>
</svelte:head>

{#if !user}
	<h1
		role="button"
		on:click={async () => {
			await supabase.auth.signIn(
				{
					provider: 'google'
				},
				{ redirectTo: dev ? 'http://localhost:3000' : undefined }
			);
		}}
	>
		Sign in
	</h1>
{:else}
	<div class="container">
		<h1>Stack.</h1>
		<div class="card" on:click={focusTextbox}>
			<div
				bind:this={textbox}
				class="editable-text"
				role="textbox"
				contenteditable="true"
				bind:textContent={$currentText}
			/>
		</div>
		<div class="toolbar">
			<img
				class="icon"
				src="/icons/tick.svg"
				alt="complete-note"
				role="button"
				on:click={async () => {
					const itemToComplete = stack.pop();
					if (!itemToComplete) return;

					await useSupabase().delete().match({ id: itemToComplete.id });

					const newHead = stack[stack.length - 1];
					if (!newHead) {
						reset({ text: '' });
					} else {
						const { id, text } = newHead;
						reset({ id, text });
					}

					focusTextbox();
				}}
			/>
			<img
				class="icon"
				src="/icons/add.svg"
				alt="add-note"
				role="button"
				on:click={async () => {
					if (!$currentText) return;

					const { data } = await useSupabase().insert({ text: '', user_id: user?.id });

					if (!data?.length) return;

					reset({ text: '', id: data[0].id });

					focusTextbox();
				}}
			/>
		</div>
	</div>
{/if}

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
