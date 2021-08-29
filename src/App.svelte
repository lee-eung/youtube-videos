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

const URL_PREFIX = "https://www.googleapis.com/youtube/v3/playlistItems?"
const videos = { videos: [] }

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
	write_progress_log('유튜브 채널 영상 목록을 가져오기 시작합니다...\n')
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
		// youtube downloader에 videoId 넘기면 됨.
		const video_info = get_video_info(items, index)
		write_progress_log(`${video_info.position}, `)
	}
	check_keep_going(result)
}

function check_keep_going(result) {
	optionParams['pageToken'] = result.nextPageToken
	keep_going = (videos['videos'].length < result.pageInfo.totalResults)
	console.log({
		'videos.length': videos['videos'].length,
		'keep_going': keep_going,
		'result.nextPageToken': result.nextPageToken,
	})
	if (!keep_going) {
		clearInterval(interval)
		copy_button = '';
		write_progress_log('\n유튜브 채널 영상 목록을 가져오기를 마쳤습니다.\n')
		console.log(videos)
		result_json = JSON.stringify(videos)
	}
}

function get_video_info(items, index) {
	const video_info = {
		position: items[index].snippet.position,
		publishedAt: items[index].snippet.publishedAt,
		videoId: items[index].snippet.resourceId.videoId,
		videoURL: `https://www.youtube.com/watch?v=${items[index].snippet.resourceId.videoId}`,
		thumbnail: items[index].snippet.thumbnails.high.url,
		title: items[index].snippet.title,
		description: items[index].snippet.description,
	}
	videos['videos'].push(video_info)
	return video_info
}

function write_progress_log(message) {
	progress_log += message
}
</script>

<svelte:head>
	<title>유튜브 채널 동영상 전체목록 가져오기</title>
</svelte:head>

<div class="form">
    <div>
        YouTube Data API v3 - API Key: <input type="password" bind:value={api_key} bind:this={apiKeyInput}> <span class="input_msg">{apiKeyInputMsg}</span>
    </div>
    <div>
        YouTube Channel List ID: <input type="text" bind:value={playlist_id} bind:this={playlistIdInput}> <span class="input_msg">{playlistIdInputMsg}</span>
    </div>
    <button on:click={startIntervalFetch}>영상 리스트 가져오기 시작</button>
	<button on:click={copyResultJson} class={copy_button}>json 내용 클립보드에 복사하기</button> {copyResultMsg}
	<div id="clipboard"></div>
</div>
<div>
    <textarea>{progress_log}</textarea>
    <textarea>{result_json}</textarea>
</div>

<style>
button {
    cursor: pointer;
}
textarea {
	width: 100%;
	height: 300px;
	display: block;
}
.hidden {
	display: none;
}
.input_msg {
	color: red;
}
</style>