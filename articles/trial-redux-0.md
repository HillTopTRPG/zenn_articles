---
title: "Redux はじめました #0"
emoji: "🤩"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Docker", "Vite", "React", "TypeScript", "Redux"]
published: true
published_at: 2024-08-30 09:10
---

## Redux はじめました #0

Reduxを使ってみたい！  
と言うことで `Hello World!` レベルのプロジェクトを作っていきます。

## 今回の目標

Reduxを導入して基本的な状態管理の機能を使ってみる

## 構成・使うもの

- Docker
- Vite
- React
- TypeScript
- Redux

※ GitHubリポジトリは作りません

## 0. 前提環境

- docker composeが動くところからスタート
- macOS上で試してます（難しいコマンドは使ってないので適宜読み換えてください）

## 1. Docker

### 1-1. フォルダの準備

```sh
# ルートフォルダ
mkdir redux-sample
# フロントエンドのコンテナ用のフォルダ
mkdir redux-sample/frontend
```

### 1-2. 初期ファイルの設置

```yml:./docker-compose.yml
version: '3.7'

services:
  frontend:
    build: ./frontend
    working_dir: /app/redux-sample
    tty: true
# あとでコメントインする①
#   ports:
#     - "82:5173"
    volumes:
      - './frontend:/app/redux-sample'
# あとでコメントインする②
#   command: bash -c "npm i && npm run dev"

```

```Docker:./frontend/Dockerfile
FROM node:20-alpine3.18

WORKDIR /app/react-sample

ENV NPM_CONFIG_PREFIX /home/node/.npm-global
ENV PATH $PATH:/home/node/.npm-global/bin

RUN apk upgrade --no-cache && \
    apk add --update --no-cache vim git make curl bash && \
    yarn global add vite

```

### 1-3. コンテナ作成 & 起動

```sh
# 起動しようとしてイメージのビルドからやってくれる（少しだけ時間がかかる）
docker compose up -d
# 起動したコンテナに入る
docker compose exec frontend bash
```

これでDockerコンテナ内でNode.jsが使えるようになりました。

## 2. Vite + React

### 2-1. プロジェクト作成

公式情報を元にコマンドを叩く

https://vitejs.dev/guide/

```sh
npm create vite@latest . -- --template react-swc-ts
npm i
```

コマンドが成功すると `./frontend` フォルダの中にプロジェクトファイルが生成された状態になります。

### 2-2. ローカルサーバーの公開設定

ホストPCからローカルサーバーで公開したページにアクセスするための準備その１です。  
2-1 で生成された package.json を編集します。

```diff json:./frontend/package.json
{
  "name": "redux-sample",
  "private": true,
  "version": "0.0.0",
  "type": "module",
  "scripts": {
+    "dev": "vite --host --port 5173",
-    "dev": "vite",
    "build": "tsc -b && vite build",
    "lint": "eslint .",
    "preview": "vite preview"
  },
  ...以下略...
}

```
※ デフォルトのポートは `5173` なので `--port 5173` は冗長ですが、分かりやすさのため記述しています

### 2-3. Dockerコンテナのポートフォワードの設定

ホストPCからローカルサーバーで公開したページにアクセスするための準備その２です。  
1-2 で作成した ./docker-compose.yml の①の部分をコメントインします。

```diff yml:./docker-compose.yml
version: '3.7'

services:
  frontend:
    build: ./frontend
    working_dir: /app/redux-sample
    tty: true
+    ports:
+      - "82:5173"
-# あとでコメントインする①
-#   ports:
-#     - "82:5173"
    volumes:
      - './frontend:/app/redux-sample'
# あとでコメントインする②
#   command: bash -c "npm i && npm run dev"

```

docker-compose.yml が編集できたらコンテナを立ち上げ直して反映させましょう。

```sh
# 1-3 で入ったコンテナから抜ける
exit
# ホストPCに戻ってきたのでコンテナを停止
docker compose stop
# 起動し直して再度コンテナに入る
docker compose up -d
docker compose exec frontend bash
```

### 2-4. ローカルサーバーの起動

```sh
npm run dev
```

