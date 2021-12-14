# pinia-plugin-persistedstate-2

[![CI](https://github.com/iendeavor/pinia-plugin-persistedstate-2/actions/workflows/ci.yml/badge.svg?branch=main)](https://github.com/iendeavor/pinia-plugin-persistedstate-2/actions/workflows/ci.yml)
[![NPM version](https://img.shields.io/npm/v/pinia-plugin-persistedstate-2.svg)](https://www.npmjs.com/package/pinia-plugin-persistedstate-2)
[![Bundle size](https://badgen.net/bundlephobia/minzip/pinia-plugin-persistedstate-2)](https://bundlephobia.com/result?p=pinia-plugin-persistedstate-2)

[![Conventional Commits](https://img.shields.io/badge/Conventional%20Commits-1.0.0-yellow.svg)](https://conventionalcommits.org)
[![Prettier](https://img.shields.io/badge/Code_Style-Prettier-ff69b4.svg)](https://github.com/prettier/prettier)

[![PRs Welcome](https://img.shields.io/badge/PRs-Welcome-brightgreen.svg?style=flat-square)](http://makeapullrequest.com)

## Getting Started

### Installation

#### Package Manager

```sh
# npm
npm i pinia-plugin-persistedstate-2

# yarn
yarn add pinia-plugin-persistedstate-2

# pnpm
pnpm add pinia-plugin-persistedstate-2
```

#### CDN

```html
<script src="https://unpkg.com/pinia-plugin-persistedstate-2"></script>
```

You can find the library on `window.PiniaPluginPersistedstate_2`.

### Usage

```ts
import { createPinia } from 'pinia'
import { createPersistedStatePlugin } from 'pinia-plugin-persistedstate-2'

const pinia = createPinia()
pinia.use(createPersistedStatePlugin())
```

### Examples

[Vue 3](https://codesandbox.io/s/github/iendeavor/pinia-plugin-persistedstate-2/tree/main/examples/vue3-example?fontsize=14&hidenavigation=1&theme=dark&view=preview)

[localForage (asynchronous storage)](https://codesandbox.io/s/github/iendeavor/pinia-plugin-persistedstate-2/tree/main/examples/localforage-example?fontsize=14&hidenavigation=1&theme=dark&view=preview)

[Nuxt.js (client-only, with localStorage)](https://codesandbox.io/s/github/iendeavor/pinia-plugin-persistedstate-2/tree/main/examples/nuxtjs-client-example?fontsize=14&hidenavigation=1&theme=dark&view=preview)

[Nuxt3 (universal, with cookies)](https://codesandbox.io/s/github/iendeavor/pinia-plugin-persistedstate-2/tree/main/examples/nuxt3-universal-example?fontsize=14&hidenavigation=1&theme=dark&view=preview)

## SSR

### Nuxt.js

Follow [Pinia - Nuxt.js installation steps](https://pinia.esm.dev/ssr/nuxt.html#installation).

```js
// nuxt.config.js
export default {
  // ... other options
  buildModules: [
    // Nuxt 2 only:
    // https://composition-api.nuxtjs.org/getting-started/setup#quick-start
    '@nuxtjs/composition-api/module',
    '@pinia/nuxt',
  ],
}
```

### With localStorage (client-only)

Create the plugin below to plugins config in your nuxt.config.js file.

```js
// nuxt.config.js
export default {
  // ... other options
  plugins: ['@/plugins/persistedstate.js'],
}
```

```ts
// plugins/persistedstate.js
import { createPersistedStatePlugin } from 'pinia-plugin-persistedstate-2'

export default function ({ $pinia }) {
  if (process.client) {
    $pinia.use(createPersistedStatePlugin())
  }
}
```

### With cookies (universal)

```js
// nuxt.config.js
export default {
  // ... other options
  plugins: ['@/plugins/persistedstate.js'],
}
```

```ts
// plugins/persistedstate.js
import { createPersistedStatePlugin } from 'pinia-plugin-persistedstate-2'
import Cookies from 'js-cookie'
import cookie from 'cookie'

export default function ({ $pinia, ssrContext /* Nuxt 3 example */ }) {
  $pinia.use(
    createPersistedStatePlugin({
      storage: {
        getItem: (key) => {
          // See https://nuxtjs.org/guide/plugins/#using-process-flags
          if (process.server) {
            const parsedCookies = cookie.parse(ssrContext.req.headers.cookie)
            return parsedCookies[key]
          } else {
            return Cookies.get(key)
          }
        },
        // Please see https://github.com/js-cookie/js-cookie#json, on how to handle JSON.
        setItem: (key, value) =>
          Cookies.set(key, value, { expires: 365, secure: false }),
        removeItem: (key) => Cookies.remove(key),
      },
    }),
  )
}
```

## API

For more details, see [type.ts](./src/type.ts).

### Common Options

- `persist?: boolean`: Defaults to `true`. Whether to persist store.

- `storage?: IStorage`: Defaults to `localStorage`. Where to store persisted state.

- `assertStorage?: (storage: IStorage) => void | never`: Perform a Write-Delete operation by default. To ensure `storage` is available.

- `overwrite?: boolean`: Defaults to `false`. Whether to overwrite initial state when rehydrating. When this flat is true use `store.$state = persistedState`, `store.$patch(persistedState)` otherwise.

- `serialize?: (value: any): any`: Defaults to `JSON.stringify`. This method will be called right before `storage.setItem`.

- `deserialize?: (value: any): any`: Defaults to `JSON.parse`. This method will be called right after `storage.getItem`.

- `filter: (mutation, state): boolean`: A function that will be called to filter any mutations which will trigger setState on storage eventually.

#### IStorage

- `getItem: (key: string) => any | Promise<any>`: Any value other than `undefined` or `null` will be rehydrated.

- `setItem: (key: string, value: any) => void | Promise<void>`

- `removeItem: (key: string) => void | Promise<void>`

### Plugin Options

> Supports all [common options](#Common-Options). These options are the default values for each store, you can set the most commonly used options in the _plugin options_, and override/extend it in the _store options_.

```ts
createPersistedStatePlugin({
  // plugin options goes here
})
```

### Store Options

> Supports all [common options](#Common-Options).

```ts
defineStore(
  'counter-store',
  () => {
    const currentValue = ref(0)
    const increment = () => currentValue.value++

    return {
      currentValue,
      increment,
    }
  },
  {
    persistedState: {
      // store options goes here
    },
  },
)
```

- `key?: string`: Defaults to `store.$id`. The key to store the persisted state under.

- `includePath?: string[]`: An array of any paths to partially persist the state.

- `excludePath?: string[]`

### Store Properties

- `store.$persistedState.isReady: () => Promise<void>`: Whether store is hydrated

- `store.$persistedState.pending: boolean`: Whether store is persisting

## Contributing

Please read [CONTRIBUTING.md](/CONTRIBUTING.md) for details on our code of conduct, and the process for submitting pull
requests to us.

## Versioning

This project use [SemVer](https://semver.org/) for versioning. For the versions available, see the tags on this repository.

## License

This project is licensed under the MIT License - see the [LICENSE](/LICENSE) file for details
