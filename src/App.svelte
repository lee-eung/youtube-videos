<script>
import CopyClipBoard from './CopyClipBoard.svelte';

let api_key = ''
let playlist_id = ''
let progress_log = ''
let result_json = ''
let copy_button = 'hidden';
let apiKeyInputMsg = '';
let playlistIdInputMsg = '';
let copyResultMsg = '';
let keep_going = true
let apiKeyInput
let playlistIdInput
let interval

const PAGE_TITLE = '유튜브 채널 동영상 전체목록 가져오기'
const URL_PREFIX = "https://www.googleapis.com/youtube/v3/playlistItems?"
const playlist = { videos: [] }

const optionParams = {
	part		: "snippet",
	playlistId	: "",
	key			: "",
	maxResults	: 50
}
const fetchOptions = {
	method: 'GET',
}

const copyResultJson = () => {
	const app = new CopyClipBoard({
		target: document.getElementById('clipboard'),
		props: { result_json },
	})
	copyResultMsg = '클립보드에 복사되었습니다.'
	app.$destroy()
}

function startIntervalFetch() {
	if( !is_input_ok() )
		return
	write_progress_log('유튜브 채널 영상 목록을 가져오기 시작합니다.\n')
	write_progress_log('----------------------------------------\n')
	interval = setInterval(() => {
		sendRequest()
	}, 3000)
}

function is_input_ok() {
	if(api_key.trim().length < 5) {
		apiKeyInput.focus()
		apiKeyInputMsg = 'API Key를 입력해주세요.'
		return false
	}
	apiKeyInputMsg = ''
	if(playlist_id.trim().length < 5) {
		playlistIdInput.focus()
		playlistIdInputMsg = 'YouTube Channel List ID를 입력해주세요.'
		return false
	}
	playlistIdInputMsg = ''
	optionParams['key'] = api_key
	optionParams['playlistId'] = playlist_id
	return true
}

function getURL() {
	let params = []
	for(let option in optionParams) {
		params.push(`${option}=${optionParams[option]}`)
	}
	return URL_PREFIX + params.join('&')
}

function sendRequest() {
	fetch(getURL(), fetchOptions)
		.then(response => response.text())
		.then(result => get_video_list(result))
		.catch(error => console.log('error', error))
}

function get_video_list(body) {
	const result = JSON.parse(body)
	const items = result.items
	for (let index in items) {
		const video_info = get_video_info(items[index])
		playlist['videos'].length > 1 && write_progress_log(', ')
		write_progress_log(video_info.position + 1)
	}
	check_keep_going(result)
}

function check_keep_going(result) {
	optionParams['pageToken'] = result.nextPageToken
	keep_going = (playlist['videos'].length < result.pageInfo.totalResults)
	console.log({
		'number of videos': playlist['videos'].length,
		'keep_going': keep_going,
		'result.nextPageToken': result.nextPageToken,
	})
	if (!keep_going) {
		clearInterval(interval)
		copy_button = '';
		write_progress_log('\n----------------------------------------\n')
		write_progress_log('유튜브 채널 영상 목록 가져오기를 마쳤습니다.\n')
		result_json = JSON.stringify(playlist)
	}
}

function get_video_info(item) {
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

function write_progress_log(message) {
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
			YouTube Data API v3 - API Key: <input type="password" bind:value={api_key} bind:this={apiKeyInput}> <span class="input_msg">{apiKeyInputMsg}</span>
		</div>
		<div>
			YouTube Channel List ID: <input type="text" bind:value={playlist_id} bind:this={playlistIdInput}> <span class="input_msg">{playlistIdInputMsg}</span>
		</div>
		<button on:click={startIntervalFetch}>영상 리스트 가져오기</button>
		<button on:click={copyResultJson} class={copy_button}>json 내용을 클립보드에 복사하기</button> <span class="copy_result_msg">{copyResultMsg}</span>
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