```sh:出力例
  VITE v5.4.2  ready in 808 ms

  ➜  Local:   http://localhost:5173/
  ➜  Network: http://172.27.0.2:5173/
  ➜  press h + enter to show help

```

出力の通り、Dockerコンテナ内では5173番ポートで公開されています。  
```json:これは ./frontend/package.json の設定のおかげ
  "scripts": {
    "dev": "vite --host --port 5173",
```

そして、docker composeのポートフォワードによって82番ポートに紐付けられています。

```yml:これは ./docker-compose.yml の設定のおかげ
   ports:
     - "82:5173"
```

というわけで、ホストPCのブラウザ（普段使っているブラウザ）からは下記URLで表示を確認します。

http://localhost:82/

![](/images/trial-redux-0/01-vite-react-ts-init.png)
*うまくいくとブラウザにこのような画面が表示される*

:::message
**表示できない場合の問題の切り分け方法**
- ローカルサーバーが起動しているかどうか
  - Dockerコンテナ内から `curl` コマンドを使ってページにアクセス
- ポートフォワードが失敗しているかどうか
  - `docker compose ps` で `PORTS` の列を確認する
:::

### 2-5. コンテナ起動時に自動的にローカルサーバーを起動するようにする


1-2 で作成した ./docker-compose.yml の②の部分をコメントインします。

```diff yml:./docker-compose.yml
version: '3.7'

services:
  frontend:
    build: ./frontend
    working_dir: /app/redux-sample
    tty: true
    ports:
      - "82:5173"
    volumes:
      - './frontend:/app/redux-sample'
+   command: bash -c "npm i && npm run dev"
-# あとでコメントインする②
-#   command: bash -c "npm i && npm run dev"

```

2-3 と同様にコンテナを再起動して上記修正を反映させます。

```sh
# 2-4 で起動したローカルサーバーを停止するために `control + c` キーを押す(macOSの場合)

# 2-3 で入ったコンテナから抜ける
exit
# ホストPCに戻ってきたのでコンテナを停止
docker compose stop
# 今回はコンテナを起動するだけ
docker compose up -d
```

コメントインした部分のおかげで、2-4 で叩いた下記コマンドは手動で叩く必要がなくなりました。

```sh
npm run dev
```

![](/images/trial-redux-0/01-vite-react-ts-init.png)
*コンテナ起動後、すぐに（少し待つ）http://localhost:82/ にアクセスして表示を確認できるようになった*

## 3. Redux

基本的には公式情報（わかりやすかった）に従えば問題なかったです。

https://redux.js.org/tutorials/quick-start#usage-summary

現在時点で実際にどうやったのかを下記に残しておきます。
※ 拡張子は js となっているものは適宜 ts や tsx に読み換えて実施しました

```sh
npm install @reduxjs/toolkit react-redux
```

Reduxのストア（複数のスライスへの参照を持つことができる）を作成して
```ts:./frontend/src/app/store.tsを作成
import { configureStore } from '@reduxjs/toolkit'

export default configureStore({
  reducer: {}
})

```

ストアのスコープとなるコンポーネントをProviderコンポーネント（ストアを参照）で囲います。
```diff tsx:./frontend/src/main.tsxを編集
import { StrictMode } from 'react'
import { createRoot } from 'react-dom/client'
import App from './App.tsx'
import './index.css'
+import store from './app/store'
+import { Provider } from 'react-redux'

createRoot(document.getElementById('root')!).render(
  <StrictMode>
+    <Provider store={store}>
+      <App />
+    </Provider>
-    <App />
  </StrictMode>,
)

```
上記の書きっぷりから、Reduxはデザインパターンで言うところのプロバイダパターンで実装されていることがわかります。

https://zenn.dev/morinokami/books/learning-patterns-1/viewer/provider-pattern

