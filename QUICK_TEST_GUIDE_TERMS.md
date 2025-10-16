# Quick Test Guide - Terms & Conditions

## All Issues Fixed! ✅

I've fixed all three issues you reported:

1. ✅ **Schema mismatch** - Changed `title` → `name` across all files
2. ✅ **"All available terms added" visibility** - Terms list is always shown below
3. ✅ **PDF rendering** - Fixed backend to use correct field name
4. ✅ **Backend rebuilt** - Changes applied
5. ✅ **Frontend restarted** - New code active

---

## Test Now (5 minutes)

### Test 1: Add Terms to a Quote

1. **Navigate to edit page**
   ```
   http://localhost:3000/quotes/1/edit
   ```

2. **Scroll down** to "Terms & Conditions" section (below the quote form)

3. **Select a template** from dropdown:
   - You should see: "Payment Terms", "Warranty Terms", etc.
   - NOT: undefined or blank entries

4. **Click "Add"** button

5. **Verify the terms list appears below** showing:
   - ✓ Position badge with "1"
   - ✓ Term name "Payment Terms" (or whichever you selected)
   - ✓ Category chip (e.g., "PAYMENT_TERMS")
   - ✓ Content preview (first few lines)
   - ✓ Edit and Delete buttons

6. **Try editing** the term:
   - Click edit (pencil) icon
   - Modal should show term name in header
   - Make a small change
   - Click "Save Changes"
   - Should see "Term updated successfully" toast

7. **Add another term**:
   - Select different template from dropdown
   - Click "Add"
   - Should see it appear in list as position "2"
   - Use arrow buttons to reorder if desired

---

### Test 2: Generate PDF with Terms

1. **Save your quote** (if you made any changes)

2. **Navigate to quote detail view**:
   ```
   http://localhost:3000/quotes/1
   ```

3. **Click "Download PDF"** button

4. **Open the PDF** and scroll to the bottom

5. **Verify you see**:
   ```
   Terms and Conditions
   
   1. Payment Terms
   [Full content of payment terms...]
   
   2. [Second term if you added one]
   [Content...]
   ```

6. **Check formatting**:
   - ✓ Section titled "Terms and Conditions" (14pt bold)
   - ✓ Numbered terms (1, 2, 3...)
   - ✓ Term titles in bold (11pt)
   - ✓ Content justified (10pt)
   - ✓ Proper spacing between terms

---

### Test 3: Verify "All Terms Added" Message

1. **Keep adding terms** until dropdown is empty

2. **Verify you see**:
   - "All available terms have been added to this document."
   - AND the full list of all 7 terms below

3. **This is correct behavior!** 
   - The message tells you can't add more
   - The list shows what you have
   - You can still edit, reorder, and delete existing terms

---

## What Should Work Now

### ✅ Templates Display
- Dropdown shows template names (not undefined)
- Preview panel shows correct info
- Category and description visible

### ✅ Terms List
- Shows position numbers
- Displays term names correctly
- Content preview works
- Edit/Delete buttons functional
- Reorder arrows work

### ✅ Edit Modal
- Shows term name in header
- Content editable
- Save/Cancel work
- "Restore Template" button works

### ✅ PDF Generation
- Terms section appears at bottom
- Terms numbered correctly
- Titles and content display
- Custom content used when edited
- Professional formatting

---

## If Something Still Doesn't Work

### Issue: Dropdown shows undefined or blank
**Cause:** Frontend still using old code (cache issue)
**Fix:** 
```bash
# Hard refresh browser (Ctrl+Shift+R or Cmd+Shift+R)
# OR clear browser cache
# OR try incognito/private window
```

### Issue: Terms don't appear in PDF
**Cause:** No terms actually attached to the quote
**Fix:**
1. Go to quote edit page
2. Scroll down to Terms & Conditions
3. Verify terms list is NOT empty
4. If empty, add at least one term
5. Try PDF generation again

### Issue: PDF shows "undefined" for term names
**Cause:** Backend still using old compiled code
**Fix:**
```bash
docker compose restart backend
# Wait 10 seconds for backend to start
# Try PDF generation again
```

### Issue: "All available terms added" but list is empty
**This is a bug!** Let me know and I'll investigate.

---

## Create New Documents with Terms

### Important: You CAN'T add terms during creation
**Why:** Terms Management component needs a document ID, which doesn't exist until document is saved.

**Workaround (Recommended):**
1. Create the quote/invoice/change order normally
2. After creation, click "Edit" button
3. Scroll down to Terms & Conditions section
4. Add your terms
5. Save
6. Generate PDF

**This is by design** - adding terms is a post-creation step, like editing any other field.

---

## Success Criteria

You'll know everything is working when:

1. ✅ You can see and select all 7 terms templates
2. ✅ Selected terms appear in the list with correct names
3. ✅ You can edit, reorder, and delete terms
4. ✅ Generated PDF includes "Terms and Conditions" section
5. ✅ PDF shows term titles and content correctly formatted
6. ✅ All of the above works for Quotes, Invoices, AND Change Orders

---

## Next Steps After Testing

Once you verify everything works:

1. **Commit the changes**:
   ```bash
   git add .
   git commit -m "Fix: Schema alignment for terms templates (title→name)"
   ```

2. **Update the todo list** (mark "Test terms management UI end-to-end" as complete)

3. **Test the same flow** on Invoices and Change Orders

4. **Ready for Phase 4** when you are!

---

## Need Help?

If any test fails, let me know:
- What you were doing (which step)
- What you expected to see
- What actually happened
- Any error messages in browser console (F12)

I can check logs, database state, or debug further!
