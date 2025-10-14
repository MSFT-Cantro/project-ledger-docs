# Status Component Testing Guide

## Manual Testing Checklist

### QuoteDetailPage Testing

#### Test 1: Draft Quote Status Changes
1. Navigate to a quote with status "Draft"
2. Click the "Status" button in the action buttons row
3. **Expected**: Menu opens with only "Mark as Sent" option
4. Click "Mark as Sent"
5. **Expected**: Confirmation dialog appears: "Mark this quote as sent?"
6. Click "OK"
7. **Expected**: 
   - Success toast appears
   - Status chip changes to "Sent" with blue color
   - Menu closes

#### Test 2: Sent Quote Status Changes
1. Navigate to a quote with status "Sent" (or use quote from Test 1)
2. Click the "Status" button
3. **Expected**: Menu opens with two options:
   - "Mark as Accepted" (with CheckCircle icon)
   - "Mark as Rejected" (with Cancel icon)
4. Click "Mark as Accepted"
5. **Expected**: Confirmation dialog appears
6. Click "OK"
7. **Expected**: Status chip changes to "Accepted" with green color

#### Test 3: Accepted Quote (Final State)
1. Navigate to a quote with status "Accepted"
2. Click the "Status" button
3. **Expected**: Menu opens but is empty (no options)
4. **Reason**: Accepted quotes cannot change status

#### Test 4: Rejected Quote Can Resend
1. Navigate to a quote with status "Rejected"
2. Click the "Status" button
3. **Expected**: Menu opens with "Mark as Sent" option
4. This allows resending rejected quotes

#### Test 5: Cancel Confirmation Dialog
1. Navigate to any quote with available transitions
2. Click "Status" button
3. Click any status option
4. **Expected**: Confirmation dialog appears
5. Click "Cancel"
6. **Expected**: 
   - Status does NOT change
   - No toast message
   - Menu closes

#### Test 6: Status Chip Display
1. Navigate to quotes with different statuses
2. **Expected Chip Colors**:
   - Draft: Gray (default)
   - Sent: Blue (primary)
   - Accepted: Green (success)
   - Rejected: Red (error)
3. All chips should have "outlined" variant

#### Test 7: Action Buttons Layout
1. Navigate to any quote detail page
2. **Expected Layout** (centered row below header):
   - Status button (primary, with ChangeCircle icon)
   - PDF button (outlined, with Download icon)
   - Print button (outlined, with Print icon)
3. Buttons should wrap on small screens

#### Test 8: Error Handling
1. Stop the backend server
2. Try to change quote status
3. **Expected**: 
   - Error toast appears
   - Status does NOT change in UI
   - Chip remains at original status

### InvoiceDetailPage Testing

#### Test 1: Verify Existing Functionality Still Works
1. Navigate to any invoice detail page
2. Click "Status" button
3. **Expected**: Menu opens with appropriate options
4. Verify confirmation dialogs still appear
5. Verify status changes work correctly

#### Test 2: Compare with Quote Page
1. Open quote detail page in one tab
2. Open invoice detail page in another tab
3. **Expected**: Both pages have:
   - Similar action button layout
   - Status button in same position
   - Same confirmation dialog pattern
   - Same visual feedback (toasts)

### Cross-Page Consistency Tests

#### Test 1: Visual Consistency
Compare the two pages and verify:
- ✅ Both use Button + Menu for status changes
- ✅ Both show status as outlined Chip
- ✅ Both have action buttons in centered row
- ✅ Both use same icons for similar actions
- ✅ Both have same button styling

#### Test 2: Interaction Consistency
- ✅ Both require confirmation before changing status
- ✅ Both close menu after selection
- ✅ Both show success/error toasts
- ✅ Both disable options for final states
- ✅ Both use same keyboard navigation

#### Test 3: Responsive Behavior
Test on different screen sizes:
1. Desktop (>1200px): Buttons in single row
2. Tablet (768-1200px): Buttons may wrap
3. Mobile (<768px): Buttons stack or wrap

## Automated Testing Recommendations

### Unit Tests for Helper Functions

```typescript
describe('getAvailableStatusTransitions', () => {
  it('should return [SENT] for DRAFT status', () => {
    expect(getAvailableStatusTransitions(QuoteStatus.DRAFT))
      .toEqual([QuoteStatus.SENT]);
  });

  it('should return [ACCEPTED, REJECTED] for SENT status', () => {
    expect(getAvailableStatusTransitions(QuoteStatus.SENT))
      .toEqual([QuoteStatus.ACCEPTED, QuoteStatus.REJECTED]);
  });

  it('should return empty array for ACCEPTED status', () => {
    expect(getAvailableStatusTransitions(QuoteStatus.ACCEPTED))
      .toEqual([]);
  });

  it('should return [SENT] for REJECTED status', () => {
    expect(getAvailableStatusTransitions(QuoteStatus.REJECTED))
      .toEqual([QuoteStatus.SENT]);
  });
});

describe('getStatusColor', () => {
  it('should return "default" for DRAFT', () => {
    expect(getStatusColor(QuoteStatus.DRAFT)).toBe('default');
  });

  it('should return "primary" for SENT', () => {
    expect(getStatusColor(QuoteStatus.SENT)).toBe('primary');
  });

  it('should return "success" for ACCEPTED', () => {
    expect(getStatusColor(QuoteStatus.ACCEPTED)).toBe('success');
  });

  it('should return "error" for REJECTED', () => {
    expect(getStatusColor(QuoteStatus.REJECTED)).toBe('error');
  });
});
```

