<script lang="ts">
	import { onMount } from 'svelte';
	import { desiredString } from '../desiredString';
	import PopUp from '$lib/components/LIR/PopUp.svelte';

	let active = 0;
	let bootStatus = 'Automatic';
	let focusOn = 'textarea';
	let textareaElement: HTMLTextAreaElement;
	let inputElement: HTMLInputElement;
	let textareaValue = ``;
	let inputValue = '';
	const chiperCode = 'olssv dvysk'; // chiper == hello world
	let successPopUp = false;

	function convertString(str: string | any[]) {
		let result = '';
		const regex = /[^\s]/;
		for (let i = 0; i < str.length; i++) {
			if (regex.test(str[i])) {
				result += str[i];
			}
		}
		console.log(result);
		
		return result;
	}

	function checkForSuccess() {
		let str1: string;
		setTimeout(() => {
			str1 = textareaValue;
		}, 500);

		setTimeout(() => {
			textareaValue += `
	
compiling ...`;
		}, 500);

		setTimeout(() => {
			if (convertString(str1) === convertString(desiredString)) {
				textareaValue += `
signature : ${chiperCode}`;
			} else {
				textareaValue += `
failed`;
				setTimeout(() => {
					textareaValue = '';
				}, 2000);
			}
		}, 2000);
	}

	const changeFocus = () => {
		if (focusOn === 'textarea') {
			focusOn = 'authinput';
			inputElement.focus();
		} else {
			focusOn = 'textarea';
			textareaElement.focus();
		}
	};

	async function pasteFromClipboard() {
		const text = (await navigator.clipboard.readText()).trim();
		textareaValue = text;
	}

	function convertToManual() {
		setTimeout(() => {
			if (inputValue.trim().toLowerCase() === chiperCode.trim().toLowerCase()) {
				bootStatus = 'Manual';
				textareaValue += `
succesfull`;
				// Pop Up
				successPopUp = true;
				setTimeout(() => {
					successPopUp = false;
				},2000)
			} else {
				textareaValue += `
failed`;
				setTimeout(() => {
					textareaValue = '';
				}, 2000);
			}
		},500)
	}

	function testKey(charCode: any) {
		const code = charCode;
		charCode = charCode.slice(0, -1);
		if (charCode === 'Digit' || charCode === 'Key' || charCode === 'Numpad' || code === 'Backspace' || code === 'Space') {
			return false;
		}
		return true;
	}

	onMount(() => {
		document.addEventListener('keydown', function (event) {
			if (event.key === 'Tab') {
				event.preventDefault();
				changeFocus();
			} else if (event.key === 'Escape') {
				event.preventDefault();
				window.location.href = '/serverRoom';
			} else if (event.key === 'Enter') {
				event.preventDefault();
				if (focusOn === 'textarea') {
					if (textareaValue.trimEnd() === '') {
						pasteFromClipboard();
					}
					checkForSuccess();
				} else {
					convertToManual();
				}
				changeFocus();
			} else if (testKey(event.code)) {
				event.preventDefault();
			}
		});
	});

	$: {
    	if (textareaElement && textareaValue) {
			setTimeout(() => {
				textareaElement.scrollTop = textareaElement.scrollHeight;
			},10)
		}
  	}

</script>

<section class="h-screen w-full cursor-none bg-[#9c9a9d] font-IBM">

	{#if successPopUp}
		<PopUp title={"Success"} message={'You have successfully changed the Boot Mode to Manual'} />
	{/if}

	<header class="flex h-[10%] w-full items-center justify-center bg-[#000069]">
		<div class="flex h-20 w-full items-center justify-center border border-x-4 border-white">
			<h1 class="text-center text-4xl font-medium text-white">Boot Mode</h1>
		</div>
	</header>
	<body class="flex h-[74%] w-full flex-nowrap bg-inherit text-3xl font-[600] tracking-normal text-black">
		<div class="flex h-full w-full flex-col">
			<div class="flex w-full flex-row justify-evenly">
				<h3>Boot status : {bootStatus}</h3>
			</div>
			<div class="flex w-full justify-center">
				<h2>To unlock manual <u>paste the right code</u></h2>
			</div>
			<div class="flex h-[26rem] w-full flex-col items-center justify-center">
				<textarea id="terminal" cols="30" rows="14" bind:value={textareaValue} bind:this={textareaElement} class="h-3/4 w-2/4 cursor-none resize-none bg-black font-normal" autocomplete="off" autocorrect="off" autocapitalize="off" spellcheck="false"></textarea>
			</div>
			<!-- auth code -->
			<div class="flex w-full flex-row items-center justify-center">
				<h3>Auth signature :</h3>
				<h3><input type="text" bind:this={inputElement} bind:value={inputValue} class="cursor-none bg-black px-2 py-1 font-normal text-white" autocomplete="off" autocorrect="off" autocapitalize="off" spellcheck="false"/></h3>
			</div>
		</div>
	</body>
	<footer class="flex h-[16%] w-full items-center justify-center border bg-[#000069] text-white">
		<div class="flex h-32 w-full items-center justify-center border border-x-4 border-white">
			<div class="flex h-[65%] w-full items-end justify-between text-nowrap bg-black px-6">
				<p class="text-3xl tracking-widest">&#x2191&#x2193=Move Highlight</p>
				<p class="text-3xl tracking-widest">&ltEnter&gt=Complete Entry</p>
				<p class="text-3xl tracking-widest">ESC=Exit Entry</p>
			</div>
		</div>
	</footer>
</section>

<style>
	textarea {
		caret: white;
	}
	input {
		caret: black;
		outline: none;
	}

	#terminal {
		background-color: #000000;
		color: #eeeeec;
		border: none;
		padding: 10px;
		overflow: auto;
		scrollbar-width: none;
	}

	textarea::-webkit-scrollbar {
		display: none; /* For Chrome, Safari and Opera */
	}
</style>
