<script lang="ts">
	import { onMount } from 'svelte';
	import ColorPicker from '$lib/components/ColorPicker.svelte';
    import { Plus, Trash2 } from '@lucide/svelte';

	onMount(async () => {
		await import('vanilla-colorful');
	});

	onMount(() => {
		document.addEventListener('mousedown', handleClick);
	});

	interface ColorPalette {
		c1: string;
		c2: string;
		c3: string;
		c4: string;
		c5: string;
	}

	const initialColorPalettes: ColorPalette[] = [
		{ c1: '#37413A', c2: '#5c7f71', c3: '#DFECE4', c4: '#DFDE9B', c5: '#E15D44' },
		{ c1: '#4a3c34', c2: '#B9714F', c3: '#C5A253', c4: '#D5CB9F', c5: '#E1907F' }
	];

	const colorKeys: (keyof ColorPalette)[] = ['c1', 'c2', 'c3', 'c4', 'c5'];

	let colorPalettes = initialColorPalettes;
	let pickerElement: HTMLDivElement | null = null;
	let showPicker = false;
	let pickerPosition = { top: 0, left: 0 };
	let color = '#000000';
	let editingIndex: number;
	let editingKey: keyof ColorPalette;
	let firstClick = false;

	const handleColorClick = (
		e: MouseEvent | KeyboardEvent,
		index: number,
		colorKey: keyof ColorPalette
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

    const handleInputChange = (
    e: Event,
    index: number,
    key: keyof ColorPalette
  ) => {
    const inputValue = (e.target as HTMLInputElement).value.toUpperCase().replace(/[^0-9A-F]/g, "");
    const updatedPalettes = [...colorPalettes];
    updatedPalettes[index][key] = `#${inputValue}`;
    colorPalettes = [...updatedPalettes];
  };

	const handleClick = (e: MouseEvent) => {
		const target = e.target as HTMLDivElement;

		const rect = target.getBoundingClientRect();
		let top = rect.top + window.scrollY;
		let left = rect.right + window.scrollX + 8;

		if (!pickerElement?.contains(target) && showPicker) {
			if (top === pickerPosition.top && left === pickerPosition.left) {
				firstClick = true;
			}
			showPicker = false;
		}
	};

    const handleAddPalette = () => {
        const newPalette: ColorPalette = {
            c1: '#EEE',
            c2: '#BBB',
            c3: '#777',
            c4: '#444',
            c5: '#111'
        };
        colorPalettes = [...colorPalettes, newPalette];
    }

    const handleDelete = (index: number) => {
        let updatePalettes = [...colorPalettes]
        updatePalettes.splice(index, 1);
        colorPalettes = [...updatePalettes]
    }

    const handleSave = () => {
        console.log('saved')
    }

</script>
<div class="flex flex-col">
<h1 class="mx-auto text-3xl font-bold mt-[40px]">colors</h1>

<div class="w-[50%] mx-auto pt-2 pb-[100px] border-solid">
	{#each colorPalettes as palette, index}
		<div class="my-2">
			<h1 class="text-l font-medium">Color Palette #{index + 1}</h1>
			<div class="flex flex-row justify-around items-center py-6 ">
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
								maxLength={6}
								value={palette[colorKey]}
                                onchange={(e) => handleInputChange(e, index, colorKey)}
							/>
						</div>
					</div>
				{/each}
                <Trash2 class="mb-[40px] w-[25px] h-[25px]" onclick={() => handleDelete(index)}/>
			</div>
		</div>
	{/each}
    <button class="btn w-full" onclick={handleAddPalette}><Plus />Add Palette</button>
    <button class="btn btn-success mt-4 w-full" onclick={handleSave}>Save</button>
</div>
</div>

<ColorPicker
	{color}
	onColorChange={handleColorChange}
	bind:showPicker
	bind:pickerPosition
	setRef={(el) => (pickerElement = el)}
/>
