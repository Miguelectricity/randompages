# Multi-Page Wizard Architecture

## Overview
A 5-step application wizard that requires navigating between pages. This pattern is common on enterprise job sites and applicant tracking systems.

## Purpose
Test the ghostwriter agent's ability to:
- Navigate a multi-step form wizard
- Maintain context across multiple pages
- Handle forward and backward navigation
- Complete all steps before final submission

## Structure

```
3-multi-page/
├── step1.html        # Personal Information
├── step2.html        # Work Experience
├── step3.html        # Education
├── step4.html        # Professional References
├── step5.html        # Review & Submit
├── confirmation.html # Success page
└── ARCHITECTURE.md   # This file
```

## User Flow

```
step1.html (Personal)
    │
    │ Next →
    ▼
step2.html (Experience)
    │
    │ ← Back / Next →
    ▼
step3.html (Education)
    │
    │ ← Back / Next →
    ▼
step4.html (References)
    │
    │ ← Back / Next →
    ▼
step5.html (Review)
    │
    │ ← Back / Submit
    ▼
confirmation.html (Success)
```

## Step Details

### Step 1: Personal Information (8 fields)
| Field | Type | Required |
|-------|------|----------|
| First Name | text | Yes |
| Last Name | text | Yes |
| Email Address | email | Yes |
| Phone Number | tel | Yes |
| Street Address | text | Yes |
| City | text | Yes |
| State | select | Yes |
| ZIP Code | text | Yes (pattern: 5 digits) |

### Step 2: Work Experience (6 fields)
| Field | Type | Required |
|-------|------|----------|
| Current Job Title | text | Yes |
| Current Company | text | Yes |
| Total Years Experience | select | Yes |
| PM Experience | select | Yes |
| Resume | file | Yes |
| Key Achievements | textarea | No |

### Step 3: Education (5 fields)
| Field | Type | Required |
|-------|------|----------|
| Highest Degree | select | Yes |
| Field of Study | text | Yes |
| School/University | text | Yes |
| Graduation Year | select | Yes |
| Certifications | textarea | No |

### Step 4: References (12 fields)
Two references, each with:
| Field | Type | Required |
|-------|------|----------|
| Full Name | text | Yes |
| Job Title | text | Yes |
| Company | text | Yes |
| Email | email | Yes |
| Phone | tel | Yes |
| Relationship | select | Yes |

### Step 5: Review & Submit (4 fields)
| Field | Type | Required |
|-------|------|----------|
| How Heard | select | Yes |
| Additional Info | textarea | No |
| Terms Checkbox | checkbox | Yes |
| Privacy Checkbox | checkbox | Yes |

**Total Fields: 35**

## Technical Details

### Step Indicator
```html
<ul class="step-indicator">
  <li class="completed">1. Personal</li>
  <li class="active">2. Experience</li>
  <li>3. Education</li>
  ...
</ul>
```
- Visual progress indicator at top of each page
- `.completed` class for finished steps (green)
- `.active` class for current step (blue)
- Default state for future steps (gray)

### Navigation Pattern
```html
<div class="nav-buttons">
  <a href="step1.html" class="btn btn-secondary">← Back</a>
  <button type="submit">Next →</button>
</div>
```
- Back: Simple anchor link (no form submission)
- Next: Form submit that validates and navigates
- First step has no Back button
- Last step has "Submit Application" instead of Next

### Form Submission
Each step validates its own fields:
```javascript
function handleNext(e) {
  e.preventDefault();
  // HTML5 validation runs before this
  window.location.href = 'step2.html';
  return false;
}
```

### No State Persistence
- Data is NOT persisted between steps
- Each page loads fresh
- In production, you'd use localStorage, sessionStorage, or server-side sessions
- For testing purposes, agent just needs to fill and navigate

## Agent Testing Scenarios

1. **Sequential Navigation**: Agent must complete steps in order
2. **Field Counting**: 35 total fields across 5 pages
3. **Checkbox Handling**: Required checkboxes on final step
4. **Back Navigation**: Agent may need to go back and edit
5. **Step Detection**: Agent should recognize which step it's on

## Key Challenges for Agent

| Challenge | Description |
|-----------|-------------|
| Page Detection | Know which step is current |
| Completion Detection | Recognize when all steps done |
| Form Persistence | Handle that data doesn't persist |
| Navigation | Find and click Next/Back buttons |
| Required Checkboxes | Must check both boxes on step 5 |

## Limitations

- No actual state persistence between steps
- No ability to jump to arbitrary steps
- No save draft functionality
- Back navigation loses form data
