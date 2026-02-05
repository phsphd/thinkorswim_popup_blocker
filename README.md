# Click-to-Remove Slide Popup (DevTools Console Script)

A lightweight JavaScript snippet you can paste into the Chrome DevTools console to remove annoying right-side slide-in panels (newsletter drawers, chat widgets, cookie/consent drawers, etc.) on the current page.

Instead of trying to guess selectors, this script lets you **click the popup** and removes the closest “overlay-ish” container.

> ⚠️ This is a **temporary, per-tab** tool. It does not permanently change Chrome settings or the website. Refreshing the page will bring the popup back.

---

## What it does

- Adds a small on-screen banner: **“Click the popup to remove it (ESC to stop)”**
- Intercepts the next click you make
- Walks up the DOM tree to find a good parent container that looks like an overlay/drawer
- Removes that element from the page
- Attempts to undo “scroll lock” (common with overlays)
- You can press **ESC** to stop the click-removal mode

---

## Usage

1. Open the page where the slide-in popup appears.
2. Open DevTools:
   - Windows/Linux: `Ctrl` + `Shift` + `I`
   - Mac: `Cmd` + `Option` + `I`
3. Go to the **Console** tab.
4. Paste the script below and press **Enter**.
5. Click the popup/drawer panel you want removed.
6. Press **ESC** to stop the remover mode.

---

## Script

```js
(() => {
  const badge = document.createElement("div");
  badge.textContent = "Click the popup to remove it (ESC to stop)";
  Object.assign(badge.style, {
    position: "fixed",
    top: "10px",
    left: "10px",
    zIndex: 2147483647,
    padding: "8px 10px",
    background: "rgba(180,0,0,0.9)",
    color: "white",
    font: "12px/1.2 sans-serif",
    borderRadius: "6px",
  });
  document.documentElement.appendChild(badge);

  const isOverlayish = (el) => {
    if (!(el instanceof HTMLElement)) return false;
    const s = getComputedStyle(el);
    const z = parseInt(s.zIndex || "0", 10);
    const pos = s.position;
    const r = el.getBoundingClientRect();
    const big = r.width > innerWidth * 0.2 || r.height > innerHeight * 0.2;
    return (pos === "fixed" || pos === "sticky") && (z >= 10) && big;
  };

  const handler = (e) => {
    e.preventDefault();
    e.stopPropagation();

    let el = e.target;
    let best = el;

    // Walk up to find a good container to remove
    for (let i = 0; i < 20 && el; i++) {
      if (isOverlayish(el)) best = el;
      el = el.parentElement;
    }

    console.log("[remove]", best);
    best.remove();

    // unlock scroll if it was locked
    document.documentElement.style.overflow = "auto";
    document.body.style.overflow = "auto";
  };

  const stop = () => {
    window.removeEventListener("click", handler, true);
    window.removeEventListener("keydown", onKey, true);
    badge.remove();
    console.log("[popup-remover] stopped");
  };

  const onKey = (e) => {
    if (e.key === "Escape") stop();
  };

  window.addEventListener("click", handler, true);
  window.addEventListener("keydown", onKey, true);

  console.log("[popup-remover] active: click the popup (ESC to stop)");
})();
