<script lang="ts">
	import { GoogleGenerativeAI } from '@google/generative-ai';
	import axios from 'axios';
	import { Loader2 } from 'lucide-svelte';
	// @ts-ignore
	import { marked } from 'marked';
	import litspeare from '../assets/litspeare.png';

	let songName: string = '';
	let loading: boolean = false;
	let songList: any[] = [];
	let lyrics: string = '';
	const apiKey = 'AIzaSyASvpYQOxdfuoLw0-P_3szMkP5IFV8bIAk';
	const genAi = new GoogleGenerativeAI(apiKey);
	const model = genAi.getGenerativeModel({ model: 'gemini-1.5-flash-latest' });
	const options = {
		method: 'GET',
		url: 'https://deezerdevs-deezer.p.rapidapi.com/search',
		params: { q: 'eminem' },
		headers: {
			'x-rapidapi-key': '3dc01137bfmsh363953871259f0cp16f822jsn12f849342afa',
			'x-rapidapi-host': 'deezerdevs-deezer.p.rapidapi.com'
		}
	};

	async function searchSong(name: string) {
		try {
			loading = true;
			const response = await axios.get('https://deezerdevs-deezer.p.rapidapi.com/search', {
				params: { q: name },
				headers: {
					'x-rapidapi-key': '3dc01137bfmsh363953871259f0cp16f822jsn12f849342afa',
					'x-rapidapi-host': 'deezerdevs-deezer.p.rapidapi.com'
				}
			});
			console.log(response.data.data);
			songList = response.data.data.slice(0, 5);
		} catch (error) {
			console.error(error);
		} finally {
			loading = false;
		}
	}

	async function generateResponse(song: string, artist: string) {
		try {
			loading = true;
			songName = '';
			songList = [];
			let prompt = `Generate lyrics in Shakespearean English for the song ${song} by ${artist} and provide the lyrics in markdown format`;
			const response = await model.generateContent(prompt);
			console.log(response.response.text());
			lyrics = response.response.text();
		} catch (e) {
			console.error(e);
		} finally {
			loading = false;
		}
	}
</script>

<div class="w-full mt-10 flex flex-col justify-center items-center text-[#200000]">
	<h1 class="text-4xl font-medium">Generate Lyrics</h1>
	<p class="mb-7 mt-1">for a song in Shakespearean</p>
	<div class="md:w-1/3 w-full flex flex-row items-end gap-3">
		<input
			placeholder="e.g Never gonna give you up"
			class="px-3 py-2 rounded w-full bg-[#20000015]"
			bind:value={songName}
			on:input={() => {
				if (lyrics.length > 0) {
					lyrics = '';
				}
				if (songName == '') {
					songList = [];
				}
				searchSong(songName);
			}}
		/>
		<!-- <button
        class="bg-[#1a1a1a] px-3 py-2 text-lg font-medium text-white rounded"
        on:click={() => searchSong(songName)}>Search</button
		> -->
	</div>
	<div class="flex flex-col md:w-1/3 w-full py-3 mt-2 px-3 bg-[#20000015] rounded">
		{#if loading}
			<Loader2 size="24" class="self-center text-center animate-spin" />
		{/if}
		<p class="">{@html marked.parse(lyrics)}</p>
		{#each songList as song}
			<div
				class="flex flex-row hover:cursor-default hover:bg-[#20000020] rounded justify-between px-3 py-1 last:border-b-0 border-b border-b-[#20000025]"
				on:click={() => generateResponse(song.title, song.artist.name)}
			>
				<p>{song.title}</p>
				<p>{song.artist.name}</p>
			</div>
		{/each}
	</div>
	<img src={litspeare} class="md:w-1/2 w-full fixed md:top-[42vh] top-[72vh] right-[20px] opacity-40" alt="litspeare " />
</div>
