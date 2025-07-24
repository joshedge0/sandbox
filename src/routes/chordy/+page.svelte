<script lang="ts">
	const strings = ['E', 'A', 'D', 'G', 'B', 'E'];
	const frets = Array.from({ length: 13 }, (_, i) => i);
	let selectedFrets: Record<number, number> = {};

	const toggleFret = (stringIndex: number, fret: number) => {
		if (selectedFrets[stringIndex] == fret) {
			delete selectedFrets[stringIndex];
		} else {
			selectedFrets[stringIndex] = fret;
		}

		selectedFrets = { ...selectedFrets };
	};

	const clearAll = () => {
		console.log('clearing');
		selectedFrets = {};
	};

	const getSelectedPattern = () => {
		const pattern: Array<{
			string: string;
			stringIndex: number;
			fret: number;
			note: string;
		}> = [];

		Object.entries(selectedFrets).forEach(([stringIndex, fret]) => {
			const stringIdx = parseInt(stringIndex);
			pattern.push({
				string: strings[stringIdx],
				stringIndex: stringIdx,
				fret: fret,
				note: getNoteAtFret(stringIdx, fret)
			});
		});

		return pattern.sort((a, b) => a.stringIndex - b.stringIndex);
	};

	// Chromatic scale and hardcoded to E standard tuning
	const getNoteAtFret = (stringIndex: number, fret: number) => {
		const notes = ['C', 'C#', 'D', 'D#', 'E', 'F', 'F#', 'G', 'G#', 'A', 'A#', 'B'];
		const openStringNotes = [4, 9, 2, 7, 11, 4]; // E, A, D, G, B, E in semitones from C, make variable for other tunings
		const noteIndex = (openStringNotes[stringIndex] + fret) % 12;
		return notes[noteIndex];
	};
</script>

<div class="flex flex-col">
	<h1 class="mx-auto text-3xl font-bold mt-[40px] mb-24">whats my chord?</h1>

	<!-- Guitar Neck -->
	<div class="w-fit mx-auto bg-[#c9a168] p-4 rounded-lg shadow-lg overflow-x-scroll">
		<div class="min-w-max">
			<!-- Fret numbers -->
			<div class="flex mb-2">
				<div class="w-12 text-center text-amber-100 font-semibold text-sm">String</div>
				{#each frets as fret}
					<div class="w-16 text-center text-amber-100 font-semibold text-sm">
						{fret === 0 ? 'Open' : fret}
					</div>
				{/each}
			</div>

			<!-- Strings/Frets -->
			{#each strings as stringNote, stringIndex}
				<div class="flex items-center mb-1 relative">
					<!-- String names -->
					<div class="w-12 text-center text-amber-100 font-bold bg-[#8f6d3e] rounded py-2">
						{stringNote}
					</div>

					<!-- Fret line -->
					<div class="absolute left-12 right-0 h-0.5 bg-gray-300" style:top="50%"></div>

					<!-- Frets -->
					{#each frets as fret (fret)}
						{@const isSelected = selectedFrets[stringIndex] === fret}
						{@const note = getNoteAtFret(stringIndex, fret)}

						<div class="w-16 flex flex-col items-center relative">
							<!-- Fret wire (except for open string) -->
							{#if fret > 0}
								<div class="absolute left-0 h-11 w-0.5 bg-gray-400"></div>
							{/if}

							<!-- Checkbox styled as fret dot -->
							<label class="cursor-pointer flex flex-col items-center py-1">
								<input
									type="checkbox"
									class="sr-only"
									checked={isSelected}
									on:change={() => toggleFret(stringIndex, fret)}
								/>
								<div
									class="w-6 h-6 rounded-full border-2 flex items-center justify-center text-xs font-semibold transition-all
                                {isSelected
										? 'bg-blue-500 border-blue-600 text-white shadow-lg scale-110'
										: 'bg-amber-200 border-amber-400 hover:bg-amber-100'}"
								>
									{isSelected ? note : ''}
								</div>
							</label>

							<!-- Fret position markers -->
							{#if stringIndex === 2 && [3, 5, 9].includes(fret)}
								<div class="absolute bottom-0 w-2 h-2 bg-amber-900 rounded-full -mb-[6px]"></div>
							{/if}
							{#if stringIndex === 2 && [7, 12].includes(fret)}
								<div class="absolute bottom-0 w-2 h-2 bg-amber-900 rounded-full -mb-[50px]"></div>
								<div class="absolute bottom-0 w-2 h-2 bg-amber-900 rounded-full mb-[38px]"></div>
							{/if}
						</div>
					{/each}
				</div>
			{/each}
		</div>
	</div>

        <button
		on:click={() => clearAll()}
		class="bg-red-600 mt-8 hover:bg-red-800 text-white px-4 py-2 rounded transition-colors w-48 mx-auto"
	>
		Clear All
	</button>

    <!-- Display selected frets -->
     {#if Object.keys(selectedFrets).length > 0}

        <div class="mt-2 p-4 bg-white rounded-lg flex flex-col">
          <h3 class="text-lg font-semibold text-gray-800 mb-3 mx-auto">Selected Frets</h3>
          <div class="grid grid-cols-3 gap-2 mx-auto w-fit mx-auto">
                 {#each getSelectedPattern() as pattern}

              <div class="bg-blue-50 w-64 px-3 py-2 rounded text-sm">
                <span class="font-semibold">{pattern.string} string</span> - 
                <span class="ml-1">Fret {pattern.fret}</span> - 
                <span class="ml-1 text-blue-600 font-semibold">{pattern.note}</span>
              </div>
            {/each}
          </div>
        </div>
        
        {/if}


</div>
