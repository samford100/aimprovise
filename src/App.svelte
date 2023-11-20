<script>
  let chords = ["G", "Em", "C", "D"];
  let chord = "";
  let explanation = "";

  function play() {
	const polySynth = new Tone.PolySynth(Tone.Synth).toDestination();

	let now = Tone.now()
	for (let [idx, chord] of chords.entries()) {
		let notes = chordToNotes(chord).map(note => note + "4");
		polySynth.triggerAttackRelease(notes, "4n", now + 1*idx);
	}
  }

  function chordToNotes(chord) {
    return _chordToNotes[chord].map((note) => new Note(note));
  }

  let notes = [ "A", "A#", "B", "C", "C#", "D", "D#", "E", "F", "F#", "G", "G#"]

  function majTriad(root) { return [root, notes[(notes.indexOf(root) + 4) % 12], notes[(notes.indexOf(root) + 7) % 12]] };
  function minTriad(root) { return [root, notes[(notes.indexOf(root) + 3) % 12], notes[(notes.indexOf(root) + 7) % 12]] };
  function dimTriad(root) { return [root, notes[(notes.indexOf(root) + 3) % 12], notes[(notes.indexOf(root) + 6) % 12]] };
  function maj7(root) { return [root, notes[(notes.indexOf(root) + 4) % 12], notes[(notes.indexOf(root) + 7) % 12], notes[(notes.indexOf(root) + 11) % 12]] };
  function min7(root) { return [root, notes[(notes.indexOf(root) + 3) % 12], notes[(notes.indexOf(root) + 7) % 12], notes[(notes.indexOf(root) + 10) % 12]] };
  function dom7(root) { return [root, notes[(notes.indexOf(root) + 4) % 12], notes[(notes.indexOf(root) + 7) % 12], notes[(notes.indexOf(root) + 10) % 12]] };

  // create mapping from chord name (note + type) to notes
  function createChordToNotes() {
	let chordToNotes = {};

	for (let root of notes) {
		chordToNotes[root] = majTriad(root);
		chordToNotes[root+'m'] = minTriad(root);
		chordToNotes[root+'dim'] = dimTriad(root);
		chordToNotes[root+'maj7'] = maj7(root);
		chordToNotes[root+'m7'] = min7(root);
		chordToNotes[root+'7'] = dom7(root);
	}
	return chordToNotes;
  }

  function handleAddChord() {
    chords.push(chord);
    chords = chords;
    chord = "";
  }

  let _chordToNotes = createChordToNotes();

  function getHue(chord) {
	let idx = notes.indexOf(chord[0]);
	return Math.sin(idx/12) * 360;;
  }

  async function sendRequest(prompt) {
    let apiKey = "Insert here";

    let req = {
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
  }

  function handleClear() {
    chords = [];
  }

  function handleRemoveChord(idx) {
	return () => {
	  chords.splice(idx, 1);
	  chords = chords;
	};
  }

  async function handleJazzify() {
    let prompt = `Jazzify the following chord progression: ${chords.join("")}. 
	Return the result as a comma separated list of chords only. Do not respond with anything except the chords.`;

	chords = (await sendRequest(prompt)).choices[0].message.content.split(",").map(chord => chord.trim()).map(chord => chord.replace("min", "m"));
	console.log(chords)
  }

  async function handleExtend() {
    let prompt = `Extend the following chord progression: ${chords.join( "")} by adding ${chords.length} more chords.
	Return the original chords and the new chords as a comma separated list of chords only. Do not respond with anything except the chords.`;

	chords = (await sendRequest(prompt)).choices[0].message.content.split(",").map(chord => chord.trim()).map(chord => chord.replace("min", "m"));
  }

  async function handleExplain() {
	let prompt = `Explain the following chord progression: ${chords.join("")}. 
	Be concise, but provide enough information to understand the progression.`;
	let data = await sendRequest(prompt);

	explanation = data.choices[0].message.content;
  } 

  async function handleDarken() {
	let prompt = `Darken the following chord progression: ${chords.join("")}.
	Return the result as a comma separated list of chords only. Do not respond with anything except the chords.`;

	chords = (await sendRequest(prompt)).choices[0].message.content.split(",").map(chord => chord.trim()).map(chord => chord.replace("min", "m"));
  }

  async function handleBrighten() {
	let prompt = `Brighten the following chord progression: ${chords.join("")}.
	Return the result as a comma separated list of chords only. Do not respond with anything except the chords.`;

	chords = (await sendRequest(prompt)).choices[0].message.content.split(",").map(chord => chord.trim()).map(chord => chord.replace("min", "m"));
  }

  async function handleEmbellish() {
	let prompt = `Add embellishments to the following chord progression: ${chords.join("")}.
	Return the result as a comma separated list of chords only. Do not respond with anything except the chords.`;

	chords = (await sendRequest(prompt)).choices[0].message.content.split(",").map(chord => chord.trim()).map(chord => chord.replace("min", "m"));
  }



</script>

<div class="min-h-screen flex items-center justify-center">
  <div class="bg-gray-10 p-8 rounded-lg shadow-lg max-w-xl w-full">
    <form on:submit|preventDefault={() => null} class="flex justify-center mb-4 space-x-4">
      <input bind:value={chord} placeholder="Add Chord" id="add-chord" class="bg-gray-100 hover:bg-gray-200 text-black py-2 px-4 rounded" />
      <button on:click={handleAddChord} id="add-chord" class="bg-gray-300 hover:bg-gray-400 text-black font-bold py-2 px-4 rounded" >Add Chord</button>
      <button on:click={play} class="bg-gray-300 hover:bg-gray-400 text-black font-bold py-2 px-4 rounded" >Play</button>
	</form>
    <div class="flex flex-wrap justify-center mb-4 space-x-4">
      {#each chords as chord, idx}
        <button on:click={handleRemoveChord(idx)}  class="my-2 p-2 w-20 font-bold rounded" style="background-color: hsl({getHue(chord)}, 100%, 75%)">{chord}</button>
      {/each}
    </div>
    <div class="flex flex-wrap justify-center">
      <button on:click={handleJazzify} id="jazzify" class="m-2 bg-gray-300 hover:bg-gray-400 text-black font-bold py-2 px-4 rounded" >Jazzify</button>
      <button on:click={handleDarken} id="jazzify" class="m-2 bg-gray-300 hover:bg-gray-400 text-black font-bold py-2 px-4 rounded" >Darken</button>
      <button on:click={handleBrighten} id="jazzify" class="m-2 bg-gray-300 hover:bg-gray-400 text-black font-bold py-2 px-4 rounded" >Brighten</button>
      <button on:click={handleEmbellish} id="jazzify" class="m-2 bg-gray-300 hover:bg-gray-400 text-black font-bold py-2 px-4 rounded" >Embellish</button>
      <button on:click={handleExtend} id="extend" class="m-2 bg-gray-300 hover:bg-gray-400 text-black font-bold py-2 px-4 rounded" >Extend</button>
      <button on:click={handleClear} id="clear" class="m-2 bg-gray-300 hover:bg-gray-400 text-black font-bold py-2 px-4 rounded" >Clear</button>
      <!-- <button on:click={handleExplain} id="clear" class="m-2 bg-gray-300 hover:bg-gray-400 text-black font-bold py-2 px-4 rounded" >Explain</button> -->
    </div>
	<!-- <div class="flex justify-center">
		<textarea bind:value={explanation} class="m-2 p-2 font-bold bg-gray-200 rounded" />
	</div> -->
  </div>
</div>
