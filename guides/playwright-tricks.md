# Playwright Tricks for Google Services and More

Hard-won patterns from automating Google Business Profile, Looker Studio, Synology DSM, and Google Sheets via Playwright MCP.

## Google Material Design Comboboxes (Critical)

**Problem:** Google's autocomplete comboboxes (GBP categories, Looker Studio fields, Workspace settings) show an empty dropdown when you use `fill()` or `type()`.

**Root cause:** Google Material Design components listen to `beforeinput` events, not just `input`. Playwright's `type()` and `fill()` only emit `input` events.

**Fix:** Use `keyboard.insertText()`:

```javascript
await combobox.click();                    // focus
await combobox.click({ clickCount: 3 });   // select all
await page.keyboard.press('Backspace');    // clear
await page.waitForTimeout(400);
await page.keyboard.insertText('text');    // generates beforeinput event
await page.waitForTimeout(2000);           // wait for fetch
// Now options appear:
await page.getByRole('option', { name: 'desired option' }).click();
```

**Tested with:** GBP category picker, Looker Studio dimension/metric selectors.

## Looker Studio

### Setting SQL in CodeMirror editors
Looker Studio uses CodeMirror which doesn't respond to normal Playwright input. Use `page.evaluate()`:

```javascript
await page.evaluate((sql) => {
  document.querySelector('.CodeMirror').CodeMirror.setValue(sql);
}, myQuery);
```

### Canvas coordinates
The Looker Studio canvas is 900x1200px with `transform: scale(~0.57)`. Elements are positioned via `top/left` on `.cdk-drag.lego-component-repeat`. Viewport coordinates from `getBoundingClientRect()` map correctly — no manual scaling needed.

### Date filter gotcha
"Dimension Periodo" defaults to "Los ultimos 30 dias (excluir hoy)". You must enable "Incluir hoy" for same-day data.

### Removing chips
Clicking a dimension/metric chip opens the field picker. Press Escape first, then the X button becomes visible and clickable.

## Google Sheets

### Date format for Spanish locale
Use `d/m/yyyy H:mm:ss` (es_ES locale) with `value_input_option="USER_ENTERED"` for Google Sheets to recognize dates natively.

### Service Account sharing order
The Service Account must exist in GCP **before** you share the sheet with it. If the SA doesn't exist yet, Google creates an "external invitation" that doesn't grant API access.

## GCP Console

### Cookie banner intercepts clicks
Always dismiss the "Entendido" cookie banner before interacting with GCP Console elements.

## Synology DSM (NAS admin)

### ExtJS checkboxes
Click the **label text**, not the checkbox element. DSM uses ExtJS which puts an overlay that intercepts direct checkbox clicks.

### ExtJS combobox
Not a native `<select>`. Click to open the dropdown, wait for the list, then click the option.

### Package installation overlays
Installing packages may leave orphan mask overlays that block the entire UI. Fix:
```javascript
const masks = await page.$$('.x-mask');
for (const mask of masks) {
  await mask.evaluate(el => el.style.display = 'none');
}
```

### QuickConnect ID rules
Only letters, numbers, and hyphens (no underscores). Must start with a letter.

## General tips

- **Wait for network idle** before taking screenshots or asserting content — many SPAs have delayed renders.
- **Scroll to bottom and back** before capturing full-page screenshots — triggers lazy loading.
- **Cookie/GDPR banners**: always dismiss first. They sit on top of everything.
- For **visual regression**, capture at exactly 1280x800 (desktop) and 375x812 (mobile) for consistent baselines.