### Integration Tests

```typescript
describe('QuoteDetailPage Status Changes', () => {
  it('should show confirmation dialog when changing status', async () => {
    render(<QuoteDetailPage />);
    
    const statusButton = screen.getByText('Status');
    fireEvent.click(statusButton);
    
    const markAsSent = screen.getByText('Mark as Sent');
    fireEvent.click(markAsSent);
    
    expect(window.confirm).toHaveBeenCalledWith('Mark this quote as sent?');
  });

  it('should update status when confirmed', async () => {
    window.confirm = jest.fn(() => true);
    render(<QuoteDetailPage />);
    
    const statusButton = screen.getByText('Status');
    fireEvent.click(statusButton);
    
    const markAsSent = screen.getByText('Mark as Sent');
    fireEvent.click(markAsSent);
    
    await waitFor(() => {
      expect(screen.getByText('Sent')).toBeInTheDocument();
    });
  });

  it('should not update status when cancelled', async () => {
    window.confirm = jest.fn(() => false);
    render(<QuoteDetailPage />);
    
    const statusButton = screen.getByText('Status');
    fireEvent.click(statusButton);
    
    const markAsSent = screen.getByText('Mark as Sent');
    fireEvent.click(markAsSent);
    
    // Status should remain Draft
    expect(screen.getByText('Draft')).toBeInTheDocument();
  });
});
```

## Performance Testing

### Test Scenarios:
1. **Menu Opening Speed**: Should open instantly
2. **Status Update Speed**: Should complete within 1-2 seconds
3. **UI Update Speed**: Chip should update immediately after API success
4. **No Flash**: Status chip should not flash between colors

## Accessibility Testing

### Keyboard Navigation:
1. Tab to "Status" button
2. Press Enter to open menu
3. Arrow keys to navigate menu items
4. Enter to select item
5. Escape to close menu without selection

### Screen Reader Testing:
1. Status button should announce "Status" button
2. Menu items should announce full text ("Mark as Sent")
3. Status chip should announce current status
4. Success/error toasts should be announced

### Color Contrast:
1. All chip colors should meet WCAG AA standards
2. Button text should be readable on all backgrounds
3. Menu text should have sufficient contrast

## Browser Compatibility

Test in:
- [ ] Chrome (latest)
- [ ] Firefox (latest)
- [ ] Safari (latest)
- [ ] Edge (latest)
- [ ] Mobile Safari (iOS)
- [ ] Chrome Mobile (Android)

## Known Issues & Limitations

### Non-Issues:
- Empty menu for final states is expected behavior
- Confirmation dialogs use native `window.confirm()` (intentional for simplicity)

### Potential Improvements:
- Replace `window.confirm()` with Material-UI Dialog for better styling
- Add keyboard shortcuts (e.g., Ctrl+Shift+S for "Send")
- Add status transition animations
- Add undo functionality for accidental changes

## Regression Testing

Verify these features still work:
- [ ] Edit Quote button
- [ ] Back button
- [ ] PDF download
- [ ] Print button
- [ ] More menu (Save as Template)
- [ ] Quote data loading
- [ ] Error states
- [ ] Loading states

## Test Data Requirements

Create test quotes with various statuses:
1. Quote in DRAFT status
2. Quote in SENT status
3. Quote in ACCEPTED status
4. Quote in REJECTED status

Verify database has quotes in each state, or create them via API/UI before testing.

## Success Criteria

✅ All manual tests pass
✅ Status changes work reliably
✅ Confirmation dialogs always appear
✅ Visual consistency between pages
✅ No console errors or warnings
✅ Responsive layout works on all screen sizes
✅ Accessibility standards met
✅ No performance regressions

## Test Report Template

```
# Status Component Testing Report
Date: [DATE]
Tester: [NAME]
Environment: [DEV/STAGING/PROD]

## Quote Page Tests
- [ ] Draft → Sent: PASS/FAIL
- [ ] Sent → Accepted: PASS/FAIL
- [ ] Sent → Rejected: PASS/FAIL
- [ ] Rejected → Sent: PASS/FAIL
- [ ] Accepted (no transitions): PASS/FAIL
- [ ] Confirmation dialogs: PASS/FAIL
- [ ] Status chip colors: PASS/FAIL
- [ ] Action buttons layout: PASS/FAIL

## Invoice Page Tests
- [ ] Existing functionality: PASS/FAIL
- [ ] Visual consistency: PASS/FAIL

## Cross-Page Tests
- [ ] Interaction consistency: PASS/FAIL
- [ ] Responsive behavior: PASS/FAIL

## Issues Found
1. [Description]
2. [Description]

## Notes
[Any additional observations]
```
