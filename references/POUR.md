# POUR Audit Checklist

Comprehensive accessibility guidelines based on WCAG 2.2 and Lighthouse accessibility audits. Goal: make content usable by everyone, including people with disabilities.

## WCAG Principles

| Principle | Description |

| ------------------ | ------------------------------------------------- |

| **P**erceivable | Content can be perceived through different senses |

| **O**perable | Interface can be operated by all users |

| **U**nderstandable | Content and interface are understandable |

| **R**obust | Content works with assistive technologies |

## Conformance levels

| Level | Requirement | Target |

| ------- | ---------------------- | ----------------------------------------------------- |

| **A** | Minimum accessibility | Must pass |

| **AA** | Standard compliance | Should pass (legal requirement in many jurisdictions) |

| **AAA** | Enhanced accessibility | Nice to have |

---

## Perceivable

### Text alternatives (1.1)

**Images require alt text:**

```jsx
{/* ❌ Missing alt */}

<img src="chart.png" />

{/* ✅ Descriptive alt */}

<img
  src="chart.png"
  alt="Bar chart showing 40% increase in Q3 sales"
/>

{/* ✅ Decorative image (empty alt) */}

<img src="decorative-border.png" alt="" role="presentation" />

{/* ✅ Complex image with longer description */}

<figure>
  <img
    src="infographic.png"
    alt="2024 market trends infographic"
    aria-describedby="infographic-desc"
  />

  <figcaption id="infographic-desc">
    {/* Detailed description */}
  </figcaption>
</figure>
```

**Icon buttons need accessible names:**

```jsx
{/* ❌ No accessible name */}

<button>
  <svg>{/* menu icon */}</svg>
</button>

{/* ✅ Using aria-label */}

<button aria-label="Open menu">
  <svg aria-hidden="true" focusable="false">{/* menu icon */}</svg>
</button>

{/* ✅ Using visually hidden text */}

<button>
  <svg aria-hidden="true" focusable="false">{/* menu icon */}</svg>

  <span className="visually-hidden">Open menu</span>
</button>
```

**SVG elements:**

| Use case | Required attributes |

|----------|---------------------|

| Decorative (icon inside button/link) | `aria-hidden="true" focusable="false"` |

| Informative (standalone illustration, chart) | `role="img" aria-labelledby="[id]"` + `<title id="[id]">` as first child |

| Interactive (clickable SVG) | Wrap in `<button>` or `<a>` — don't make the SVG itself interactive |

```jsx
{/* ✅ Informative SVG */}

<svg role="img" aria-labelledby="chart-title">
  <title id="chart-title">Q3 revenue by region</title>

  {/* ... */}
</svg>

{/* ✅ Decorative SVG */}

<svg aria-hidden="true" focusable="false">{/* ... */}</svg>
```

**SVGR imports:** When an `.svg` file is imported as a React component (`import Logo from './logo.svg'`), audit its rendered output for the above attributes — SVGR-generated components often strip accessibility attributes entirely.

**Visually hidden class:**

```css
.visually-hidden {
  position: absolute;

  width: 1px;

  height: 1px;

  padding: 0;

  margin: -1px;

  overflow: hidden;

  clip: rect(0, 0, 0, 0);

  white-space: nowrap;

  border: 0;
}
```

### Color contrast (1.4.3, 1.4.6)

> **Static analysis limitation:** Contrast ratios can only be evaluated when color values are directly readable in source (hex, RGB, or HSL literals). When colors are defined via Tailwind classes, CSS custom properties (`var(--x)`), styled-components theme tokens, or design system token files, static analysis cannot compute the ratio. In these cases, note the limitation in your finding and recommend verifying with the Level Access scan or the Colour Contrast Analyser tool.

| Text Size | AA minimum | AAA enhanced |

| ------------------------------------------- | ---------- | ------------ |

| Normal text (< 24px / < 18.66px [14pt] bold) | 4.5:1 | 7:1 |

| Large text (≥ 24px / ≥ 18.66px [14pt] bold) | 3:1 | 4.5:1 |

| UI components & graphics | 3:1 | 3:1 |

