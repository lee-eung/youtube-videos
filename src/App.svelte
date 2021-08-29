<script>
import CopyClipBoard from './CopyClipBoard.svelte'

let api_key = ''
let playlist_id = ''
let progress_log = ''
let result_json = ''
let how_many_videos = 50
let need_videos = -1
let num_of_videos_class = 'hidden'
let copy_button_class = 'hidden'
let api_key_input_msg = ''
let playlist_id_input_msg = ''
let copy_result_msg = ''
let get_all_text_class = ''
let keep_going = true
let api_key_input
let playlist_id_input
let checkbox
let interval

const IS_DEBUG_MODE = !true
const PAGE_TITLE = '유튜브 채널 동영상 목록 가져오기'
const URL_PREFIX = "https://www.googleapis.com/youtube/v3/playlistItems?"
const playlist = { videos: [] }
const DEFAULT_MAX_RESULTS = 50

const option_params = {
    part        : "snippet",
    playlistId    : "",
    key            : "",
    maxResults    : DEFAULT_MAX_RESULTS
}
const FETCH_OPTIONS = {
    method: 'GET',
}

function startIntervalFetch() {
    if( !isInputOk() )
        return
    writeProgressLog('유튜브 채널 영상 목록을 가져오기 시작합니다.\n')
    writeProgressLog('(아래 출력되는 번호들은 영상의 index 번호이며 0번부터 시작합니다.)\n')
    writeProgressLog('----------------------------------------\n')
    interval = setInterval(() => {
        sendRequest()
    }, 3000)
}

function isInputOk() {
    initializeAll()
    if(api_key.trim().length < 5) {
        api_key_input.focus()
        api_key_input_msg = 'API Key를 입력해주세요.'
        return false
    }
    if(playlist_id.trim().length < 5) {
        playlist_id_input.focus()
        playlist_id_input_msg = 'YouTube Channel List ID를 입력해주세요.'
        return false
    }
    return true
}

function initializeAll() {
    api_key_input_msg = ''
    playlist_id_input_msg = ''
    copy_result_msg = ''
    copy_button_class = 'hidden'
    progress_log = ''
    result_json = ''
    need_videos = checkbox.checked ? -1 : how_many_videos
    option_params['pageToken'] = ''
    option_params['key'] = api_key
    option_params['playlistId'] = playlist_id
    playlist['videos'] = []
}

function getMaxResults() {
    if(checkbox.checked) {
        return DEFAULT_MAX_RESULTS
    }
    const current_videos = playlist['videos'].length
    need_videos = how_many_videos - current_videos
    const max_results = need_videos < DEFAULT_MAX_RESULTS ? need_videos : DEFAULT_MAX_RESULTS
    IS_DEBUG_MODE && console.log({
        'current_videos': current_videos,
        'need_videos': need_videos,
        'max_results': max_results,
    })
    return max_results
}

function getURL() {
    option_params['maxResults'] = getMaxResults()
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
    try {
        result.items.forEach(item => {
            const video_info = getVideoInfo(item)
            playlist['videos'].length > 1 && writeProgressLog(', ')
            writeProgressLog(video_info.index)
        })
        checkKeepGoing(result)
    } catch (e) {
        console.log({
            'result.items': result.items
        })
        console.log(e)
    }
}

function checkKeepGoing(result) {
    option_params['pageToken'] = result.nextPageToken
    if(checkbox.checked) {
        keep_going = (playlist['videos'].length < result.pageInfo.totalResults)
    } else {
        keep_going = (need_videos > DEFAULT_MAX_RESULTS)
    }
    IS_DEBUG_MODE && console.log({
        'number of videos': playlist['videos'].length,
        'keep_going': keep_going,
        'nextPageToken': result.nextPageToken,
    })
    if (!keep_going) {
        clearInterval(interval)
        copy_button_class = ''
        writeProgressLog('\n----------------------------------------\n')
        writeProgressLog('유튜브 채널 영상 목록 가져오기를 마쳤습니다.\n')
        result_json = JSON.stringify(playlist, null, 4)
    }
}

function getVideoInfo(item) {
    const video_info = {
        index: item.snippet.position,
        publishedAt: item.snippet.publishedAt.replace('T', ' ').replace('Z', ''),
        videoId: item.snippet.resourceId.videoId,
        videoURL: `https://www.youtube.com/watch?v=${item.snippet.resourceId.videoId}`,
        thumbnail: item.snippet.thumbnails.high.url,
        title: item.snippet.title,
        description: item.snippet.description,
    }
    playlist['videos'].push(video_info)
    return video_info
}

const copyResultJson = () => {
    const app = new CopyClipBoard({
        target: document.getElementById('clipboard'),
        props: { result_json },
    })
    copy_result_msg = '클립보드에 복사되었습니다.'
    app.$destroy()
}

function toggleHowManyVideos() {
    if(checkbox.checked) {
        get_all_text_class = ''
        num_of_videos_class = 'hidden'
    } else {
        get_all_text_class = 'vague_text'
        num_of_videos_class = ''
    }
}

function writeProgressLog(message) {
    progress_log += message
}

const onKeyPress = e => {
    if(e.charCode === 13) {
        startIntervalFetch()
    }
}
</script>

<svelte:head>
    <title>{PAGE_TITLE}</title>
</svelte:head>

<main>
    <header>{PAGE_TITLE}</header>
    <div class="form">
        <div>
            YouTube Data API v3 - API Key: <input
                type="password" bind:value={api_key}
                bind:this={api_key_input}
                on:keypress={onKeyPress}> <span 
                class="input_msg">{api_key_input_msg}</span>
        </div>
        <div>
            YouTube Channel List ID: <input
                type="text" bind:value={playlist_id}
                bind:this={playlist_id_input}
                on:keypress={onKeyPress}> <span
                class="input_msg">{playlist_id_input_msg}</span>
        </div>
        <input type="checkbox" bind:this={checkbox}
            checked on:click={toggleHowManyVideos} class="checkbox_class"> <span
            class={get_all_text_class}>전체</span>
        <span class={num_of_videos_class}>최근 <input
            type="number" bind:value={how_many_videos}
            on:keypress={onKeyPress}
            min="1" class="input_number">개</span>
        <button on:click={startIntervalFetch}>영상 목록 가져오기</button>
        <button on:click={copyResultJson}
            class="button-primary {copy_button_class}">json 내용을 클립보드에 복사하기</button> <span
            class="copy_result_msg">{copy_result_msg}</span>
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
.checkbox_class {
    width: 20px;
}
.vague_text {
    color: silver;
    text-decoration: line-through;
}
.input_number {
    width: 70px;
    text-align: end;
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
.button-primary {
    background-color: blue;
    color: white;
}
</style>
