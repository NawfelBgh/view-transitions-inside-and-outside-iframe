# Testing View Transitions Inside and Outside an Iframe

This repository contains example HTML pages for testing view transitions.

- Open `iframe.html` to test view transitions directly on the page.
- Open `index.html` (which embeds `iframe.html`) to test view transitions inside an iframe.

To test the correctness of browser implementations, the iframe provides an option to render a pending state either before starting a view transition or during a view transition.

## Expected Behavior

- The browser should take a snapshot of all DOM content written before the call to `document.startViewTransition`.
- The browser should paint the DOM content immediately before callback passed to `document.startViewTransition` is called.
- The browser should pause painting new content until the `document.startViewTransition` callback returns.
- The browser should animate from the pre-transition snapshot to the post-transition state.
- The behavior should be the same both inside and outside an iframe.

## Observed Behavior by Browser

- ✅ **Brave works as expected.**
  - Version: 1.98.63 (x86_64), Chromium: 156.0.8078.80

- **Safari (version: 27.2)**
  - When you open `iframe.html`, Safari works as expected. ✅
  - When you open `index.html`, Safari displays the correct DOM content before, during, and after the view transition, but does not animate the transition.

- **Firefox (versions: 159.0 and 160.0b3)**
  - When you open `iframe.html`:
    - If you check "Set pending state BEFORE startViewTransition()":
      - ⚠️ Firefox does not paint the latest version of the DOM (with the pending indicator).
      - ⚠️ It pauses painting during the view transition, so the pending indicator is not displayed.
      - ❌ When the `document.startViewTransition` callback returns, Firefox immediately renders the DOM with the pending indicator and then animates from this suddenly rendered state to the DOM's post-transition state.
    - If you uncheck "Set pending state BEFORE startViewTransition()":
      - ✅ Firefox works as expected.
  - When you open `index.html`:
    - If you check "Set pending state BEFORE startViewTransition()":
      - ✅ Firefox works as expected.
    - If you uncheck "Set pending state BEFORE startViewTransition()":
      - ❌ Firefox renders a pending state that was written to the DOM inside `document.startViewTransition`, then animates from this state to the DOM's post-transition state.

## Takeaways

- ✅ **Brave** implements view transitions correctly and consistently, both inside and outside iframes.
- ✅ **Safari** renders view transitions correctly but does not animate them when running inside an iframe. ⚠️
- ❌ **Firefox** implements view transitions inconsistently inside and outside iframes, with different bugs in each mode:
  - Outside iframes, Firefox does not render the latest DOM content before pausing painting at the start of a view transition. Then, when the transition ends, it suddenly renders the DOM content from before the transition and animates from that state to the page's new state.
  - Inside iframes, Firefox renders DOM changes written during the view transition, even though painting should be paused.
