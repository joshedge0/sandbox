<script lang="ts">
	import { onMount, tick } from 'svelte';
	import ColorPicker from '$lib/components/ColorPicker.svelte';
	import { Plus, Trash2, PencilLine, Check } from '@lucide/svelte';

	interface ColorPalette {
		name: string;
		isEditing?: boolean;
		c1: string;
		c2: string;
		c3: string;
		c4: string;
		c5: string;
	}

    type ColorKey = 'c1' | 'c2' | 'c3' | 'c4' | 'c5';

	onMount(async () => {
		await import('vanilla-colorful');
	});

	let colorPalettesString;
	let colorPalettes: ColorPalette[] = [];

	onMount(() => {
		document.addEventListener('mousedown', handleClick);
		colorPalettesString = localStorage.getItem('colorPalettes');
		if (colorPalettesString && colorPalettesString !== '[]') {
			try {
				colorPalettes = JSON.parse(colorPalettesString) as ColorPalette[];
			} catch (e) {
				console.error('Error parsing color palettes:', e);
			}
		} else {
			colorPalettes = [
				{
					name: 'Color Palette',
					isEditing: false,
					c1: '#F2F7E9',
					c2: '#CCDDB4',
					c3: '#B2D187',
					c4: '#799C4A',
					c5: '#4C593A'
				}
			];
		}
	});

    const colorKeys: ColorKey[] = ['c1', 'c2', 'c3', 'c4', 'c5'];
	let pickerElement: HTMLDivElement | null = null;
	let showPicker = false;
	let pickerPosition = { top: 0, left: 0 };
	let color: string = '';
	let editingIndex: number;
	let editingKey: keyof ColorPalette;
	let firstClick = false;
	let showAlert = false;

	const handleColorClick = (
		e: MouseEvent | KeyboardEvent,
		index: number,
		colorKey: ColorKey
	) => {
		const rect = (e.currentTarget as HTMLDivElement).getBoundingClientRect();
		let top = rect.top + window.scrollY;
		let left = rect.right + window.scrollX + 8;
		color = colorPalettes[index][colorKey];

		if (top === pickerPosition.top && left === pickerPosition.left && firstClick) {
			firstClick = false;
		} else {
			pickerPosition.top = top;
			pickerPosition.left = left;
			showPicker = true;
			editingIndex = index;
			editingKey = colorKey;
		}
	};

	const handleColorChange = (newColor: string) => {
		if (editingIndex !== null && editingKey !== null) {
			const updatedPalettes = [...colorPalettes];
			updatedPalettes[editingIndex] = {
				...updatedPalettes[editingIndex],
				[editingKey]: newColor
			};
			colorPalettes = [...updatedPalettes];
			color = newColor;
		}
	};

	const handleInputChange = (e: Event, index: number, key: ColorKey) => {
		const inputValue = (e.target as HTMLInputElement).value.toUpperCase().replace(/[^0-9A-F]/g, '');
		const updatedPalettes = [...colorPalettes];
		updatedPalettes[index][key] = `#${inputValue}`;
		colorPalettes = [...updatedPalettes];
	};

	const handleClick = (e: MouseEvent) => {
		if (showPicker) {
            const target = e.target as HTMLDivElement;
        const rect = target.getBoundingClientRect();
			let top = rect.top + window.scrollY;
			let left = rect.right + window.scrollX + 8;

			if (!pickerElement?.contains(target)) {
				if (top === pickerPosition.top && left === pickerPosition.left) {
					firstClick = true;
				}
				showPicker = false;
			}
		}
	};

	const handleAdd = () => {
		const newPalette: ColorPalette = {
			name: 'Color Palette',
			isEditing: false,
			c1: randomHexGen('FFFFFF', 'BBBBBB'),
			c2: randomHexGen('BBBBBB', '777777'),
			c3: randomHexGen('777777', '444444'),
			c4: randomHexGen('444444', '111111'),
			c5: randomHexGen('111111', '000000')
		};
		colorPalettes = [...colorPalettes, newPalette];
	};

	const handleDelete = (index: number) => {
		let updatePalettes = [...colorPalettes];
		updatePalettes.splice(index, 1);
		colorPalettes = [...updatePalettes];
	};

	const handleSave = () => {
		localStorage.setItem('colorPalettes', JSON.stringify(colorPalettes));
		showAlert = true;
		setTimeout(() => {
			showAlert = false;
		}, 2000);
	};

	const randomHexGen = (min: string, max: string): string => {
		const minInt = parseInt(min, 16);
		const maxInt = parseInt(max, 16);

		// Generate random integer in range
		const rand = Math.floor(Math.random() * (maxInt - minInt + 1)) + minInt;

		// Convert back to hex and pad with zeros
		return `#${rand.toString(16).padStart(6, '0')}`;
	};
