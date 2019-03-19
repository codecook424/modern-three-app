# threejs-modern-app

> Boilerplate and utils for a fullscreen Three.js app

![demo]()
(assets thanks to poliigon & blender)

Example of a production scale project: [shrimpc.at]()


It is inspired from [mattdesl](https://twitter.com/mattdesl)'s [threejs-app](https://github.com/mattdesl/threejs-app), but it was rewritten and simplified using **ES6** syntax rather than node, making it easier to read and well commented, so it can be easily customized to fit your needs.

## Features
- All the **Three.js boilerplate code is tucked away** in a file, the exported `WebGLApp` is easily configurable from the outside, for example you can enable postprocessing, [orbit controls](https://github.com/Jam3/orbit-controls), [FPS stats](https://github.com/mrdoob/stats.js/), a [control-panel](https://github.com/freeman-lab/control-panel) and use the save screenshot functionality. It also has built-in support for [Cannon.js](https://github.com/schteppe/cannon.js) and [Tween.js](https://github.com/tweenjs/tween.js/). [[Read more](#webglapp)]
- A **scalable Three.js component structure** where each component is a class which extends `THREE.Group`, so you can add any object to it. The class also has update, resize, and touch hooks. [[Read more](#component-structure)]
- An **asset manager** which handles the preloading of `.gltf` models, images, audios, videos and can be easily extended to support other files. It also automatically uploads a texture to the GPU, loads cube env maps or parses equirectangular projection images. [[Read more](#asset-manager)]
- global `window.DEBUG` flag which is true when the url contains `?debug` as a query parameter. So you can enable **debug mode** both locally and in production. [[Read more](#debug-mode)]
- [glslify](https://github.com/glslify/glslify) to import shaders from `node_modules`. [[Read more](#glslify)]
- Hot reload not enabled by default. [[Read more](#hot-reload)]
- Modern and customizable development tools such as webpack, babel, eslint, prettier and browserslist.

## User Guide

### WebGLApp

```js
import WebGLApp from './lib/WebGLApp'

const webgl = new WebGLApp({ ...options })
```

The WebGLApp class contains all the code needed for Three.js to run a scene, it is always the same so it makes sense to hide it in a standalone file and don't think about it.

You can see an example configuration here:

https://github.com/marcofugaro/threejs-modern-app/blob/4af53b2748e2ea923f2e4482c657c894b2b848b3/boilerplate/src/index.js#L18-L52

You can pass the class the options you would pass to the [THREE.WebGLRenderer](https://threejs.org/docs/#api/en/renderers/WebGLRenderer), and also some more options:

| Option | Default | Description |
| --- | --- | --- |
| `background` | `'#000'` | The background of the scene |
| `backgroundAlpha` | 1 | The transparency of the background |
| `maxPixelRatio` | 2 | The clamped pixelRatio, for performance reasons |
| `maxDeltaTime` | 1 / 30 | Clamp the `dt` to prevent stepping anything too far forward |
| `postprocessing` | false | Enable Three.js postprocessing. The composer gets exposed as `webgl.composer`. |
| `showFps` | false | Show the [stats.js](https://github.com/mrdoob/stats.js/) fps counter |
| `orbitControls` | undefined | Accepts an object with the [orbit-controls](https://github.com/Jam3/orbit-controls) options. Exposed as `webgl.orbitControls`. |
| `panelInputs` | undefined | Accepts an array with the [control-panel](https://github.com/freeman-lab/control-panel) inputs. Exposed ad `webgl.panel`. |
| `world` | undefined | Accepts an instance of the [cannon.js](https://github.com/schteppe/cannon.js) world (`new CANNON.World()`). Exposed as `webgl.world`. |
| `tween` | undefined | Accepts the [TWEEN.js](https://github.com/tweenjs/tween.js/) library (`TWEEN`). Exposed as `webgl.tween`. |

The `webgl` instance will contain all the Three.js elements such as `webgl.scene`, `webgl.renderer`, `webgl.camera` or `webgl.canvas`. It also exposes some methods:

#### webgl.saveScreenshot({ ...options })

Save a screenshot of the application as a png.

| Option | Default | Description |
| --- | --- | --- |
| `width` | 2560 | The width of the screenshot |
| `height` | 1440 | The height of the screenshot |
| `fileName` | `'image.png'` | The filename, can be only .png |

#### webgl.onUpdate((dt, time) => {})

Subscribe to the update `requestAnimationFrame` without having to create a component.


### Component structure

TODO

### Asset Manager

The Asseet Manager handles the preloading of all the assets needed to run the scene, you use it like this:

https://github.com/marcofugaro/threejs-modern-app/blob/52cbbb330419ce830eb4cc8c9ef06584f21d1bd7/boilerplate/src/scene/Suzanne.js#L12-L42

In detail, first you queue the asset you want to preload in the component where you will use it

```js
import assets from '../lib/AssetManager'

const key = assets.queue({
  url: 'assets/model.gltf',
  type: 'gltf',
})
```

Then you import the component in the `index.js` so that code gets executed

```js
import Component from './scene/Component'
```

And then you start the queue assets loading promise, always in the `index.js`

```js
assets.load({ renderer: webgl.renderer }).then(() => {
  // assets loaded! we can show the canvas
})
```

After that, you init the component and use the asset in the component like this

```js
const modelGltf = assets.get(key)
```

These are all the exposed methods:

#### assets.queue({ url, type, ...others })

Queue an asset to be downloaded later with `assets.load()`.

| Option | Default | Description |
| --- | --- | --- |
| `url` |  | The url of the asset relative to the `public/` folder. |
| `type` | autodetected | The type of the asset, can be either `gltf`, `image`, `svg`, `texture`, `env-map`, `json`, `audio` or `video`. If omitted it will be discerned from the asset extension. |
| `equirectangular` | false | Only if you set `type: 'env-map'`, you can pass `equirectangular: true` if you have a single [equirectangular image](https://www.google.com/search?q=equirectangular+image&tbm=isch) rather than the six squared subimages. |
| ...others |  | Other options that get passed to [loadEnvMap](https://github.com/marcofugaro/threejs-modern-app/blob/master/boilerplate/src/lib/loadEnvMap.js) or [loadTexture](https://github.com/marcofugaro/threejs-modern-app/blob/master/boilerplate/src/lib/loadTexture.js) when the type is either `env-map` or `texture` |

Returns a `key` that later you can use with `assets.get()`.

#### assets.load({ renderer })

Load all the assets previously queued.

| Option | Default | Description |
| --- | --- | --- |
| `renderer` |  | The WebGLRenderer of your application, exposed as `webgl.renderer` |

#### assets.loadSingle({ url, type, renderer, ...others })

Load a single asset without having to pass through the queue. Useful if you want to lazy-load some assets after the application has started. Usually the assets that are not needed immediately.

| Option | Default | Description |
| --- | --- | --- |
| `renderer` |  | The WebGLRenderer of your application, exposed as `webgl.renderer` |
| `url` |  | The url of the asset relative to the `public/` folder. |
| `type` | autodetected | The type of the asset, can be either `gltf`, `image`, `svg`, `texture`, `env-map`, `json`, `audio` or `video`. If omitted it will be discerned from the asset extension. |
| `equirectangular` | false | Only if you set `type: 'env-map'`, you can pass `equirectangular: true` if you have a single [equirectangular image](https://www.google.com/search?q=equirectangular+image&tbm=isch) rather than the six squared subimages. |
| ...others |  | Other options that get passed to [loadEnvMap](https://github.com/marcofugaro/threejs-modern-app/blob/master/boilerplate/src/lib/loadEnvMap.js) or [loadTexture](https://github.com/marcofugaro/threejs-modern-app/blob/master/boilerplate/src/lib/loadTexture.js) when the type is either `env-map` or `texture` |

Returns a `key` that later you can use with `assets.get()`.

#### assets.addProgressListener((progress) => {})

Pass a function that gets called each time an assets finishes downloading. The argument `progress` goes from 0 to 1, with 1 being every asset queued has been downloaded.

#### assets.get(key)

Retrieve an asset previously loaded with `assets.load()` or `assets.loadSingle()`.

| Option | Default | Description |
| --- | --- | --- |
| `key` |  | The key returned from `assets.queue()` or `assets.loadSingle()`. It corresponds to the url of the asset. |

### Debug mode

Often you want to show the fps count or debug helpers such as the [SpotLightHelper](https://threejs.org/docs/#api/en/helpers/SpotLightHelper) only when you're developing or debugging.

A really manageable way is to have a global `window.DEBUG` constant which is true only if you append `?debug` to your url, for example `http://localhost:8080/?debug` or even in production like `https://example.com/?debug`.

This is done [here](https://github.com/marcofugaro/threejs-modern-app/blob/4af53b2748e2ea923f2e4482c657c894b2b848b3/boilerplate/src/index.js#L13) in just one line:

```js
window.DEBUG = window.location.search.includes('debug')
```

You could also add more global constants by just using more query-string parameters, like this `?debug&fps`.

### glslify

You can import shaders from `node_modules` with glslify, here is an example that uses [glsl-vignette](https://github.com/TyLindberg/glsl-vignette):

https://github.com/marcofugaro/threejs-modern-app/blob/master/boilerplate/src/scene/shaders/vignette.frag

For a list of shaders you can import check out [stack.gl packages list](http://stack.gl/packages/), more info on [glslify's readme](https://github.com/glslify/glslify).

### Hot reload

TODO

## console screenshots