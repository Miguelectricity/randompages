# Greenhouse React Form Architecture

## Overview

This form uses **actual React components** to replicate Greenhouse ATS patterns. Unlike the vanilla JS version (form #8), this uses React 18 with hooks, state management, and `ReactDOM.createPortal` for dropdown menus—exactly like real Greenhouse forms.

## Purpose

Test the ghostwriter agent's ability to handle:
- **React-rendered components** (not in initial HTML)
- **ReactDOM.createPortal** for dropdown menus
- **React state** managing option visibility
- **useEffect async loading** of options
- **Virtual DOM** (options exist in React state, not DOM until opened)

## Source Reference

- Based on real Greenhouse form patterns
- Jira ticket: DROID-10752
- Vanilla JS version: `8-greenhouse/`

## Structure

```
9-greenhouse-react/
├── index.html        # React application (no build step needed)
├── confirmation.html # Success page
└── ARCHITECTURE.md   # This file
```

## Why React Makes Extraction Harder

### The Additional Challenge

Beyond the vanilla JS challenges, React adds:

| Challenge | Why It's Harder |
|-----------|-----------------|
| **Virtual DOM** | Options exist in React state, not DOM until rendered |
| **Component lifecycle** | Options only load when `useEffect` fires on open |
| **createPortal** | Menu rendered to a different DOM tree entirely |
| **State-driven rendering** | No options in DOM until `isOpen === true` |
| **Hydration timing** | Initial HTML has no form content |

### How Options Are Managed

```jsx
// Options start empty - NOT in DOM
const [options, setOptions] = useState([]);
const [isOpen, setIsOpen] = useState(false);

// Only fetched when dropdown opens
useEffect(() => {
  if (isOpen && options.length === 0) {
    fetchOptions(name).then(setOptions);
  }
}, [isOpen]);

// Rendered via portal when open
{isOpen && createPortal(
  <div className="css-26l3qy-menu">
    {options.map(opt => (
      <div data-value={opt.value}>{opt.label}</div>
    ))}
  </div>,
  document.getElementById('select-portal-root')
)}
```

### DOM States

**Initial page load:**
```html
<div id="application-root">
  <!-- React hasn't rendered yet -->
</div>
<div id="select-portal-root">
  <!-- Empty -->
</div>
```

**After React hydration:**
```html
<div id="application-root">
  <div class="application-panel">
    <div class="css-2b097c-container">
      <input type="hidden" name="age_18" value="">
      <div class="css-yk16xz-control">
        <span class="css-1wa3eu0-placeholder">Select...</span>
      </div>
      <!-- NO MENU - not open yet -->
    </div>
  </div>
</div>
```

**After clicking dropdown:**
```html
<div id="select-portal-root">
  <!-- Menu appears HERE via createPortal -->
  <div class="css-26l3qy-menu">
    <div class="css-4ljt47-MenuList">
      <div class="css-1n7v3ny-option" data-value="yes">Yes</div>
      <div class="css-1n7v3ny-option" data-value="no">No</div>
    </div>
  </div>
</div>
```

## Technical Implementation

### React Setup (No Build Step)

Uses CDN-loaded React with Babel for JSX transformation:

```html
<script crossorigin src="https://unpkg.com/react@18/umd/react.development.js"></script>
<script crossorigin src="https://unpkg.com/react-dom@18/umd/react-dom.development.js"></script>
<script src="https://unpkg.com/@babel/standalone/babel.min.js"></script>

<script type="text/babel">
  // JSX code here, transformed at runtime
</script>
```

### GHSelect Component

```jsx
function GHSelect({ name, placeholder, required, onChange, error }) {
  const [isOpen, setIsOpen] = useState(false);
  const [options, setOptions] = useState([]);
  const [isLoading, setIsLoading] = useState(false);
  const [selectedOption, setSelectedOption] = useState(null);

  // Async load options on open
  useEffect(() => {
    if (isOpen && options.length === 0) {
      setIsLoading(true);
      fetchOptions(name).then((data) => {
        setOptions(data);
        setIsLoading(false);
      });
    }
  }, [isOpen]);

  // Render menu in portal
  const menuPortal = isOpen ? createPortal(
    <div className="css-26l3qy-menu" style={menuPosition}>
      <div className="css-4ljt47-MenuList">
        {isLoading ? (
          <div>Loading...</div>
        ) : (
          options.map(opt => (
            <div
              className="css-1n7v3ny-option"
              data-value={opt.value}
              onClick={() => handleSelect(opt)}
            >
              {opt.label}
            </div>
          ))
        )}
      </div>
    </div>,
    document.getElementById('select-portal-root')
  ) : null;

  return (
    <div className="css-2b097c-container">
      <input type="hidden" name={name} value={selectedOption?.value || ''} />
      <div className="css-yk16xz-control" onClick={() => setIsOpen(!isOpen)}>
        {selectedOption ? selectedOption.label : placeholder}
      </div>
      {menuPortal}
    </div>
  );
}
```

### Simulated API

```javascript
const fetchOptions = (fieldName) => {
  return new Promise((resolve) => {
    setTimeout(() => {
      resolve(DROPDOWN_OPTIONS_API[fieldName] || []);
    }, 200); // 200ms network delay
  });
};
```

## Extraction Strategy

To extract options from this React form, the extractor needs to:

### 1. Detect React Portals

```javascript
// Check for portal root containers
const portalRoot = document.getElementById('select-portal-root');
const menuPortal = document.querySelector('[class*="-menu"]');
```

### 2. Wait for React Rendering

```javascript
// After clicking, React needs time to:
// 1. Update state (isOpen = true)
// 2. Trigger useEffect
// 3. Await async fetchOptions
// 4. Re-render with options
// Total: ~250-300ms minimum
```

### 3. Handle Multiple Portals

When multiple dropdowns exist, only one should be open at a time. The extractor should:
- Click dropdown to open
- Wait for portal to appear
- Extract options from portal
- Click elsewhere to close (triggering React state change)
- Repeat for next dropdown

## Form Fields

### React-Controlled Dropdowns

| Field | Options | Async Delay |
|-------|---------|-------------|
| Phone Country | 160+ countries | 200ms |
| Age 18+ | Yes, No | 200ms |
| Previous Employee | Yes, No | 200ms |
| Work Authorization | Yes, No | 200ms |
| Sponsorship | Yes, No | 200ms |
| Essential Functions | Yes, No | 200ms |

### Standard Inputs

| Field | Type | Required |
|-------|------|----------|
| First/Last Name | text | Yes |
| Email | email | Yes |
| Phone | tel | Yes |
| LinkedIn/Website | url | No |
| Desired Salary | text | Yes |
| Text areas | textarea | No |

## Comparison: React vs Vanilla JS

| Aspect | Vanilla JS (Form 8) | React (Form 9) |
|--------|---------------------|----------------|
| Initial DOM | Has trigger elements | Empty root div |
| Options storage | JS variable | React state |
| Menu rendering | DOM manipulation | createPortal |
| State changes | Direct DOM updates | Virtual DOM reconciliation |
| Timing | 150ms delay | 200ms + React render cycle |
| Cleanup | Manual DOM removal | Automatic on state change |
| Phone country options | 160+ | 160+ |

## Testing Scenarios

1. **React hydration wait**: Form content appears after React mounts
2. **Portal detection**: Find `#select-portal-root` after click
3. **Async state**: Options load via useEffect, not immediately
4. **State-driven visibility**: Menu only in DOM when `isOpen === true`
5. **Multiple dropdowns**: Each opens to same portal location
6. **Large dropdown**: Phone country has 160+ options

## Success Criteria

The `dropdown_extractor.py` improvements should:

1. Wait for React hydration (form content to appear)
2. Click dropdown trigger in React component
3. Wait for portal to populate (state update + async load)
4. Find options in `#select-portal-root`
5. Extract `data-value` attributes
6. Handle 200ms async loading + React render time
7. Work with all 6 dropdown fields
8. Handle 160+ options in phone country dropdown

## Files to Modify

Same as Form 8, plus:

1. Consider React-specific timing (longer waits)
2. Handle empty initial DOM state
3. Detect portal root containers by ID pattern
