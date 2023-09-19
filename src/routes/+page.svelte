<script>
    import * as Tone from "tone";
    import { onMount } from "svelte";

    let player;
    let isPlaying = false;
    let audioFile;
    let audioUrl;
    let playbackRate = 1.0;
    let pitchShift; 
    let showPlayButton = false;
    let pitchRate = 0;
    let volume = 0;
    let playerInitialized = false;



    function handleFileChange(e) {
        const selectedFile = e.target.files[0];

        if (selectedFile) {
            // Store the selected file in the audiofile variable
            audioFile = selectedFile;

            // You can use the 'audiofile' variable to do further processing
            console.log("Selected audio file:", audioFile);

            audioUrl = URL.createObjectURL(audioFile);

            if (!playerInitialized){
                initializeTone();
            }else{
                player.stop();
                console.log('stop');
                player.load(audioUrl);

                if(isPlaying){
                    isPlaying=false;
                }
                
            }
            
            setTimeout(function(){
                showPlayButton = true;
            }, 100);

        }
    }



    
    async function initializeTone() {
        await Tone.start();
        
        pitchShift = new Tone.PitchShift().toDestination();
        player = new Tone.Player(audioUrl).connect(pitchShift);

        player.loop = true;

        const toneFFT = new Tone.FFT();
		pitchShift.connect(toneFFT);
        
        player.autostart = false;

        playerInitialized = true;
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

    function changeVolume() {
        player.volume.value = volume;
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



<br>
<br>
<h3>Lautstärke</h3>
<input
    bind:value={volume}
    step="1"
    min="-5"
    max="5"
    type="range"
    on:click={changeVolume}
    oninput="this.nextElementSibling.value = this.value"
/>
<output>1.0</output>


{#if showPlayButton}
    <button on:click={togglePlayback} >
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
