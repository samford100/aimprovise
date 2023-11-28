<script>
  /* setup */
  let apiKey = "";

  /* consts */
  const polySynth = new Tone.PolySynth(Tone.Synth).toDestination();
  const notes = [ "C", "C#", "D", "D#", "E", "F", "F#", "G", "G#", "A", "A#", "B" ];

  /* global vars */
  let chords = ["G", "Em", "C", "D"];
  let chord = "";
  let rotatedNotes = notes;

  /* Reactive vars */
  $: notesInChords = new Set(chords.flatMap(chordToNotes));

  // construct octaves based on rotated notes s.t. pitch increases left to right
  $: octaves = ((rotatedNotes) => {
    let octave = 4;
    return rotatedNotes.map((note, index) => {
      if (note === "C" && index !== 0) {
        octave = 5;
      }
      return octave;
    });
  })(rotatedNotes);

  /* Sound functions */
  const playAll = () => {
    const now = Tone.now();
    chords.forEach((_, idx) => playChord(idx, now + idx));
  };

  const playChord = (idx, time) => {
    const notesInChord = chordToNotes(chords[idx]);
    // Map each note in the chord to 'note + octave' like C4, D5
    const notesToPlay = notesInChord.map(
      (note) => note + octaves[rotatedNotes.indexOf(note)]
    );
    polySynth.triggerAttackRelease(notesToPlay, "4n", time);
  };

  const playNote = (idx) =>
    polySynth.triggerAttackRelease(rotatedNotes[idx] + octaves[idx], "4n");

  const increaseOctave = (idx) => octaves[idx] = octaves[idx] + 1;
  const decreaseOctave = (idx) => octaves[idx] = octaves[idx] - 1;

  /* Music theory functions */
  const fromDegrees = (root, degrees) => degrees.map(degree => notes[(notes.indexOf(root) + degree) % 12]);

  const majTriad = (root) => fromDegrees(root, [0, 4, 7]);
  const minTriad = (root) => fromDegrees(root, [0, 3, 7]);
  const dimTriad = (root) => fromDegrees(root, [0, 3, 6]);
  const maj7 = (root) => fromDegrees(root, [0, 4, 7, 11]);
  const min7 = (root) => fromDegrees(root, [0, 3, 7, 10]);
  const dom7 = (root) => fromDegrees(root, [0, 4, 7, 10]);

  const _chordToNotes = (() => {
    const chordToNotes = {};

    for (let root of notes) {
      chordToNotes[root] = majTriad(root);
      chordToNotes[root + "m"] = minTriad(root);
      chordToNotes[root + "dim"] = dimTriad(root);
      chordToNotes[root + "maj7"] = maj7(root);
      chordToNotes[root + "m7"] = min7(root);
      chordToNotes[root + "7"] = dom7(root);
    }
    return chordToNotes;
  })();

  const chordToNotes = (chord) => _chordToNotes[chord];

  /* UI helpers */
  const getHue = (note) => {
    const idx = notes.indexOf(note);
    return Math.sin(idx / 12) * 360;
  };

  /* Button handlers */
  const handleAddChord = () => {
    chords.push(chord);
    chords = chords;
    chord = "";
  };

  const handleClear = () => (chords = []);

  const handleRemoveChord = (idx) => {
    return () => {
      chords.splice(idx, 1);
      chords = chords;
    };
  };

  const rotateLeftOnce = () => rotatedNotes = [...rotatedNotes.slice(1), rotatedNotes[0]];

  const rotateRightOnce = () => rotatedNotes = [ rotatedNotes[rotatedNotes.length - 1], ...rotatedNotes.slice(0, rotatedNotes.length - 1) ];

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

