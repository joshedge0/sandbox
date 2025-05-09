<script lang="ts">
	export let color: string;
	export let onColorChange: (color: string) => void;
	export let showPicker: boolean;
	export let pickerPosition: { top: number; left: number };
	export let setRef: (el: HTMLDivElement) => void;
	let container: HTMLDivElement;

	$: if (container && typeof setRef === 'function') {
		setRef(container);
	}

	function handleColorChanged(event: CustomEvent<{ value: string }>) {
		onColorChange?.(event.detail.value);
	}
</script>

{#if showPicker}
	<div
		bind:this={container}
		class="absolute z-10"
		style="top: {pickerPosition.top}px; left: {pickerPosition.left}px;"
	>
		<hex-color-picker {color} on:color-changed={handleColorChanged}></hex-color-picker>
	</div>
{/if}
