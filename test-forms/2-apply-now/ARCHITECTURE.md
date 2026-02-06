# Apply Now → Form Architecture

## Overview
This form pattern simulates job sites where clicking "Apply Now" navigates to a separate application form page.

## Purpose
Test the ghostwriter agent's ability to:
- Navigate from a job listing to an application form
- Detect and click the "Apply Now" button/link
- Handle the two-page flow (listing → form → confirmation)

## Structure

```
2-apply-now/
├── index.html        # Job listing with "Apply Now" button
├── form.html         # The actual application form
├── confirmation.html # Success page after submission
└── ARCHITECTURE.md   # This file
```

## User Flow

```
index.html (Job Listing)
    │
    │ Click "Apply Now"
    ▼
form.html (Application Form)
    │
    │ Submit
    ▼
confirmation.html (Success)
```

## Page Details

### index.html - Job Listing
- Company: TechCorp
- Position: Data Analyst
- Contains: role description, responsibilities, requirements, benefits
- CTA: "Apply Now" link to form.html

### form.html - Application Form

| Field | Type | Required | Notes |
|-------|------|----------|-------|
| First Name | text | Yes | |
| Last Name | text | Yes | |
| Email | email | Yes | |
| Phone | tel | Yes | |
| Current Location | text | Yes | City, State format |
| Resume | file | Yes | .pdf/.doc/.docx |
| SQL Experience | select | Yes | Dropdown with ranges |
| Python Experience | select | Yes | Dropdown with ranges |
| Visualization Tools | select (multiple) | No | Multi-select list |
| Why Interested | textarea | No | |

### confirmation.html
- Shows success checkmark
- Confirms application received
- Link back to test forms index

## Technical Details

### Navigation Pattern
```html
<!-- In index.html -->
<a href="form.html" class="btn">Apply Now</a>
```
- Simple anchor tag navigation
- Styled as a button with `.btn` class
- No JavaScript required for navigation

### Multi-Select Field
```html
<select id="visualization_tools" name="visualization_tools" multiple size="4">
  <option value="tableau">Tableau</option>
  ...
</select>
```
- Tests agent's ability to handle multi-select inputs
- `size="4"` shows 4 options at once

### Form Submission
- Standard form with `onsubmit` handler
- Prevents default and redirects to confirmation

## Agent Testing Scenarios

1. **Navigation Detection**: Agent must find and click "Apply Now" link
2. **Page Transition**: Agent should wait for form.html to load
3. **Mixed Field Types**: Combination of text, select, multi-select, file
4. **Two-Step Process**: Agent must handle landing page → form flow

## Key Differences from Single-Page

| Aspect | Single-Page | Apply Now |
|--------|-------------|-----------|
| Pages | 1 (+ confirmation) | 2 (+ confirmation) |
| Navigation | None | Required |
| Entry Point | Direct to form | Job listing first |
| CTA | Submit button | Apply Now → Submit |

## Limitations

- No back navigation from form to listing (could add)
- Multi-select has no "select all" option
- No application preview before submit