```css
/* ❌ Low contrast (~2.85:1) */

.low-contrast {
  color: #999;

  background: #fff;
}

/* ✅ Sufficient contrast (~12.6:1) */

.high-contrast {
  color: #333;

  background: #fff;
}

/* ✅ Focus states need contrast too */

:focus-visible {
  outline: 2px solid #005fcc;

  outline-offset: 2px;
}
```

**Don't rely on color alone:**

```jsx
{/* ❌ Only color indicates error (border-color: red in the stylesheet) */}

<input className="error-border" />

{/* ✅ Color + icon + text */}

<div className="field-error">
  <input aria-invalid="true" aria-describedby="email-error" />

  <span id="email-error" className="error-message">
    <svg aria-hidden="true">{/* error icon */}</svg>

    Please enter a valid email address
  </span>
</div>
```

### Media alternatives (1.2)

```jsx
{/* Video with captions */}

<video controls>
  <source src="video.mp4" type="video/mp4" />

  <track
    kind="captions"
    src="captions.vtt"
    srcLang="en"
    label="English"
    default
  />

  <track
    kind="descriptions"
    src="descriptions.vtt"
    srcLang="en"
    label="Descriptions"
  />
</video>

{/* Audio with transcript */}

<audio controls>
  <source src="podcast.mp3" type="audio/mp3" />
</audio>

<details>
  <summary>Transcript</summary>

  <p>Full transcript text...</p>
</details>
```

---

### Resize Text (1.4.4)

**Text can be resized to 200% without loss of functionality**

#### Use rems for font-sizing and

```css
/* ❌ Never hard code font sizes */

p {
  font-size: 16px;
}

/* ✅ Use rem for font related sizing */

p {
  font-size: 1rem;
}
```

---

## Operable

### Keyboard accessible (2.1)

**All functionality must be keyboard accessible:**

```jsx
{/* ❌ Only handles click */}

<div onClick={handleAction}>Submit</div>

{/* ✅ Use a native, keyboard-accessible element instead of reimplementing key handling */}

<button onClick={handleAction}>Submit</button>

{/* ✅ If a non-native element must be interactive, wire up Enter/Space explicitly */}

<div
  role="button"
  tabIndex={0}
  onClick={handleAction}
  onKeyDown={(e) => {
    if (e.key === "Enter" || e.key === " ") {
      e.preventDefault();
      handleAction();
    }
  }}
>
  Submit
</div>
```