ストアから参照するスライス（データの入れ物のstateと更新処理のreducerのひとまとまり）を定義
```ts:./frontend/src/features/counter/counterSlice.tsを作成
import { createSlice } from '@reduxjs/toolkit'

export const counterSlice = createSlice({
  name: 'counter',
  initialState: {
    value: 0
  },
  reducers: {
    increment: state => {
      // Redux Toolkit allows us to write "mutating" logic in reducers. It
      // doesn't actually mutate the state because it uses the Immer library,
      // which detects changes to a "draft state" and produces a brand new
      // immutable state based off those changes
      state.value += 1
    },
    decrement: state => {
      state.value -= 1
    },
    incrementByAmount: (state, action) => {
      state.value += action.payload
    }
  }
})

// Action creators are generated for each case reducer function
export const { increment, decrement, incrementByAmount } = counterSlice.actions

export default counterSlice.reducer

```

ストアに追加しました。
```diff ts:./frontend/src/app/store.tsを編集
import { configureStore } from '@reduxjs/toolkit'
+import counterReducer from '../features/counter/counterSlice'

export default configureStore({
+  reducer: {
+    counter: counterReducer
+  }
-  reducer: {}
})

```

そして、スライスを利用するコンポーネントを作りました。
```diff tsx:./frontend/src/features/counter/Counter.tsxを作成（一部改変）
-import React from 'react'
import { useSelector, useDispatch } from 'react-redux'
import { decrement, increment } from './counterSlice'
-import styles from './Counter.module.css'

export function Counter() {
  const count = useSelector(state => state.counter.value)
  const dispatch = useDispatch()

  return (
    <div>
      <div>
        <button
          aria-label="Increment value"
          onClick={() => dispatch(increment())}
        >
          Increment
        </button>
        <span>{count}</span>
        <button
          aria-label="Decrement value"
          onClick={() => dispatch(decrement())}
        >
          Decrement
        </button>
      </div>
    </div>
  )
}

```

これで `<Counter />` コンポーネントが `<App />` の中で動くようになったはずです。  

早速追加してみます。  
コンポーネントを跨いで値が保持されていることを確認するために２つ追加してみます。

```diff tsx:./frontend/src/App.tsxを編集
import { useState } from 'react'
import reactLogo from './assets/react.svg'
import viteLogo from '/vite.svg'
import './App.css'
+import { Counter } from './features/counter/Counter.tsx'


function App() {
  const [count, setCount] = useState(0)

  return (
    <>
      <div>
        <a href="https://vitejs.dev" target="_blank">
          <img src={viteLogo} className="logo" alt="Vite logo" />
        </a>
        <a href="https://react.dev" target="_blank">
          <img src={reactLogo} className="logo react" alt="React logo" />
        </a>
      </div>
      <h1>Vite + React</h1>
      <div className="card">
        <button onClick={() => setCount((count) => count + 1)}>
          count is {count}
        </button>
        <p>
          Edit <code>src/App.tsx</code> and save to test HMR
        </p>
      </div>
      <p className="read-the-docs">
        Click on the Vite and React logos to learn more
      </p>
+      <Counter />
+      <Counter />
    </>
  )
}

export default App

```

![](/images/trial-redux-0/02-use-counter-component.png)
*画面下部に追加したコンポーネントが表示された*

![](/images/trial-redux-0/03-incremented-counter.png)
*Incrementボタンを1回押すと全てのコンポーネントの値が変化した*

## 4. TypeScript

前項までで当初の目的は果たしましたが、TypeScriptとしての書き方も勉強しておきます。

https://redux.js.org/tutorials/typescript-quick-start#project-setup

なるほど、下記の `state.counter` の部分の型をいい感じにしようとする書きっぷりを教えてくれるんですね。

```tsx:./frontend/src/features/counter/Counter.tsx
  const count = useSelector(state => state.counter.value)
```

ストアに型定義のexportを追加
```diff ts:./frontend/src/app/store.tsを編集
import { configureStore } from '@reduxjs/toolkit'

+export const store = configureStore({
-export default configureStore({
  reducer: {}
})

+// Infer the `RootState`,  `AppDispatch`, and `AppStore` types from the store itself
+export type RootState = ReturnType<typeof store.getState>
+// Inferred type: {posts: PostsState, comments: CommentsState, users: UsersState}
+export type AppDispatch = typeof store.dispatch
+export type AppStore = typeof store

```

