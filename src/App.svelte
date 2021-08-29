<script>
import CopyClipBoard from './CopyClipBoard.svelte';

let api_key = ''
let playlist_id = ''
let progress_log = ''
let result_json = ''
let copy_button = 'hidden';
let api_key_input_msg = '';
let playlist_id_input_msg = '';
let copy_result_msg = '';
let keep_going = true
let api_key_input
let playlist_id_input
let interval

const PAGE_TITLE = '유튜브 채널 동영상 전체목록 가져오기'
const URL_PREFIX = "https://www.googleapis.com/youtube/v3/playlistItems?"
const playlist = { videos: [] }

const option_params = {
	part		: "snippet",
	playlistId	: "",
	key			: "",
	maxResults	: 50
}
const FETCH_OPTIONS = {
	method: 'GET',
}

const copyResultJson = () => {
	const app = new CopyClipBoard({
		target: document.getElementById('clipboard'),
		props: { result_json },
	})
	copy_result_msg = '클립보드에 복사되었습니다.'
	app.$destroy()
}

function startIntervalFetch() {
	if( !isInputOk() )
		return
	writeProgressLog('유튜브 채널 영상 목록을 가져오기 시작합니다.\n')
	writeProgressLog('----------------------------------------\n')
	interval = setInterval(() => {
		sendRequest()
	}, 3000)
}

function isInputOk() {
	if(api_key.trim().length < 5) {
		api_key_input.focus()
		api_key_input_msg = 'API Key를 입력해주세요.'
		return false
	}
	api_key_input_msg = ''
	if(playlist_id.trim().length < 5) {
		playlist_id_input.focus()
		playlist_id_input_msg = 'YouTube Channel List ID를 입력해주세요.'
		return false
	}
	playlist_id_input_msg = ''
	option_params['key'] = api_key
	option_params['playlistId'] = playlist_id
	return true
}

function getURL() {
	let params = []
	for(let option in option_params) {
		params.push(`${option}=${option_params[option]}`)
	}
	return URL_PREFIX + params.join('&')
}

function sendRequest() {
	fetch(getURL(), FETCH_OPTIONS)
		.then(response => response.text())
		.then(body => getVideoList(body))
		.catch(error => console.log('error', error))
}

function getVideoList(body) {
	const result = JSON.parse(body)
	result.items.forEach(item => {
		const video_info = getVideoInfo(item)
		playlist['videos'].length > 1 && writeProgressLog(', ')
		writeProgressLog(video_info.position + 1)
	})
	checkKeepGoing(result)
}

function checkKeepGoing(result) {
	option_params['pageToken'] = result.nextPageToken
	keep_going = (playlist['videos'].length < result.pageInfo.totalResults)
	// console.log({
	// 	'number of videos': playlist['videos'].length,
	// 	'keep_going': keep_going,
	// 	'result.nextPageToken': result.nextPageToken,
	// })
	if (!keep_going) {
		clearInterval(interval)
		copy_button = '';
		writeProgressLog('\n----------------------------------------\n')
		writeProgressLog('유튜브 채널 영상 목록 가져오기를 마쳤습니다.\n')
		result_json = JSON.stringify(playlist)
	}
}

function getVideoInfo(item) {
	const video_info = {
		position: item.snippet.position,
		publishedAt: item.snippet.publishedAt,
		videoId: item.snippet.resourceId.videoId,
		videoURL: `https://www.youtube.com/watch?v=${item.snippet.resourceId.videoId}`,
		thumbnail: item.snippet.thumbnails.high.url,
		title: item.snippet.title,
		description: item.snippet.description,
	}
	playlist['videos'].push(video_info)
	return video_info
}

function writeProgressLog(message) {
	progress_log += message
}
</script>

<svelte:head>
	<title>{PAGE_TITLE}</title>
</svelte:head>

<main>
	<header>{PAGE_TITLE}</header>
	<div class="form">
		<div>
			YouTube Data API v3 - API Key: <input type="password" bind:value={api_key} bind:this={api_key_input}> <span class="input_msg">{api_key_input_msg}</span>
		</div>
		<div>
			YouTube Channel List ID: <input type="text" bind:value={playlist_id} bind:this={playlist_id_input}> <span class="input_msg">{playlist_id_input_msg}</span>
		</div>
		<button on:click={startIntervalFetch}>영상 목록 가져오기</button>
		<button on:click={copyResultJson} class={copy_button}>json 내용을 클립보드에 복사하기</button> <span class="copy_result_msg">{copy_result_msg}</span>
		<div id="clipboard"></div>
	</div>
	<div>
		<textarea readonly>{progress_log}</textarea>
		<textarea readonly>{result_json}</textarea>
	</div>
</main>

<style>
main {
	padding: 1em;
}
header {
	width: 100%;
	font-size: 1.5em;
	font-weight: bolder;
	margin-bottom: 0.6em;
}
button {
    cursor: pointer;
}
input {
	width: 300px;
}
textarea {
	width: 100%;
	height: 250px;
	display: block;
}
.hidden {
	display: none;
}
.input_msg {
	color: red;
}
.copy_result_msg {
	color: blue;
}
</style>
