<script>
  import { flip } from 'svelte/animate';
  import { writable, get } from 'svelte/store'
  import Color from 'color';
  import Button from './Button.svelte';

  const localStorageProgressions = localStorage.getItem("savedProgressions");
  const savedProgressionsStore = writable(JSON.parse(localStorageProgressions));

  // save the entire progression on a call to set()
  savedProgressionsStore.subscribe(value => {
    localStorage.setItem("savedProgressions", JSON.stringify(value));
  });

  // initialize the store if it doesn't exist
  if (!$savedProgressionsStore) {
    savedProgressionsStore.set([]);
  }

  /* setup */
  let apiKey = "";

  /* consts */
  const polySynth = new Tone.PolySynth(Tone.Synth).toDestination();
  const notes = [ "C", "C#", "D", "D#", "E", "F", "F#", "G", "G#", "A", "A#", "B" ];

  /* global vars */
  let chords = ["Em7", "A7", "Dmaj7", "Gmaj7", "C#dim7", "F#7", "Bm7"];
  // let chords = ["C", "Dm", "G"];
  let chord = "";
  let rotatedNotes = notes;
  let octave = 4;

  /* Reactive vars */
  $: notesInChords = new Set(chords.flatMap(chordToNotes));
  $: highlightedNotes = [];

  // construct octaves based on rotated notes s.t. pitch increases left to right
  $: octaves = ((rotatedNotes, octave) => {
    return rotatedNotes.map((note, index) => {
      if (note === "C" && index !== 0) {
        octave += 1;
      }
      return octave;
    });
  })(rotatedNotes, octave);

  /* Sound functions */
  const playAll = () => {
    const now = Tone.now();
    chords.forEach((_, idx) => playChord(idx, now + idx));
  };

  const playChord = (idx, time) => {
    const notesInChord = chordToNotes(chords[idx]);
    // Map each note in the chord to 'note + octave' like C4, D5
    const notesToPlay = notesInChord.map(note => note + octaves[rotatedNotes.indexOf(note)]);
    polySynth.triggerAttackRelease(notesToPlay, "4n", time);
  };

  const playNote = (idx) => {
    const newOctave = Number(octaves[idx % 12] + Math.floor(idx / 12));
    polySynth.triggerAttackRelease(rotatedNotes[idx % 12] + "" + newOctave, "4n");
  }

  /* Music theory functions */
  const fromDegrees = (root, degrees) => degrees.map(degree => notes[(notes.indexOf(root) + degree) % 12]);

  const majTriad = (root) => fromDegrees(root, [0, 4, 7]);
  const minTriad = (root) => fromDegrees(root, [0, 3, 7]);
  const dimTriad = (root) => fromDegrees(root, [0, 3, 6]);
  const maj7 = (root) => fromDegrees(root, [0, 4, 7, 11]);
  const min7 = (root) => fromDegrees(root, [0, 3, 7, 10]);
  const dom7 = (root) => fromDegrees(root, [0, 4, 7, 10]);
  // half diminished
  const dim7 = (root) => fromDegrees(root, [0, 3, 6, 10]);
  const fdim7 = (root) => fromDegrees(root, [0, 3, 6, 9]);

  const _chordToNotes = (() => {
    const chordToNotes = {};

    for (let root of notes) {
      chordToNotes[root] = majTriad(root);
      chordToNotes[root + "m"] = minTriad(root);
      chordToNotes[root + "dim"] = dimTriad(root);
      chordToNotes[root + "maj7"] = maj7(root);
      chordToNotes[root + "m7"] = min7(root);
      chordToNotes[root + "7"] = dom7(root);
      chordToNotes[root + "dim7"] = dim7(root);
      chordToNotes[root + "fdim7"] = fdim7(root);
    }
    return chordToNotes;
  })();

  const chordToNotes = (chord) => _chordToNotes[chord];

  /* UI helpers */
  const getHue = (note) => {
    const idx = notes.indexOf(note);
    const hue = Math.sin(idx / 12) * 360;
    return hue;
  };

  // function to take in hue and return it as hex, lightened
  const lightenColor = (hue, amount=.1) => {
    return Color.hsl(hue, 80, 50).lighten(amount).hex(); 
  }

  /* Button handlers */
  const handleAddChord = () => {
    if (chordToNotes(chord) === undefined) {
      return;
    }
    chords.push(chord);
    chords = chords;
    chord = "";
  };

  const handleClear = () => (chords = []);

  const handleSave = () => {
    const progressionsArray = get(savedProgressionsStore);
    progressionsArray.push([...chords]);
    savedProgressionsStore.set(progressionsArray);
  };

  const handleRemoveSavedProgression = (idx) => {
    const progressionsArray = get(savedProgressionsStore);
    progressionsArray.splice(idx, 1);
    savedProgressionsStore.set(progressionsArray);
  };

  const setChords = (progression) => {
    chords = [...progression]
  };

  const handleRemoveChord = (idx) => {
    chords.splice(idx, 1);
    chords = chords;
  };

  const rotateLeftOnce = () => rotatedNotes = [...rotatedNotes.slice(1), rotatedNotes[0]];

  const rotateRightOnce = () => rotatedNotes = [ rotatedNotes[rotatedNotes.length - 1], ...rotatedNotes.slice(0, rotatedNotes.length - 1) ];

  const handleKeyDown = (e) => {
    // check if the text box is not focused
    if (document.activeElement.id === "chord") {
      return;
    }
    if (e.key === "ArrowLeft") {
      rotateLeftOnce();
    } else if (e.key === "ArrowRight") {
      rotateRightOnce();
    } else if (e.key === "ArrowUp") {
      if (octave < 8) {
        octave += 1;
      }
    } else if (e.key === "ArrowDown") {
      if (octave > 0) {
        octave -= 1;
      }
    } else if (e.key === "q" || e.key == "1") {
      playNote(0);
    } else if (e.key === "w") {
      playNote(1);
    } else if (e.key === "e" || e.key === "2") {
      playNote(2);
    } else if (e.key === "r") {
      playNote(3);
    } else if (e.key === "t" || e.key === "3") {
      playNote(4);
    } else if (e.key === "y" || e.key === "4") {
      playNote(5);
    } else if (e.key === "u") {
      playNote(6);
    } else if (e.key === "i" || e.key === "5") {
      playNote(7);
    } else if (e.key === "o") {
      playNote(8);
    } else if (e.key === "p" || e.key === "6") {
      playNote(9);
    } else if (e.key === "[") {
      playNote(10);
    } else if (e.key === "]" || e.key === "7") {
      playNote(11);
    } else if (e.key == "\\" || e.key == "8") {
      playNote(12); // play the root note one octave higher
    }
  }

  /* GPT  handlers */
  const handleJazzify = async () => {
    const prompt = `Jazzify the following chord progression: ${chords.join("")}.
	Return the result as a comma separated list of chords only. Do not respond with anything except the chords.`;

    chords = (await sendRequest(prompt)).choices[0].message.content
      .split(",")
      .map((chord) => chord.trim())
      .map((chord) => chord.replace("min", "m"));
  };

  const handleExtend = async () => {
    const prompt = `Extend the following chord progression: ${chords.join( "")} by adding ${chords.length} more chords.
	Return the original chords and the new chords as a comma separated list of chords only. Do not respond with anything except the chords.`;

    chords = (await sendRequest(prompt)).choices[0].message.content
      .split(",")
      .map((chord) => chord.trim())
      .map((chord) => chord.replace("min", "m"));
  };

  const handleDarken = async () => {
    const prompt = `Darken the following chord progression: ${chords.join("")}.
	Return the result as a comma separated list of chords only. Do not respond with anything except the chords.`;

    chords = (await sendRequest(prompt)).choices[0].message.content
      .split(",")
      .map((chord) => chord.trim())
      .map((chord) => chord.replace("min", "m"));
  };

  const handleBrighten = async () => {
    const prompt = `Brighten the following chord progression: ${chords.join("")}.
	Return the result as a comma separated list of chords only. Do not respond with anything except the chords.`;

    chords = (await sendRequest(prompt)).choices[0].message.content
      .split(",")
      .map((chord) => chord.trim())
      .map((chord) => chord.replace("min", "m"));
  };

  const handleEmbellish = async () => {
    const prompt = `Add embellishments to the following chord progression: ${chords.join( "")}.
	Return the result as a comma separated list of chords only. Do not respond with anything except the chords.`;

    chords = (await sendRequest(prompt)).choices[0].message.content
      .split(",")
      .map((chord) => chord.trim())
      .map((chord) => chord.replace("min", "m"));
  };

  /* GPT helpers */
  const sendRequest = async (prompt) => {
    const req = {
      method: "POST",
      headers: {
        Authorization: "Bearer " + apiKey,
        "content-type": "application/json",
      },
      body: JSON.stringify({
        model: "gpt-3.5-turbo",
        messages: [{ role: "user", content: prompt }],
        max_tokens: 50,
        temperature: 0.7,
      }),
    };

    return fetch("https://api.openai.com/v1/chat/completions", req)
      .then((response) => {
        return response.json();
      })
      .then((data) => {
        return data;
      });
  };
