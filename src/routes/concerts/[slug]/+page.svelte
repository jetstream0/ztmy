<script lang="ts">
  //imports
  import { onMount } from 'svelte';
  import SetItem from '$lib/SetItem.svelte';
  import LangToggle from '$lib/LangToggle.svelte';
  import Video from '$lib/Video.svelte';
  import Lyrics from '$lib/Lyrics.svelte';
  import OffsetInput from '$lib/OffsetInput.svelte';
  import concerts_metadata from '$lib/data/concerts/concerts_metadata.json';
  import type { ConcertInfo } from '$lib/types.ts';
  
  //exports
  export let data;

  //get concert info
  const slug: string = data.slug;
  const concert_info: ConcertInfo = data.concert_info;
  let video_src: string | boolean = data.video_src;
  const lyrics_text: string | boolean = data.lyrics_text;

  //variables
  let lang: string = "eng";
  let time_jump: number = -1;
  let current_time: number = 0;
  let offset: number = 0;

  //toggle info
  let show_info = false;

  function toggle_info() {
    if (show_info) {
      show_info = false;
    } else {
      show_info = true;
    }
  }

  function toggle_info_keypress(event: KeyboardEvent) {
    if (event.key === "Enter") {
      toggle_info();
    }
  }

  onMount(() => {
    //language based on preference
    if (navigator.language.toLowerCase().startsWith("ja")) {
      lang = "jap";
    } else if (navigator.language.toLowerCase().startsWith("en")) {
      lang = "eng";
    }
    //url params
    let search_params = new URLSearchParams(window.location.search);
    let default_video_src: string | null = search_params.get("default_video_src");
    if (default_video_src) {
      video_src = default_video_src;
    }
  });
</script>

<svelte:head>
  <title>{ concert_info.title }</title>
</svelte:head>

<div>
  <div id="lang-toggle-container">
    <LangToggle bind:lang={lang}/>
  </div>
  <div id="concert-container">
    <div>
      <h2>
        Info
        <span id="clickable-info" role="button" on:click={toggle_info} on:keypress={toggle_info_keypress} tabindex="0">
          {#if show_info}
            -
          {:else}
            +
          {/if}
        </span>
      </h2>
      {#if show_info}
        <div>
          {concert_info.date}
        </div>
      {/if}
      <h2>Setlist</h2>
      <div id="setlist" class="{ show_info ? 'smaller-setlist' : '' }">
        <ol>
          {#each concert_info.setlist as set_item}
            <li><SetItem {...set_item} {lang} {current_time} {offset} bind:time_jump={time_jump}/></li>
          {/each}
        </ol>
      </div>
    </div>
    <div>
      <h2>Video</h2>
      <div>
        <Video {video_src} sub_src={concert_info.sub_src} ytdlp={concert_info.ytdlp} {time_jump} bind:current_time={current_time}/>
        <OffsetInput bind:offset={offset}/>     
      </div>
    </div>
    <div>
      {#if lang == "jap"}
        <h2>歌詞</h2>
      {:else}
        <h2>Lyrics</h2>
      {/if}
      <div>
        <Lyrics lyrics={lyrics_text} {current_time} {lang} {offset}/>
      </div>
    </div>
  </div>
</div>

<style>
  #lang-toggle-container {
    float: right;
  }

  #setlist {
    max-height: 80vh;
    overflow-y: scroll;
    scrollbar-color: #A6A6A6 #efefef;
    scrollbar-width: thin;
    padding-right: 7px;
  }

  /* manage scrollbar for chromium/safari too */
  #setlist::-webkit-scrollbar {
    width: 5px;
    background-color: transparent;
  }

  #setlist::-webkit-scrollbar-thumb {
    background: #868686;
  }

  .smaller-setlist {
    max-height: 72vh !important;
  }

  #concert-container {
    display: grid;
    grid-template-columns: 19% 55% 26%;
    column-gap: 24px;
  }

  #clickable-info {
    cursor: pointer;
    padding: 3px;
  }

  @media screen and (max-width: 1000px) {
    #concert-container {
      grid-template-columns: auto;
    }

    #setlist {
      max-height: 45vh;
    }
  }
</style>
