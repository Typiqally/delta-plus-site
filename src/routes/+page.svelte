<div class="h-screen text-sm text-white flex flex-col content-between" on:click={() => terminalInput.focus()}>
	{#each history as line}
		<div class="whitespace-pre">{line}</div>
	{/each}
	{#if !processing}
		<div class="flex items-center">
			<div class="whitespace-nowrap mr-2">typically@macos ~ %</div>
			<input bind:this={terminalInput} class="w-full bg-transparent focus:outline-none" on:keydown={handleCommandLineKeyDown} />
		</div>
	{/if}
</div>

<script lang="ts">
	import { onMount } from 'svelte';
	import { messageOfTheDay } from '../motd';

	let terminalInput: HTMLInputElement;
	let processing = false;

	let history: string[] = [];
	let commandHistoryPointer: number = 0;
	let commandHistory: string[] = [];

	onMount(() => {
		history = [...history, messageOfTheDay];
		terminalInput.focus();
	});

	const appendHistory = (value: string) => history = [...history, value];

	const handleCommandLineKeyDown = (event: KeyboardEvent) => {
		switch (event.code) {
			case 'Enter':
				handleCommandEnter();
				break;
			case 'ArrowUp':
				handleCommandHistoryLookup('up');
				break;
			case 'ArrowDown':
				handleCommandHistoryLookup('down');
				break;
		}
	};

	const handleCommandHistoryLookup = (mode: 'up' | 'down') => {
		const isAtFirstCommand = commandHistoryPointer <= 1;
		const isAtLastCommand = commandHistoryPointer >= commandHistory.length;

		if (isAtFirstCommand && mode === 'down') {
			commandHistoryPointer = 0;
			terminalInput.value = '';
			return;
		}

		if (isAtLastCommand && mode === 'up') {
			return;
		}

		commandHistoryPointer += (mode === 'up') ? 1 : -1;
		terminalInput.value = commandHistory[commandHistoryPointer - 1];
	};

	const handleCommandEnter = () => {
		const command = terminalInput.value;

		appendHistory(`typically@macos ~ % ${command}`);
		terminalInput.value = '';

		handleCommand(command);
	};

	const handleCommand = (command: string) => {
		commandHistoryPointer = 0;
		commandHistory = [command, ...commandHistory];

		switch (command) {
			case 'clear':
			case 'cls':
				history = []
				break;
			default:
				appendHistory(`zsh: command not found: ${command}`);
				break;
		}
	};
</script>

<style lang="postcss">
    :global(html) {
        background-color: theme(colors.stone.900);
    }
</style>