（公式情報では載ってないけど）デフォルトエクスポートではなくなったので参照を変更

```diff tsx:./frontend/src/main.tsxを編集
import { StrictMode } from 'react'
import { createRoot } from 'react-dom/client'
import App from './App.tsx'
import './index.css'
+import { store } from './app/store'
-import store from './app/store'
import { Provider } from 'react-redux'

...以下略...
```

`useSelector`, `useDispatch` のかわりに使う関数を定義します（フックと呼ぶらしいです）

```ts:./src/app/hooks.ts
import { useDispatch, useSelector } from 'react-redux'
import type { AppDispatch, RootState } from './store'

// Use throughout your app instead of plain `useDispatch` and `useSelector`
export const useAppDispatch = useDispatch.withTypes<AppDispatch>()
export const useAppSelector = useSelector.withTypes<RootState>()

```

フックを利用するように編集
```diff tsx:./frontend/src/features/counter/Counter.tsxを編集
+import { useAppSelector, useAppDispatch } from '../../app/hooks'
-import { useSelector, useDispatch } from 'react-redux'
import { decrement, increment } from './counterSlice'

export function Counter() {
+  // The `state` arg is correctly typed as `RootState` already
+  const count = useAppSelector(state => state.counter.value)
+  const dispatch = useAppDispatch()
-  const count = useSelector(state => state.counter.value)
-  const dispatch = useDispatch()

  return (
    <div>
...以下略...
```

スライスについてもTypeScriptの書きっぷりが提示されていました。
```diff ts:./frontend/src/features/counter/counterSlice.tsを編集
import { createSlice } from '@reduxjs/toolkit'
+import type { RootState } from '../../app/store'

+// Define a type for the slice state
+export interface CounterState {
+  value: number
+}

+// Define the initial state using that type
+const initialState: CounterState = {
+  value: 0
+}

export const counterSlice = createSlice({
  name: 'counter',
+  initialState,
-  initialState: {
-    value: 0
-  },
  reducers: {
    increment: state => {
      // Redux Toolkit allows us to write "mutating" logic in reducers. It
      // doesn't actually mutate the state because it uses the Immer library,
      // which detects changes to a "draft state" and produces a brand new
      // immutable state based off those changes
      state.value += 1
    },
    decrement: state => {
      state.value -= 1
    },
+    // Use the PayloadAction type to declare the contents of `action.payload`
+    incrementByAmount: (state, action: PayloadAction<number>) => {
-    incrementByAmount: (state, action) => {
      state.value += action.payload
    }
  }
})

// Action creators are generated for each case reducer function
export const { increment, decrement, incrementByAmount } = counterSlice.actions

+// Other code such as selectors can use the imported `RootState` type
+export const selectCount = (state: RootState) => state.counter.value

export default counterSlice.reducer

```

initialStateの宣言の部分でTypeScriptが警告を出してくるようなら、下記のような書きっぷりがいいぞと書いてありました。  
```ts
// Workaround: cast state instead of declaring variable type
const initialState = {
  value: 0
} as CounterState
```
TypeScript 4.9以降であれば、`as` より `as const satisfies` の方がいいかもですね。

`export const selectCount` を利用していないようなので一瞬「はて」となりましたが、  
利用するとしたら下記のような書きっぷりになると思われます。

```diff tsx:./frontend/src/features/counter/Counter.tsxを編集
import { useAppSelector, useAppDispatch } from '../../app/hooks'
+import { decrement, increment, selectCount } from './counterSlice'
-import { decrement, increment } from './counterSlice'

export function Counter() {
  // The `state` arg is correctly typed as `RootState` already
+  const count = useAppSelector(selectCount)
-  const count = useAppSelector(state => state.counter.value)
  const dispatch = useAppDispatch()

  return (
    <div>
...以下略...
```

## まとめ

無事にReduxを導入でき、最も基本的な状態管理の機能を体感することができました。  
非同期処理(API呼び出しなど)を伴う状態管理やその他Reduxならではの強力な機能については、また別の機会にやってみようと思います。
