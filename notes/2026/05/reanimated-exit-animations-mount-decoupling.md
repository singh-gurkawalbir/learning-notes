---
title: "Reanimated exit animations: decoupling mount from animation state"
slug: "reanimated-exit-animations-mount-decoupling"
type: "concept"
tags: ["react-native", "reanimated", "animations", "lifecycle", "ui-thread"]
summary: "Why Reanimated's auto exiting animations sometimes get cut short, and the shared-value pattern that takes manual control over the fade-then-unmount lifecycle."
created: 2026-05-08
updated: 2026-08-15
source_question: "Why does my Reanimated exit animation get cut short, and how do I make a fade-out play through its full duration before unmount?"
links:
review:
  last_reviewed: "2026-08-15"
  next_review: 2026-08-18
  step: 2
  confidence: 2
quiz:
  - q: "A child Animated.View has `exiting={FadeOut.duration(200)}`. Logs show React unmounts the component ~119ms after the trigger, not ~200ms. What's the most likely cause?"
    a: "The parent component is also unmounting (or destabilising) at the same moment, so Reanimated can't keep the native view alive for the full exit duration. Either stabilise the parent (always render the slot, conditionally render only the child) or take manual control via shared values."
  - q: "Why decouple `visible` (animation direction) from `mounted` (React tree presence) in the parent hook?"
    a: "So the React component stays in the tree for the full exit-animation duration. If they're combined, React unmounts the moment `show` flips false, cutting the fade-out short. Decoupled, `mounted` only flips false after a setTimeout that matches the exit duration plus a small grace period."
  - q: "A user says a fade-out 'looks like it drops down before disappearing', even though only opacity is animated. What is most likely happening?"
    a: "Layout-shift perception. When the element unmounts, its margin/padding collapses and siblings shift to fill the space. The eye reads that relative motion as the fading element moving. Fix: keep the element mounted (taking layout space) until opacity reaches 0, or animate the layout collapse synchronously."
---

**Topic:** Reanimated exit animations: decoupling mount from animation state
**Tags:** react-native, reanimated, animations, lifecycle, ui-thread
**Summary:** Why Reanimated's auto exiting animations sometimes get cut short, and the shared-value pattern that takes manual control over the fade-then-unmount lifecycle.

## Mental model

Reanimated's `entering` / `exiting` props look declarative — slap them on an `Animated.View` and trust the library to play them on mount and unmount. But the exit lives in a fragile spot: when a child is conditionally unmounted, Reanimated keeps the *native* view alive on the UI thread for the configured duration even though React has logically removed the component. This works **only if the parent stays stable** during that window. As soon as the parent also unmounts (or there's a complex wrapper chain that confuses Reanimated's tracking), the native view gets torn down early — the fade-out is cut short.

The fix is to stop relying on auto layout-animations and drive the animation manually with shared values, while explicitly **decoupling React-tree presence from animation state**. The component sticks around in the tree for as long as its fade needs; React only removes it after the animation has finished.

## Diagram

```mermaid
sequenceDiagram
    participant H as Parent hook
    participant R as React tree
    participant N as Native view
    participant E as User's eye

    Note over H,E: BROKEN — auto exiting, parent unmounts too
    H->>R: setShow(false)
    R-->>N: unmount Animated.View
    N->>N: start FadeOut (200ms)
    R->>R: layout collapses (margins jump)
    Note right of E: sees nudge "drop"<br/>as siblings jump up
    N-->>R: native view torn down ~119ms (early)

    Note over H,E: FIXED — shared values + decoupled mount
    H->>H: setVisible(false)
    H-->>N: opacity = withTiming(0, 200ms)
    N->>N: fade to 0 over 200ms
    Note right of E: sees pure fade,<br/>no movement
    H->>H: setTimeout 250ms → setMounted(false)
    R-->>R: NOW unmount; layout collapses
    Note right of E: nudge already invisible<br/>so jump is unnoticed
```

## Prerequisites

- React `useEffect` cleanup semantics — the cleanup function runs when React removes the component from the tree
- Reanimated 3 basics: `useSharedValue`, `useAnimatedStyle`, `withTiming`
- Difference between Reanimated's auto layout-animations (`entering` / `exiting` props) and manual shared-value driven animations

## How it actually works

The broken path:

1. Component mounts with `<Animated.View entering={FadeInDown} exiting={FadeOut.duration(200)}>`
2. Some state flips, parent re-renders with the child conditionally absent
3. Reanimated *should* keep the native view alive for 200ms while opacity animates to 0
4. In practice, when the parent also unmounts (e.g. `{show ? <Wrapper><Child /></Wrapper> : null}`), or when there are HOC-like wrappers between Reanimated's tracking and the actual `Animated.View`, the JS-side React unmount fires early
5. The `useEffect` cleanup logs prove it: unmount fires at ~119ms instead of ~200ms+
6. While the native view is mid-fade, React has already removed the component from the layout tree, collapsing any margin/padding it owned and shifting its siblings — the eye reads that as the fading element moving

The fixed path:

1. Drop `entering` / `exiting` from the `Animated.View`
2. Add `useSharedValue` for `opacity` (and any transform you want to animate)
3. In a `useEffect([visible])`, drive shared values explicitly: `opacity.value = withTiming(visible ? 1 : 0, { duration: ... })`
4. Build a `useAnimatedStyle` that consumes those shared values
5. In the parent, separate `visible` (drives animation) from `mounted` (drives whether the `Animated.View` is in the React tree)
6. When `visible` flips true, set `mounted` true immediately
7. When `visible` flips false, *do not* unmount immediately — set a timeout for `EXIT_MS + grace`, then flip `mounted` false
8. The animation now plays through its full duration before React removes the component, so layout collapse happens *after* opacity has reached 0 and there is nothing visible to perceive