<div class="min-h-screen flex items-center justify-center p-0">
  <div class="bg-gray-10 p-8 rounded-lg shadow-lg max-w-xl w-full">
    <div class="flex items-center justify-center">
      <h2 class="text-1xl font-bold mb-4 text-center items-center justify-center bg-gradient-to-r from-red-400 via-green-500 to-indigo-400 inline-block text-transparent bg-clip-text">
        A I M P R O V I S E
      </h2>
    </div>
    <form on:submit|preventDefault={() => null} class="flex justify-center mb-4 space-x-4">
      <input bind:value={chord} placeholder="Add Chord" id="chord" class="bg-gray-100 hover:bg-gray-200 text-black py-2 px-4 rounded" />
      <button on:click={handleAddChord} id="add-chord" class="bg-gray-300 hover:bg-gray-400 text-black font-bold py-2 px-4 rounded" >Add Chord</button>
      <button on:click={() => playAll()} class="bg-gray-300 hover:bg-gray-400 text-black font-bold py-2 px-4 rounded" >Play</button>
    </form>
    <div class="flex flex-wrap justify-center mb-4 space-x-4">
      {#each chords as chord, idx}
        <div class="relative group">
          <button on:click={() => playChord(idx, Tone.now())} class="border-none my-2 p-2 w-20 font-bold rounded" style="background-color: hsl({getHue(chord[0])}, 100%, 75%)">{chord}</button>
          <button on:click={handleRemoveChord(idx)} class="absolute top-0 right-[-8px] p-2 w-4 h-4 font-bold text-gray-700 rounded-full bg-white flex items-center justify-center font-mono opacity-0 group-hover:opacity-100">x</button>
        </div>
      {/each}
    </div>

    <!-- -->
    <ul class="flex justify-center space-x-2 mb-0">
      {#each octaves as octave, idx}
        <button on:click={() => increaseOctave(idx)} class="border-none font-bold inline list-none rounded-full w-7 h-7 items-center flex justify-center">{"+"}</button>
      {/each}
    </ul>
    <ul class="flex justify-center space-x-2 mb-0">
      <button on:click={() => rotateLeftOnce()} class="border-none font-bold inline list-none rounded-full bg-white-300 w-6 h-6 items-center flex justify-center">{"‹"}</button>
      {#each rotatedNotes as note, idx}
        {#if notesInChords.has(note)}
          <button on:click={() => playNote(idx)} class="border-none click:bg-gray-800 inline list-none rounded-full w-7 h-7 items-center flex justify-center" style="background-color: hsl({getHue(note)}, 100%, 75%)">{note}<sub class="text-[.5rem] align-bottom">{octaves[idx]}</sub></button>
        {:else}
          <button on:click={() => playNote(idx)} class="border-none click:bg-gray-200 inline list-none rounded-full bg-white-300 w-7 h-7 items-center flex justify-center">{note}<sub class="text-xs align-bottom">{octaves[idx]}</sub></button>
        {/if}
      {/each}
      <button on:click={() => rotateRightOnce()} class="border-none font-bold inline list-none rounded-full bg-white-300 w-6 h-6 items-center flex justify-center">{"›"}</button>
    </ul>
    <ul class="flex justify-center space-x-2 mb-4">
      {#each octaves as octave, idx}
        <button on:click={() => decreaseOctave(idx)} class="border-none font-bold inline list-none rounded-full w-7 h-7 items-center flex justify-center">{"-"}</button>
      {/each}
    </ul>
    <!-- -->

    <div class="flex flex-wrap justify-center">
      <button on:click={handleJazzify} id="jazzify" class="m-2 bg-gray-300 hover:bg-gray-400 text-black font-bold py-2 px-4 rounded">Jazzify</button>
      <button on:click={handleDarken} id="jazzify" class="m-2 bg-gray-300 hover:bg-gray-400 text-black font-bold py-2 px-4 rounded">Darken</button>
      <button on:click={handleBrighten} id="jazzify" class="m-2 bg-gray-300 hover:bg-gray-400 text-black font-bold py-2 px-4 rounded">Brighten</button>
      <button on:click={handleEmbellish} id="jazzify" class="m-2 bg-gray-300 hover:bg-gray-400 text-black font-bold py-2 px-4 rounded">Embellish</button>
      <button on:click={handleExtend} id="extend" class="m-2 bg-gray-300 hover:bg-gray-400 text-black font-bold py-2 px-4 rounded">Extend</button>
      <button on:click={handleClear} id="clear" class="m-2 bg-gray-300 hover:bg-gray-400 text-black font-bold py-2 px-4 rounded">Clear</button>
    </div>
  </div>
  <div class="fixed bottom-1">
    <input bind:value={apiKey} type="password" placeholder="OpenAI API KEY" class="bg-gray-100 hover:bg-gray-200 text-black py-2 px-4 rounded text-center"/>
  </div>
</div>
