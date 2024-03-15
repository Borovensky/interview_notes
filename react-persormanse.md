Categories of performance issues

### Unnecessary renders.
    Memoization
    - React.memo()
    - useCallback/useMemo a changing func/obj
    - useMemo a chunk of jsx
    - Default object/array: lift upwards
    Context & Refactorings
    - useContext: memoize the passed value
    - useContext: useContextSelector (thirty part library)
    - Move the changing state into a separete leaf component

### Expensice renders & Effects.
    Mark them cheaper
    - Wrap the expensive computation with useMemo()
    - Wrap the function that computes the value with _.memoize or memoize-one
    - Make the algorithm cheaper (e.g., avoid deep comparisons or deep cloning)
    - Split one long computation into several short ones with isInputPending()
    - Move the computation to a web worker (e.g., using comlink)
    - Move the computation to the server or build time
    - Virtualization: remove unnecessary DOM nodes (react-virtuoso, react-window)
    - MobX use observable.ref or observable.shallow instead of observable
    Make them less frequent
    - Debounce: _.debounce or useDebounce
    - Throttle: _.throttle or useThrottle
    - Delay with requestIdleCallback
    - React 18: startTransition / useDeferredValue (instead of debounce/throttle) 


- Chains of renders.
- Layout traching.
- Hydration.

https://chromedevtools.github.io/devtools-protocol/tot/Emulation/#method-setCPUThrottlingRate 
https://pptr.dev/
https://developer.mozilla.org/en-US/docs/Web/API/PerformanceObserver
