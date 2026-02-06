# Single-Page Form Architecture

## Overview
This form represents the simplest job application pattern: all fields visible on a single page with no navigation required.

## Purpose
Test the ghostwriter agent's ability to:
- Identify and fill all visible form fields
- Handle HTML5 validation (`required` attribute)
- Submit a form and detect confirmation

## Structure

```
1-single-page/
├── index.html        # The application form
├── confirmation.html # Success page after submission
└── ARCHITECTURE.md   # This file
```

## Form Fields

| Field | Type | Required | Validation |
|-------|------|----------|------------|
| Full Name | text | Yes | required |
| Email Address | email | Yes | required, email format |
| Phone Number | tel | Yes | required |
| LinkedIn Profile | url | No | URL format |
| Resume | file | Yes | required, accepts .pdf/.doc/.docx |
| Cover Letter | textarea | No | none |
| Earliest Start Date | date | Yes | required |
| Desired Salary | number | No | min=0, step=1000 |

## Technical Details

### HTML5 Validation
- Uses native `required` attributes for mandatory fields
- Email field uses `type="email"` for format validation
- File input restricts to document types via `accept` attribute
- Number input has `min` and `step` constraints

### Form Submission
```javascript
function handleSubmit(e) {
  e.preventDefault();
  window.location.href = 'confirmation.html';
  return false;
}
```
- Prevents actual form submission
- Redirects to confirmation page
- No data is sent to a server

### Styling
- Uses shared `styles.css` from parent directory
- Form groups provide consistent spacing
- Required fields marked with red asterisk via CSS

## Agent Testing Scenarios

1. **Basic Fill**: Agent should identify all 8 fields and fill appropriately
2. **Validation Handling**: If agent submits without required fields, browser validation should trigger
3. **File Upload**: Agent should be able to attach a file to the resume field
4. **Confirmation Detection**: Agent should recognize the confirmation page as success

## Limitations

- No server-side validation (all client-side)
- No actual data persistence
- No error state testing (always succeeds if validation passes)
