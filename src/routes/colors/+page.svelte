<script lang="ts">
	import { onMount } from 'svelte';
	import ColorPicker from '$lib/components/ColorPicker.svelte';

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
		{ c1: '#4a3c34', c2: '#B9714F', c3: '#C5A253', c4: '#D5CB9F', c5: '#E1907F' },
		{ c1: '#3d2d39', c2: '#958B84', c3: '#CCC6BB', c4: '#A2789C', c5: '#7C2946' },
		{ c1: '#00819D', c2: '#87C2D4', c3: '#DDE5ED', c4: '#F9B5A9', c5: '#E8593c' },
		{ c1: '#2C5234', c2: '#BBC592', c3: '#E6EDE4', c4: '#C5AECF', c5: '#404466' },
		{ c1: '#204402', c2: '#7BB369', c3: '#CCDDB4', c4: '#f7f2b0', c5: '#ad507f' }
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
			colorPalettes = updatedPalettes;
			color = newColor;
		}
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
</script>

<h1>colors</h1>

<div class="card w-[50%] mx-auto pt-8 pb-[100px] border-solid">
	{#each colorPalettes as palette, index}
		<div class="card-body">
			<h1 class="card-title">Color Palette #{index + 1}</h1>
			<div class="flex justify-around py-6 flex-row">
				{#each colorKeys as colorKey}
					<div>
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
							<span class="text-sm font-medium">#</span>
							<input
								class="input"
								type="text"
								maxLength={6}
								value={palette[colorKey].replace('#', '')}
							/>
						</div>
					</div>
				{/each}
			</div>
		</div>
	{/each}
</div>

<ColorPicker
	{color}
	onColorChange={handleColorChange}
	bind:showPicker
	bind:pickerPosition
	setRef={(el) => (pickerElement = el)}
/>