Diagnostic technique: when an exit animation looks wrong, log timestamps at:

1. The state change that triggers exit
2. The parent's render with the child absent
3. The `useEffect` cleanup of the animated child (= actual React unmount)

If the unmount timestamp is less than the configured exit duration after the trigger, the auto animation is being cut short. Switch to shared-value control.

## Two examples

**Example 1 — the working pattern:**

```tsx
// Parent hook: decouples visible from mounted
export const useFadingNudge = () => {
  const [visible, setVisible] = useState(false)
  const [mounted, setMounted] = useState(false)

  // Sync mounted with visible, with a delay on the way down so the exit
  // animation has time to complete before React tears the component down.
  useEffect(() => {
    if (visible) {
      setMounted(true)
      return
    }
    if (!mounted) return
    const id = setTimeout(() => setMounted(false), EXIT_MS + 50)
    return () => clearTimeout(id)
  }, [visible, mounted])

  return { visible, mounted, setVisible }
}

// Child component: shared-value driven, no entering/exiting props
export const FadingNudge = ({ visible }: { visible: boolean }) => {
  const opacity = useSharedValue(0)
  const translateY = useSharedValue(-20)

  useEffect(() => {
    if (visible) {
      opacity.value = withTiming(1, { duration: ENTRY_MS })
      translateY.value = withTiming(0, { duration: ENTRY_MS })
    } else {
      // Pure fade — no translate on exit, the element stays in place
      opacity.value = withTiming(0, { duration: EXIT_MS })
    }
  }, [visible, opacity, translateY])

  const style = useAnimatedStyle(() => ({
    opacity: opacity.value,
    transform: [{ translateY: translateY.value }],
  }))

  return <Animated.View style={[styles.nudge, style]}>...</Animated.View>
}

// Caller: render gates on `mounted`, animation prop is `visible`
{mounted ? <FadingNudge visible={visible} /> : null}
```

**Example 2 — the wrong-but-tempting auto-animation approach:**

```tsx
// Looks clean, but exit gets cut short when the parent also unmounts
export const FadingNudge = () => (
  <Animated.View
    entering={FadeInDown.duration(300)}
    exiting={FadeOut.duration(200)}
    style={styles.nudge}
  >
    ...
  </Animated.View>
)

// Caller — parent's conditional removes the whole subtree at once
{show ? (
  <SomeWrapper>
    <FadingNudge />
  </SomeWrapper>
) : null}
```

The auto API is the right starting point — try it first. Only switch to manual control once logs prove the exit is being cut short.

## Why it's written this way

Auto layout-animations are convenient and the right default. They fail in three predictable scenarios:

- **Parent also unmounts at the same time** — Reanimated can keep the *child* alive but not if its parent is being torn down too. Always-render the immediate parent as a stable slot; conditionally render only the leaf
- **Custom wrapper chains break tracking** — DLS-style components, animated parents with their own animated styles (e.g. keyboard-tracking), or HOCs between `Animated.View` and the conditional render point can confuse Reanimated's tracking
- **Exit animation must coordinate with layout** — if a margin or padding on the animated element collapses on unmount, siblings jump. The auto API doesn't give you a hook to delay the layout collapse

Decoupling `visible` from `mounted` is the senior pattern because it makes the fade-then-unmount lifecycle explicit. The animation is driven via timing functions you control, the unmount is gated on a setTimeout you control, and there is no library magic involved that can race with React reconciliation.

The ~50ms grace on the unmount setTimeout is intentional: it covers JS-thread jitter so the native animation's final frame definitely lands before the component disappears.

## Failure modes

- **Auto exit animation cut short** — log unmount timestamps; if they're less than the configured exit duration, switch to shared values
- **Layout-shift perception during fade** — siblings jumping = eye perceives fading element as moving. Keep the element's layout space until opacity reaches 0
- **Spacing on the wrong element** — `marginBottom` on a *parent wrapper* collapses immediately on child unmount; put it on the animated child itself so the spacing leaves with the element
- **Always-render wrapper with explicit empty space** — if the wrapper has explicit padding/margin even when empty, it'll permanently take layout space. Either gate the spacing on `children` presence, or have the spacing live on the child instead
- **Forgetting to update the unmount timeout when changing exit duration** — keep the EXIT duration constant in one place and reference it from both the Reanimated `withTiming` call and the unmount setTimeout

## Quiz

### Q1

A child `Animated.View` has `exiting={FadeOut.duration(200)}`. Logs show React unmounts the component ~119ms after the trigger, not ~200ms. What's the most likely cause?

**Answer:** The parent component is also unmounting (or destabilising) at the same moment, so Reanimated can't keep the native view alive for the full exit duration. Either stabilise the parent (always render the slot, conditionally render only the child) or take manual control via shared values.

### Q2

Why decouple `visible` (animation direction) from `mounted` (React tree presence) in the parent hook?

**Answer:** So the React component stays in the tree for the full exit-animation duration. If they're combined, React unmounts the moment `show` flips false, cutting the fade-out short. Decoupled, `mounted` only flips false after a setTimeout that matches the exit duration plus a small grace period.

### Q3

A user says a fade-out "looks like it drops down before disappearing", even though only opacity is animated. What is most likely happening?

**Answer:** Layout-shift perception. When the element unmounts, its margin/padding collapses and siblings shift to fill the space. The eye reads that relative motion as the fading element moving. Fix: keep the element mounted (taking layout space) until opacity reaches 0, or animate the layout collapse synchronously.