**No keyboard traps.** Users must be able to Tab into and out of every component. Use the [modal focus trap pattern](A11Y.md#modal-focus-trap) for dialogs — the native `<dialog>` element handles this automatically.

### Focus visible (2.4.7)

```css
/* ❌ Never remove focus outlines */

*:focus {
  outline: none;
}

/* ✅ Use :focus-visible for keyboard-only focus */

:focus {
  outline: none;
}

:focus-visible {
  outline: 2px solid #005fcc;

  outline-offset: 2px;
}

/* ✅ Or custom focus styles */

button:focus-visible {
  box-shadow: 0 0 0 3px rgba(0, 95, 204, 0.5);
}
```

**Caveat:** `outline: none` on `:focus` relies on `:focus-visible` being supported. In an engine without it (or if `:focus-visible` is later removed/overridden), this silently removes all focus indication — a 2.4.7 failure. Flag `:focus { outline: none }` without a same-selector-or-broader `:focus-visible` fallback in the same stylesheet.

### Focus not obscured (2.4.11) — new in 2.2

**What 2.4.11 (Minimum, AA) actually requires:** the focused element is not *entirely* hidden by author-created content (partial obscuring is allowed). At Level AAA (2.4.12), no part of the focused element may be hidden.

```css
/* ⚠️ Partial aid only — see caveat below */

:target {
  scroll-margin-top: 80px;
}

:focus {
  scroll-margin-top: 80px;

  scroll-margin-bottom: 60px;
}
```

`scroll-margin`/`scroll-padding` only help when Tab moves focus to an element that starts **off-screen** and the browser scrolls it into view. They do **nothing** when the focused element is already in the viewport under a sticky/fixed overlay. For that case, size `scroll-padding-top`/`scroll-padding-bottom` on the scroll container to the sticky region (WCAG technique C43), and avoid overlaying focused content in the first place:

```css
/* ✅ Reserve space on the scroll container for a sticky header/footer */

html {
  scroll-padding-top: 80px;

  scroll-padding-bottom: 60px;
}
```

### Skip links (2.4.1)

Provide a skip link so keyboard users can bypass repetitive navigation. See [A11Y.md#skip-link](A11Y.md#skip-link) for full markup and styles.

### Target size (2.5.8) — new in 2.2

Interactive targets must be at least **24 × 24 CSS pixels** (AA). Five exceptions:

- **Spacing:** an undersized target is fine if a 24px-diameter circle centered on it doesn't overlap the same circle centered on any adjacent target.
- **Equivalent:** another control on the same page achieves the same function at the full 24×24 size.
- **Inline:** the target sits within a sentence or block of text (e.g., a link in a paragraph).
- **User agent control:** the target's size is determined by the browser/OS and isn't modified by the author.
- **Essential:** a specific presentation of the target is essential, or legally required, to convey the information.

```css
/* ✅ Minimum target size */

button,
[role="button"],
input[type="checkbox"] + label,
input[type="radio"] + label {
  min-width: 24px;

  min-height: 24px;
}

/* ✅ Comfortable target size (recommended 44×44) */

.touch-target {
  min-width: 44px;

  min-height: 44px;

  display: inline-flex;

  align-items: center;

  justify-content: center;
}
```

### Dragging movements (2.5.7) — new in 2.2

Any action that requires dragging must have a single-pointer alternative (e.g., buttons, inputs). See [A11Y.md#dragging-movements](A11Y.md#dragging-movements) for a sortable-list example.

### Timing (2.2)

```javascript
// Allow users to extend time limits

function showSessionWarning() {
  const modal = createModal({
    title: "Session Expiring",

    content: "Your session will expire in 2 minutes.",

    actions: [
      { label: "Extend session", action: extendSession },

      { label: "Log out", action: logout },
    ],

    timeout: 120000,
  });
}
```

### Motion (2.3)

```css
/* Respect reduced motion preference */

@media (prefers-reduced-motion: reduce) {
  *,
  *::before,
  *::after {
    animation-duration: 0.01ms !important;

    animation-iteration-count: 1 !important;

    transition-duration: 0.01ms !important;

    scroll-behavior: auto !important;
  }
}
```

---

## Understandable

### Page language (3.1.1)

```html
<!-- ❌ No language specified -->

<html>
</html>

<!-- ✅ Language specified -->

<html lang="en">
</html>

<!-- ✅ Language changes within page -->

<p>
  The French word for hello is <span lang="fr">bonjour</span>.
</p>
```

In a React app, `lang` is usually set once on the root `<html>` element (e.g., in a Next.js root `layout.tsx` or the static `index.html` `<html>` tag) rather than per-component — check that root element, not individual JSX components.

### Consistent navigation (3.2.3)

Navigation menus that repeat across pages must appear in the same relative order each time. See [A11Y.md#navigation](A11Y.md#navigation) for the markup pattern.

### Consistent help (3.2.6) — new in 2.2

If a help mechanism (contact info, chat widget, FAQ link) is repeated across pages, it must appear in the **same relative order** each time.

### Form labels (3.3.2)

Every input needs a programmatically associated label. See [A11Y.md#form-labels](A11Y.md#form-labels) for explicit, implicit, and instructional examples.

### Error handling (3.3.1, 3.3.3)

Announce errors with `role="alert"` or `aria-live`, set `aria-invalid="true"` on invalid fields, and focus the first error on submit. See [A11Y.md#error-handling](A11Y.md#error-handling) for full markup and JS.

### Redundant entry (3.3.7) — new in 2.2

Don't force users to re-enter information already provided in the same session. Auto-populate from earlier steps or let users select from previously entered values.

```jsx
{/* ✅ Auto-fill shipping address from billing */}

<fieldset>
  <legend>Shipping address</legend>

  <label>
    <input
      type="checkbox"
      id="same-as-billing"
      checked={sameAsBilling}
      onChange={(e) => setSameAsBilling(e.target.checked)}
    />

    Same as billing address
  </label>
</fieldset>
```

### Accessible authentication (3.3.8) — new in 2.2

Login flows must not rely solely on cognitive tests (memorizing passwords, solving puzzles) unless copy-paste/autofill is available or an alternative method exists (passkey, SSO, email link).

```jsx
{/* ✅ Allow paste in password fields */}

<input
  type="password"
  id="password"
  autoComplete="current-password"
/>

{/* ✅ Offer passwordless alternatives */}

<button type="button">Sign in with passkey</button>

<button type="button">Email me a login link</button>
```

---

## Robust

### ARIA usage (4.1.2)

**Prefer native elements:**

```jsx
{/* ❌ ARIA role on div */}

<div role="button" tabIndex={0}>Click me</div>

{/* ✅ Native button */}

<button>Click me</button>

{/* ❌ ARIA checkbox */}

<div role="checkbox" aria-checked="false">Option</div>

{/* ✅ Native checkbox */}

<label><input type="checkbox" /> Option</label>
```

When ARIA is needed, use correct roles and states. See [A11Y.md#aria-tabs](A11Y.md#aria-tabs) for a complete tablist example.

### Live regions (4.1.3)

Use `aria-live` regions to announce dynamic content changes without moving focus. See [A11Y.md#live-regions-and-notifications](A11Y.md#live-regions-and-notifications) for markup and a `showNotification()` helper.

### React-specific checks

_(Also check `.js` files in pre-TypeScript projects — these commonly contain JSX without the `.jsx` extension.)_

- Missing `key` props on lists — a React reconciliation correctness issue; flag as a code-quality note, not a WCAG finding

- Refs for focus management without null checks

- `createPortal` renders content outside the parent DOM subtree — this is the standard pattern for modals/tooltips and does not itself break ARIA relationships. Verify that `aria-owns`/`aria-controls`/`aria-labelledby` IDs referenced by portaled content still resolve to elements present in the DOM

- React Router `<Link>` missing accessible names

- Fragments (`<></>`) wrapping content that needs a landmark container

- **Conditionally rendered feedback** (errors, toasts, loading states, validation messages): if a component can appear or disappear based on state, check that its content is announced via `aria-live` or `role="alert"`. See [A11Y.md#live-regions-and-notifications](A11Y.md#live-regions-and-notifications) for implementation.

**Dynamic ARIA states — common React bugs:**

**`aria-expanded` must be bound to state, not hardcoded:**

```jsx

// ❌ Never reflects open/closed state

<button aria-expanded="false" onClick={toggle}>Menu</button>



// ✅ Reflects current state

<button aria-expanded={isOpen} onClick={toggle}>Menu</button>

```

**`aria-hidden="true"` must not contain focusable children:**

```jsx

// ❌ AT users can't perceive these buttons; keyboard users still can — ghost tab stops

<div aria-hidden="true">

<button>Action</button>

</div>



// ✅ Option 1: remove aria-hidden if content should be accessible

<div>

<button>Action</button>

</div>



// ✅ Option 2: if truly decorative, don't render interactive children at all —
// tabIndex={-1} on the wrapper does NOT remove focusability from descendant
// buttons/links/inputs, it only affects the wrapper div itself

<div aria-hidden="true">

{/* no interactive children */}

</div>

```

**`aria-controls` must reference an ID that exists in the DOM:**

```jsx

// ❌ #panel doesn't exist until isOpen is true — aria-controls points to nothing

<button aria-controls="panel" aria-expanded={isOpen}>Toggle</button>

{isOpen && <div id="panel">...</div>}



// ✅ Always render the element; use hidden to collapse it visually

<button aria-controls="panel" aria-expanded={isOpen}>Toggle</button>

<div id="panel" hidden={!isOpen}>...</div>

```

**`aria-describedby` / `aria-labelledby` ID mismatches:** Verify the referenced ID actually exists in the rendered output — copy-paste and refactoring frequently leave stale references that silently break AT associations.

### Component library awareness

These libraries handle many a11y concerns internally — check how they're _used_ (props, composition), not their internals:

- **MUI (Material UI)**: Built-in ARIA for most components

- **Radix UI**: Fully accessible primitives

- **Headless UI**: Accessible by design

- **React Aria / React Spectrum**: Adobe's a11y-first library

---