</script>

<div class="flex">
  <!-- Main --> 
  <div class="flex-grow flex items-center justify-center min-h-screen">
    <div class="bg-gray-10 p-8 rounded-lg shadow-lg max-w-xl">
      <div class="flex items-center justify-center">
        <h2 class="text-1xl font-bold mb-4 text-center bg-gradient-to-r from-red-400 via-green-500 to-indigo-400 inline-block text-transparent bg-clip-text">
          A I M P R O V I S E
        </h2>
      </div>
      <form on:submit|preventDefault={() => null} class="flex justify-center mb-4 space-x-4">
        <input bind:value={chord} placeholder="Add Chord" id="chord" class="bg-gray-100 hover:bg-gray-200 text-black mt-3 h-19 px-4 rounded" />
        <Button onClick={handleAddChord} text="Add Chord" id="add-chord" />
        <Button onClick={playAll} text="Play All" id="play-all" />
      </form>
      <div class="flex flex-wrap justify-center mb-4 space-x-4">
        <!-- TODO: make chords a list of objects with unique ids so the animations work -->
        {#each chords as chord, idx (idx)}
          <div class="relative group">
            <button 
               on:click={() => playChord(idx, Tone.now())} 
               on:mouseenter={() => highlightedNotes = chordToNotes(chords[idx])}
               on:mouseleave={() => highlightedNotes = []}
               class="chord-button 
               button w-20 h-12 
               rounded-lg cursor-pointer select-none
               active:translate-y-2  
               active:border-b-[0px]
               transition-all duration-150 
               border-b-[0px] 
               text-gray-700 font-bold text-lg mx-1 my-3"
               style="background-color: hsl({getHue(chord[0])}, 100%, 80%);
               box-shadow: 0px 10px 0px 0px {lightenColor(getHue(chord[0]))},0 15px 0 0 {lightenColor(getHue(chord[0]))}41;
               ">
               {chord}
              </button>
            <button on:click={() => handleRemoveChord(idx)} class="absolute top-0 right-[-8px] p-2 w-4 h-4 font-bold text-gray-700 rounded-full bg-white flex items-center justify-center font-mono opacity-0 group-hover:opacity-100">×</button>
          </div>
        {/each}
      </div>

      <ul class="flex justify-center space-x-2 mb-4 mt-6">
        <button on:click={() => rotateLeftOnce()} class="border-none font-bold list-none rounded-full bg-white-300 w-6 h-6 flex justify-center">{"‹"}</button>
        {#each rotatedNotes as note, idx (note)}
          <div class="" animate:flip={{duration: 100}}>
            {#if notesInChords.has(note)}
              <button on:click={() => playNote(idx)} 
                class="{highlightedNotes.includes(note) ? "border-solid border-2 border-gray-500" : "rounded-full"} 
                transition ease-in-out duration-250 
                font-bold items-center  click:bg-gray-800 list-none w-7 h-7 flex justify-center" 
                style="background-color: hsl({getHue(note)}, 100%, 80%)">{note}</button>
            {:else}
              <button on:click={() => playNote(idx)} class="items-center border-none click:bg-gray-200 list-none rounded-full bg-white-300 w-7 h-7 flex justify-center">{note}</button>
            {/if}
          </div>
        {/each}
        <button on:click={() => rotateRightOnce()} class="border-none font-bold list-none rounded-full bg-white-300 w-6 h-6 flex justify-center">{"›"}</button>
      </ul>

      <div class="flex-wrap flex justify-center">
        <Button onClick={handleJazzify} text="Jazzify" />
        <Button onClick={handleDarken} text="Darken" />
        <Button onClick={handleBrighten} text="Brighten" />
        <Button onClick={handleEmbellish} text="Embellish" />
        <Button onClick={handleExtend} text="Extend" />
        <Button onClick={handleClear} text="Clear" />
        <Button onClick={handleSave} text="Save" />
      </div>
    </div>

    <div class="fixed bottom-1">
      <input bind:value={apiKey} type="password" placeholder="OpenAI API KEY" class="bg-gray-100 hover:bg-gray-200 text-black py-2 px-4 rounded text-center"/>
    </div>
  </div>
  <!-- Sidebar -->
  <div class="w-auto max-w-xl bg-gray-10 p-2 rounded-lg shadow-xl">
    <div class="items-center justify-center flex mt-10 mb-5">
      <h2 class="text-1xl font-bold mb-4 text-center bg-gradient-to-r from-red-400 via-green-500 to-indigo-400 inline-block text-transparent bg-clip-text">
        My progressions
      </h2>
    </div>
    <div class="mb-1 flex-wrap">
      {#each $savedProgressionsStore as progression, i (i)}
        <div class="mb-4 flex items-center">
          <button on:click={() => setChords(progression)} class="items-center border-none font-bold inline list-none rounded-full bg-white-300 m-2 flex justify-center">{"‹"}</button>
          <div class="overflow-auto flex-grow w-96">
            {#each progression as chord}
                <button class="m-2 border-none my-2 p-2 w-20 font-bold rounded" style="background-color: hsl({getHue(chord[0])}, 100%, 80%)">{chord}</button>
            {/each}
          </div>
          <button on:click={handleRemoveSavedProgression(i)} class="items-center border-none m-2 font-bold inline list-none rounded-full bg-white-300 flex justify-center">{"×"}</button>
        </div>
      {/each}
    </div>
  </div>
</div>

<svelte:window on:keydown={handleKeyDown}/>