</script>

<div class="flex flex-col">
	<h1 class="mx-auto text-3xl font-bold mt-[40px]">colors</h1>

	<div class="w-[50%] mx-auto pt-2 pb-4 border-solid">
		{#each colorPalettes as palette, index}
			<div class="my-2">
				{#if !palette.isEditing}
					<div class="flex">
						<div class="min-w-[150px]">
							<h1 class="font-medium w-full">{palette['name']}</h1>
						</div>
						<PencilLine
							class="pl-2 w-[25px]"
							onclick={async () => {
								palette.isEditing = true;
								await tick();
								document.getElementById(`paletteNameInput${index}`)?.focus();
							}}
						/>
					</div>
				{:else}
					<div class="flex">
						<input id="paletteNameInput{index}" 
                        class="w-[150px]" 
                        bind:value={palette.name}
                        onfocusout={() => {
                            palette.isEditing = false;

                        }} />
					</div>
				{/if}

				<div class="flex flex-row justify-around items-center py-6">
					{#each colorKeys as colorKey}
						<div class="pr-4 flex flex-col items-center">
							<div
								class="w-[100px] h-[100px] rounded"
								style="background-color: {palette[colorKey]}"
								role="button"
								tabindex="0"
								onclick={(e) => handleColorClick(e, index, colorKey)}
								onkeydown={(e) =>
									(e.key === 'Enter' || e.key === ' ') && handleColorClick(e, index, colorKey)}
							></div>

							<div class="flex items-center space-x-2 mt-2">
								<input
									class="input w-[100px] text-center"
									type="text"
									maxLength={7}
									value={palette[colorKey]}
									onchange={(e) => handleInputChange(e, index, colorKey)}
								/>
							</div>
						</div>
					{/each}
					<Trash2 class="mb-[40px] w-[22px] h-[22px]" onclick={() => handleDelete(index)} />
				</div>
			</div>
		{/each}
		<button class="btn w-full" onclick={handleAdd}><Plus />Add Palette</button>
		<button class="btn btn-success mt-4 w-full" onclick={handleSave}>Save</button>
	</div>
</div>

{#if showAlert}
	<div
		role="alert"
		class="alert alert-success w-[50%] mx-auto transform -translate-x-1/2 transition-all opacity-100 animate-fadeOut"
	>
		<svg
			xmlns="http://www.w3.org/2000/svg"
			class="h-6 w-6 shrink-0 stroke-current"
			fill="none"
			viewBox="0 0 24 24"
		>
			<path
				stroke-linecap="round"
				stroke-linejoin="round"
				stroke-width="2"
				d="M9 12l2 2 4-4m6 2a9 9 0 11-18 0 9 9 0 0118 0z"
			/>
		</svg>
		<span>Save Complete!</span>
	</div>
{/if}

<ColorPicker
	{color}
	onColorChange={handleColorChange}
	bind:showPicker
	bind:pickerPosition
	setRef={(el) => (pickerElement = el)}
/>

<style>
	@keyframes fadeOut {
		0% {
			opacity: 1;
		}
		90% {
			opacity: 1;
		}
		100% {
			opacity: 0;
		}
	}

	.animate-fadeOut {
		animation: fadeOut 1.5s ease-in-out forwards;
	}
</style>
