<script>
    import * as Tone from "tone";
    import { onMount } from "svelte";
    import { slide } from "svelte/transition";

    let player;
    let isPlaying = false;
    let audiofile;
    let audiourl;
    let playbackRate = 1.0;
    let pitchShift; 

    let pitchRate = 0;

    async function initializeTone() {
        await Tone.start();
        
        pitchShift = new Tone.PitchShift().toDestination();
        player = new Tone.Player(audiourl).connect(pitchShift);

        player.loop = true;

        const toneFFT = new Tone.FFT();
		pitchShift.connect(toneFFT);
        

        console.log(player);
        player.autostart = false;
    }

    function togglePlayback() {
        if (isPlaying) {
            player.stop();
        } else {
            player.start();
        }
        isPlaying = !isPlaying;
    }

    onMount(async () => {
        console.log("test");
    });

    function handleFileChange(e) {
        const selectedFile = e.target.files[0];

        if (selectedFile) {
            // Store the selected file in the audiofile variable
            audiofile = selectedFile;

            // You can use the 'audiofile' variable to do further processing
            console.log("Selected audio file:", audiofile);

            audiourl = URL.createObjectURL(audiofile);
            initializeTone();
        }
    }

    function faster(e) {
        player.playbackRate = playbackRate;
    }

    function pitcher(e){
        pitchShift.pitch = pitchRate;
        console.log(pitchShift.pitch);
    }

    function download(e) {
        const audioData = player.buffer;
        const blob = new Blob([audioData], { type: "audio/wav" });
        const url = URL.createObjectURL(blob);
        console.log(url);

        
    }
</script>

<h3>Audio Player</h3>

<!-- Play/Pause button -->
<input type="file" on:change={handleFileChange} />

<br>
<br>

<input
    bind:value={playbackRate}
    step="0.1"
    min="1.0"
    max="2.0"
    type="range"
    on:click={faster}
    oninput="this.nextElementSibling.value = this.value"
/>
<output>1.0</output>

<br>
<br>

<input
    bind:value={pitchRate}
    step="0.1"
    min="0.0"
    max="2.0"
    type="range"
    on:click={pitcher}
    oninput="this.nextElementSibling.value = this.value"
/>
<output>1.0</output>


{#if player}
    <button on:click={togglePlayback}>
        {#if isPlaying}
            Pause
        {:else}
            Play
        {/if}
    </button>
{/if}

<br>
<br>

<button on:click={download}>Download</button>
