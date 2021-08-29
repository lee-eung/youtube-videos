# 유튜브 채널 동영상 목록 가져오기 (a svelte app with YouTube Data API v3 to fetch videos on a channel)

YouTube Data API v3을 이용해서 유튜브 채널의 동영상 목록을 가져오는 svelte app입니다.

<a href="https://console.cloud.google.com/apis/credentials" target="_blank">Google Cloud Platform의 "API 및 서비스" > "사용자 인증 정보"</a> 페이지에서 "YouTube Data API v3"의 API Key를 발급받은 후 아래 링크 페이지에서 테스트해볼 수 있습니다.

<a href="https://lee-eung.github.io/svelte/youtube-videos/" target="_blank">https://lee-eung.github.io/svelte/youtube-videos/</a>


위 페이지의 입력 항목 중 "YouTube Channel List ID"는 재생목록ID를 의미하므로, 이미 만들어둔 재생목록의 영상 리스트를 추출할 수도 있구요. 특정 채널의 동영상 탭에서 "업로드한 동영상 > 모두 재생"을 클릭했을 때 만들어지는 재생목록ID를 가지고 해당 채널의 영상 목록 전체를 가져올 수 있습니다.

영상 목록 추출 결과물은 나름대로 딱 필요하다고 생각되는 항목들로 구성되어 있으며, json 형식이라서 개발언어에 상관없이 원하는대로 가공하여 쓸 수 있을 겁니다.

---

# svelte app

This is a project template for [Svelte](https://svelte.dev) apps. It lives at https://github.com/sveltejs/template.

To create a new project based on this template using [degit](https://github.com/Rich-Harris/degit):

```bash
npx degit sveltejs/template svelte-app
cd svelte-app
```

*Note that you will need to have [Node.js](https://nodejs.org) installed.*


## Get started

Install the dependencies...

```bash
cd svelte-app
npm install
```

...then start [Rollup](https://rollupjs.org):

```bash
npm run dev
```

Navigate to [localhost:5000](http://localhost:5000). You should see your app running. Edit a component file in `src`, save it, and reload the page to see your changes.

By default, the server will only respond to requests from localhost. To allow connections from other computers, edit the `sirv` commands in package.json to include the option `--host 0.0.0.0`.

If you're using [Visual Studio Code](https://code.visualstudio.com/) we recommend installing the official extension [Svelte for VS Code](https://marketplace.visualstudio.com/items?itemName=svelte.svelte-vscode). If you are using other editors you may need to install a plugin in order to get syntax highlighting and intellisense.

## Building and running in production mode

To create an optimised version of the app:

```bash
npm run build
```

You can run the newly built app with `npm run start`. This uses [sirv](https://github.com/lukeed/sirv), which is included in your package.json's `dependencies` so that the app will work when you deploy to platforms like [Heroku](https://heroku.com).


## Single-page app mode

By default, sirv will only respond to requests that match files in `public`. This is to maximise compatibility with static fileservers, allowing you to deploy your app anywhere.

If you're building a single-page app (SPA) with multiple routes, sirv needs to be able to respond to requests for *any* path. You can make it so by editing the `"start"` command in package.json:

```js
"start": "sirv public --single"
```

## Using TypeScript

This template comes with a script to set up a TypeScript development environment, you can run it immediately after cloning the template with:

```bash
node scripts/setupTypeScript.js
```

Or remove the script via:

```bash
rm scripts/setupTypeScript.js
```

## Deploying to the web

### With [Vercel](https://vercel.com)

Install `vercel` if you haven't already:

```bash
npm install -g vercel
```

Then, from within your project folder:

```bash
cd public
vercel deploy --name my-project
```

### With [surge](https://surge.sh/)

Install `surge` if you haven't already:

```bash
npm install -g surge
```

Then, from within your project folder:

```bash
npm run build
surge public my-project.surge.sh
```
