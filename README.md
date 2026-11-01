# Testing View Transitions inside and outside an iframe

This repo contains example HTML pages to test view transitions.

- Open `iframe.html` to test view transition directly from the page.
- Open `index.html` (which embeds `iframe.html`) to test view transition inside an iframe

To test browsers' implementation correctness, the iframe gives you the option to render a pending state either before starting view transition or during view transition.

## Expected behavior

- The browser should take a snapshot with all DOM content written until the call to `document.startViewTransition`
- The browser should paint the DOM content just before calling `document.startViewTransition`
- The browser should pause painting new content until the callback to `document.startViewTransition` returns
- The browser should animate from the pre view transition snapshot to the post view transition state
- The behavior should be the same inside and outside an iframe.

## Observed behavior by browser

- ✅ Brave works as expected
  - Version: 1.98.63 (x86_64) Chromium: 156.0.8078.80
- Safari (version: 27.2)
  - When you open `iframe.html`, Safari works as expected ✅.
  - When you open `index.html`, Safari shows correct DOM content, before, during and after view transition but doesn't animate the view transition .
- Firefox (versions: 159.0 and 160.0b3)
  - When you open `iframe.html`:
    - If you check "Set pending state BEFORE startViewTransition()"
      - ⚠️ Firefox does not paint the last version of the DOM (with pending indicator)
      - ⚠️ It pauses painting during the the view transition, showing no pending indicator
      - ❌ When `document.startViewTransition` callback returns, Firefox immediately renders the DOM with the pending indicator, and then it animates from this suddenly-rendered state into the state of the DOM after the view transition.
    - If you uncheck "Set pending state BEFORE startViewTransition()"
      - ✅ Firefox works as expected
  - When you open `iframe.html`:
    - If you check "Set pending state BEFORE startViewTransition()"
      - ✅ Firefox works as expected
    - If you uncheck "Set pending state BEFORE startViewTransition()"
      - ❌ Firefox renders a pending state which was written to the DOM inside `document.startViewTransition`, and then animates a view transition from this state to the new state of the DOM after the view transition.

## Takeaways

- Brave implement view transitions correctly consistently inside and outside iframes.
- Safari disables renders view transitions correctly but doesn't animate them when inside an iframe.
- Firefox implement view transitions inconsistently inside and outside iframes with different bugs in each mode:
  - Outside iframes, Firefox doesn't render the latest DOM content before pausing painting at the start of a view transition. Then, at the end of the view transition it renders the DOM content from before the transition suddenly and animate from that state to the new state of the page
  - Inside iframes, Firefox renders DOM changes written during the view transition when it is supposed to have paused painting.
