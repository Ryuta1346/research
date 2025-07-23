# Zustand 完全ガイド：基本的な使い方とベストプラクティス

## 目次
1. [Zustandとは](#zustandとは)
2. [他の状態管理ライブラリとの比較](#他の状態管理ライブラリとの比較)
3. [基本的な使い方](#基本的な使い方)
4. [TypeScriptとの統合](#typescriptとの統合)
5. [ベストプラクティス](#ベストプラクティス)
6. [高度な機能](#高度な機能)
7. [実践的なパターン](#実践的なパターン)
8. [テスト方法](#テスト方法)
9. [まとめ](#まとめ)

## Zustandとは

Zustandは、Reactアプリケーション向けの軽量で柔軟な状態管理ライブラリです。pmndrs（Poimandres）によって開発され、以下の特徴を持っています：

- **超軽量**: わずか2KB（gzip圧縮時）
- **シンプルなAPI**: 学習曲線が緩やか
- **TypeScript対応**: 優れた型推論サポート
- **React外での使用可能**: Vanilla JSでも動作
- **ミドルウェアシステム**: 拡張可能な機能

### 主な特徴

1. **Flux原則に基づく**: 単一方向のデータフロー
2. **ボイラープレートが少ない**: Reduxと比較して記述量が大幅に削減
3. **React Suspenseとの互換性**: 非同期処理も簡単
4. **DevToolsサポート**: Redux DevToolsで状態を可視化

## 他の状態管理ライブラリとの比較

### Redux vs Zustand

**Redux:**
```typescript
// Redux (Redux Toolkit使用)
import { createSlice, configureStore } from '@reduxjs/toolkit'

const countSlice = createSlice({
  name: 'count',
  initialState: { value: 0 },
  reducers: {
    incremented: (state, action) => {
      state.value += action.payload
    },
    decremented: (state, action) => {
      state.value -= action.payload
    },
  },
})

const store = configureStore({ reducer: countSlice.reducer })
```

**Zustand:**
```typescript
import { create } from 'zustand'

const useCountStore = create((set) => ({
  count: 0,
  increment: (qty) => set((state) => ({ count: state.count + qty })),
  decrement: (qty) => set((state) => ({ count: state.count - qty })),
}))
```

### 主な違い

| 特徴 | Zustand | Redux | Jotai | Recoil | Valtio |
|------|---------|--------|-------|---------|---------|
| バンドルサイズ | 2KB | 10KB+ | 12KB | 21KB | 5KB |
| 学習曲線 | 緩やか | 急 | 中程度 | 急 | 緩やか |
| ボイラープレート | 最小 | 多い | 少ない | 中程度 | 最小 |
| TypeScript | 優秀 | 良好 | 優秀 | 良好 | 良好 |
| レンダリング最適化 | 手動 | 手動 | 自動 | 自動 | 自動 |
| 状態モデル | 単一ストア | 単一ストア | アトミック | アトミック | プロキシ |

## 基本的な使い方

### 1. ストアの作成

```typescript
import { create } from 'zustand'

interface BearState {
  bears: number
  increase: () => void
  removeAllBears: () => void
}

const useBearStore = create<BearState>()((set) => ({
  bears: 0,
  increase: () => set((state) => ({ bears: state.bears + 1 })),
  removeAllBears: () => set({ bears: 0 }),
}))
```

### 2. コンポーネントでの使用

```tsx
function BearCounter() {
  const bears = useBearStore((state) => state.bears)
  return <h1>{bears} around here...</h1>
}

function Controls() {
  const increase = useBearStore((state) => state.increase)
  return <button onClick={increase}>one up</button>
}
```

### 3. 非同期アクション

```typescript
const useFishStore = create((set) => ({
  fishes: {},
  fetch: async (pond) => {
    const response = await fetch(pond)
    set({ fishes: await response.json() })
  },
}))
```

### 4. 状態の読み取り（get）

```typescript
const useSoundStore = create((set, get) => ({
  sound: 'grunt',
  action: () => {
    const sound = get().sound
    // 現在の状態を参照して処理を実行
    console.log(sound)
  }
}))
```

## TypeScriptとの統合

### 1. 基本的な型定義

```typescript
import { create } from 'zustand'

interface BearState {
  bears: number
  increase: (by: number) => void
}

// カリー化されたアプローチ（推奨）
const useBearStore = create<BearState>()(
  (set) => ({
    bears: 0,
    increase: (by) => set((state) => ({ bears: state.bears + by })),
  })
)
```

### 2. ミドルウェアとの組み合わせ

```typescript
import { create } from 'zustand'
import { devtools, persist } from 'zustand/middleware'

interface BearState {
  bears: number
  increase: (by: number) => void
}

const useBearStore = create<BearState>()(
  devtools(
    persist(
      (set) => ({
        bears: 0,
        increase: (by) => set((state) => ({ bears: state.bears + by })),
      }),
      { name: 'bear-storage' }
    )
  )
)
```

### 3. Slicesパターン

```typescript
import { create, StateCreator } from 'zustand'

interface BearSlice {
  bears: number
  addBear: () => void
  eatFish: () => void
}

interface FishSlice {
  fishes: number
  addFish: () => void
}

interface SharedSlice {
  addBoth: () => void
  getBoth: () => number
}

const createBearSlice: StateCreator<
  BearSlice & FishSlice,
  [],
  [],
  BearSlice
> = (set) => ({
  bears: 0,
  addBear: () => set((state) => ({ bears: state.bears + 1 })),
  eatFish: () => set((state) => ({ fishes: state.fishes - 1 })),
})

const createFishSlice: StateCreator<
  BearSlice & FishSlice,
  [],
  [],
  FishSlice
> = (set) => ({
  fishes: 0,
  addFish: () => set((state) => ({ fishes: state.fishes + 1 })),
})

const createSharedSlice: StateCreator<
  BearSlice & FishSlice,
  [],
  [],
  SharedSlice
> = (set, get) => ({
  addBoth: () => {
    get().addBear()
    get().addFish()
  },
  getBoth: () => get().bears + get().fishes,
})

const useBoundStore = create<BearSlice & FishSlice & SharedSlice>()((...a) => ({
  ...createBearSlice(...a),
  ...createFishSlice(...a),
  ...createSharedSlice(...a),
}))
```

## ベストプラクティス

### 1. セレクターによるレンダリング最適化

```typescript
// ❌ 良くない例：ストア全体を購読
const state = useBearStore()

// ✅ 良い例：必要な部分のみを購読
const bears = useBearStore((state) => state.bears)
const increase = useBearStore((state) => state.increase)
```

### 2. アクションの設計

```typescript
// ✅ アクションをストア内に含める（推奨）
const useBoundStore = create((set) => ({
  count: 0,
  text: 'hello',
  inc: () => set((state) => ({ count: state.count + 1 })),
  setText: (text) => set({ text }),
}))

// 別の方法：モジュールレベルでアクションを定義
export const inc = () =>
  useBoundStore.setState((state) => ({ count: state.count + 1 }))
export const setText = (text) => useBoundStore.setState({ text })
```

### 3. 状態の正規化

```typescript
interface TodoStore {
  todos: Record<string, Todo>
  todoIds: string[]
  addTodo: (todo: Todo) => void
}

const useTodoStore = create<TodoStore>((set) => ({
  todos: {},
  todoIds: [],
  addTodo: (todo) =>
    set((state) => ({
      todos: { ...state.todos, [todo.id]: todo },
      todoIds: [...state.todoIds, todo.id],
    })),
}))
```

### 4. パフォーマンスの最適化

```typescript
// shallow比較を使用
import { shallow } from 'zustand/shallow'

const { bears, increase } = useBearStore(
  (state) => ({ bears: state.bears, increase: state.increase }),
  shallow
)

// 頻繁に変更される値には`subscribe`を使用
const Component = () => {
  const scratchRef = useRef(useScratchStore.getState().scratches)
  
  useEffect(() => 
    useScratchStore.subscribe(
      state => (scratchRef.current = state.scratches)
    ), []
  )
}
```

## 高度な機能

### 1. ミドルウェア

#### Persist（永続化）

```typescript
import { create } from 'zustand'
import { persist, createJSONStorage } from 'zustand/middleware'

const useBearStore = create(
  persist(
    (set, get) => ({
      bears: 0,
      addABear: () => set({ bears: get().bears + 1 }),
    }),
    {
      name: 'food-storage',
      storage: createJSONStorage(() => sessionStorage),
      partialize: (state) => ({ bears: state.bears }),
    }
  )
)
```

#### DevTools

```typescript
const useStore = create(
  devtools(
    (set) => ({
      bears: 0,
      addBear: () => 
        set(
          (state) => ({ bears: state.bears + 1 }), 
          undefined, 
          'bear/addBear'
        ),
    })
  )
)
```

#### Immer

```typescript
import { immer } from 'zustand/middleware/immer'

const useStore = create(
  immer((set) => ({
    bees: 0,
    addBees: (by) =>
      set((state) => {
        state.bees += by
      }),
  }))
)
```

### 2. カスタムミドルウェア

```typescript
const logger = (config) => (set, get, api) =>
  config(
    (...args) => {
      console.log('  applying', args)
      set(...args)
      console.log('  new state', get())
    },
    get,
    api
  )

const useStore = create(
  logger((set) => ({
    bees: false,
    setBees: (input) => set({ bees: input }),
  }))
)
```

### 3. 特殊な永続化パターン

```typescript
// URL Search Parametersに保存
const searchParamsStorage = {
  getItem: (key) => new URL(location.href).searchParams.get(key),
  setItem: (key, value) => {
    const searchParams = new URL(location.href).searchParams
    searchParams.set(key, value)
    window.history.replaceState({}, '', `${location.pathname}?${searchParams}`)
  },
  removeItem: (key) => {
    const searchParams = new URL(location.href).searchParams
    searchParams.delete(key)
    window.history.replaceState({}, '', `${location.pathname}?${searchParams}`)
  },
}

const useStore = create(
  persist(
    (set) => ({
      position: { x: 0, y: 0 },
      setPosition: (position) => set({ position }),
    }),
    {
      name: 'position-storage',
      storage: createJSONStorage(() => searchParamsStorage),
    }
  )
)
```

## 実践的なパターン

### 1. Next.jsでの使用

```typescript
// ストアファクトリー
export const createCounterStore = (initState = { count: 0 }) => {
  return createStore<CounterStore>()((set) => ({
    ...initState,
    decrementCount: () => set((state) => ({ count: state.count - 1 })),
    incrementCount: () => set((state) => ({ count: state.count + 1 })),
  }))
}

// Provider
export const CounterStoreProvider = ({ children, ...props }) => {
  const storeRef = useRef<CounterStoreApi>()
  if (!storeRef.current) {
    storeRef.current = createCounterStore(props)
  }
  return (
    <CounterStoreContext.Provider value={storeRef.current}>
      {children}
    </CounterStoreContext.Provider>
  )
}
```

### 2. エラーハンドリング

```typescript
interface Store {
  data: Data | null
  error: Error | null
  loading: boolean
  fetch: () => Promise<void>
}

const useStore = create<Store>((set) => ({
  data: null,
  error: null,
  loading: false,
  fetch: async () => {
    set({ loading: true, error: null })
    try {
      const response = await fetch('/api/data')
      if (!response.ok) throw new Error('Failed to fetch')
      const data = await response.json()
      set({ data, loading: false })
    } catch (error) {
      set({ error: error as Error, loading: false })
    }
  },
}))
```

### 3. Reducerパターン

```typescript
const countReducer = (state: State, action: Action) => {
  switch (action.type) {
    case 'increment':
      return { count: state.count + action.qty }
    case 'decrement':
      return { count: state.count - action.qty }
    default:
      return state
  }
}

const useCountStore = create<State & Actions>((set) => ({
  count: 0,
  dispatch: (action) => set((state) => countReducer(state, action)),
}))
```

## テスト方法

### 1. 基本的なテスト

```typescript
// テスト用のストア作成
export const counterStoreCreator: StateCreator<CounterStore> = (set) => ({
  count: 1,
  inc: () => set((state) => ({ count: state.count + 1 })),
})

// テスト
describe('Counter', () => {
  test('should increase count by clicking a button', async () => {
    const user = userEvent.setup()
    render(<Counter />)
    
    expect(await screen.findByText(/^1$/)).toBeInTheDocument()
    await user.click(await screen.findByRole('button', { name: /one up/i }))
    expect(await screen.findByText(/^2$/)).toBeInTheDocument()
  })
})
```

### 2. モックストア

```typescript
// setup-jest.ts
import { create as actualCreate } from 'zustand'
import { act } from '@testing-library/react'

const storeResetFns = new Set<() => void>()

beforeEach(() => {
  act(() => storeResetFns.forEach((resetFn) => resetFn()))
})

export const create = (createState: StateCreator) => {
  const store = actualCreate(createState)
  const initialState = store.getState()
  storeResetFns.add(() => store.setState(initialState, true))
  return store
}
```

## まとめ

Zustandは、シンプルさと柔軟性を兼ね備えた優れた状態管理ライブラリです。以下の点で特に優れています：

### 長所
- **学習コストが低い**: 直感的なAPIで、すぐに使い始められる
- **軽量**: わずか2KBで、パフォーマンスへの影響が最小限
- **TypeScript対応**: 優れた型推論と型安全性
- **拡張性**: ミドルウェアシステムで機能を追加可能
- **React外での使用**: Vanilla JSでも動作するため、テストが容易

### 短所
- **レンダリング最適化は手動**: セレクターを適切に使用する必要がある
- **大規模アプリケーション**: 非常に複雑な状態管理には向かない場合がある
- **エコシステム**: ReduxやMobXと比較すると、周辺ツールが少ない

### 推奨される使用場面
- 中小規模のReactアプリケーション
- パフォーマンスが重要なアプリケーション
- 素早くプロトタイプを作成したい場合
- Reduxの複雑さを避けたい場合

### 使用を避けるべき場面
- 非常に複雑な状態管理が必要な大規模エンタープライズアプリケーション
- タイムトラベルデバッグが必須の場合
- 既存のReduxエコシステムに深く依存している場合

Zustandは、「必要十分」な機能を提供することで、開発者の生産性を最大化します。シンプルさを保ちながら、実用的な機能を提供するバランスの取れたライブラリと言えるでしょう。