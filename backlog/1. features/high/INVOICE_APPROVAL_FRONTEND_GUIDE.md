# Frontend Implementation - Invoice Approval UI

**File:** `apps/frontend/src/pages/portal/PortalInvoiceDetailPage.tsx`

This document shows all changes needed to add invoice approval functionality to the portal invoice detail page.

---

## Changes Overview

1. Add state management for dialogs and user input
2. Add React Query mutations for accept/dispute
3. Add action buttons (visible only for SENT invoices)
4. Add Accept Invoice dialog
5. Add Dispute Invoice dialog
6. Add History dialog (optional enhancement)

---

## Step 1: Add Imports

Add these imports to the existing imports section:

```typescript
import {
  Dialog,
  DialogTitle,
  DialogContent,
  DialogActions,
  TextField,
  FormControl,
  InputLabel,
  Select,
  MenuItem,
  Stack,
} from '@mui/material';
import {
  CheckCircle,
  Warning,
  History,
} from '@mui/icons-material';
import { useMutation, useQueryClient } from '@tanstack/react-query';
```

---

## Step 2: Add State Variables

Add these state variables after the existing `useState` declarations:

```typescript
const [acceptDialogOpen, setAcceptDialogOpen] = useState(false);
const [disputeDialogOpen, setDisputeDialogOpen] = useState(false);
const [historyDialogOpen, setHistoryDialogOpen] = useState(false);
const [notes, setNotes] = useState('');
const [disputeReason, setDisputeReason] = useState('');
```

---

## Step 3: Add React Query Mutations

Add these mutations after the state declarations:

```typescript
const queryClient = useQueryClient();

// Accept invoice mutation
const acceptMutation = useMutation({
  mutationFn: async ({ notes }: { notes?: string }) => {
    const response = await api.post(`/api/portal/invoices/${id}/accept`, { notes });
    return response.data;
  },
  onSuccess: () => {
    toast.success('Invoice accepted successfully!');
    queryClient.invalidateQueries({ queryKey: ['portal-invoice', id] });
    queryClient.invalidateQueries({ queryKey: ['portal-invoices'] });
    queryClient.invalidateQueries({ queryKey: ['portal-dashboard'] });
    setAcceptDialogOpen(false);
    setNotes('');
    fetchInvoice(); // Refresh invoice data
  },
  onError: (error: any) => {
    console.error('Accept error:', error);
    toast.error(error.response?.data?.error || 'Failed to accept invoice');
  },
});

// Dispute invoice mutation
const disputeMutation = useMutation({
  mutationFn: async ({ notes, reason }: { notes: string; reason: string }) => {
    const response = await api.post(`/api/portal/invoices/${id}/dispute`, { notes, reason });
    return response.data;
  },
  onSuccess: () => {
    toast.success('Invoice dispute submitted successfully');
    queryClient.invalidateQueries({ queryKey: ['portal-invoice', id] });
    queryClient.invalidateQueries({ queryKey: ['portal-invoices'] });
    queryClient.invalidateQueries({ queryKey: ['portal-dashboard'] });
    setDisputeDialogOpen(false);
    setNotes('');
    setDisputeReason('');
    fetchInvoice(); // Refresh invoice data
  },
  onError: (error: any) => {
    console.error('Dispute error:', error);
    toast.error(error.response?.data?.error || 'Failed to submit dispute');
  },
});

// Handler functions
const handleAccept = () => {
  acceptMutation.mutate({ notes });
};

const handleDispute = () => {
  if (!notes.trim() || !disputeReason) {
    toast.error('Please provide both a reason and detailed feedback');
    return;
  }
  disputeMutation.mutate({ notes, reason: disputeReason });
};
```

---

## Step 4: Add Action Buttons

Find where the invoice details are displayed and add action buttons.
Add this section after the invoice summary but before the line items table:

```typescript
{/* Action Buttons - Only show for SENT invoices */}
{invoice.status === 'SENT' && (
  <Box sx={{ mt: 3, mb: 3 }}>
    <Alert severity="info" sx={{ mb: 2 }}>
      <Typography variant="body2">
        Please review this invoice carefully. You can accept it if everything is correct, 
        or dispute it if you have any concerns.
      </Typography>
    </Alert>
    
    <Stack direction={{ xs: 'column', sm: 'row' }} spacing={2}>
      <Button
        variant="contained"
        color="success"
        size="large"
        startIcon={<CheckCircle />}
        onClick={() => setAcceptDialogOpen(true)}
        disabled={acceptMutation.isPending || disputeMutation.isPending}
        fullWidth
      >
        Accept Invoice
      </Button>
      <Button
        variant="outlined"
        color="warning"
        size="large"
        startIcon={<Warning />}
        onClick={() => setDisputeDialogOpen(true)}
        disabled={acceptMutation.isPending || disputeMutation.isPending}
        fullWidth
      >
        Dispute Invoice
      </Button>
    </Stack>
  </Box>
)}

{/* Status Messages for Accepted/Disputed */}
{invoice.status === 'ACCEPTED' && (
  <Alert severity="success" sx={{ mt: 3, mb: 3 }}>
    <Typography variant="body2" fontWeight="bold">
      ✅ You have accepted this invoice
    </Typography>
    <Typography variant="caption">
      Thank you for confirming. Payment processing will proceed as scheduled.
    </Typography>
  </Alert>
)}

{invoice.status === 'DISPUTED' && (
  <Alert severity="warning" sx={{ mt: 3, mb: 3 }}>
    <Typography variant="body2" fontWeight="bold">
      ⚠️ This invoice has been disputed
    </Typography>
    <Typography variant="caption">
      Our team has been notified and will contact you shortly to resolve this issue.
    </Typography>
  </Alert>
)}
```

---

## Step 5: Add Accept Dialog

Add this dialog component before the closing tag of your component:

```typescript
{/* Accept Invoice Dialog */}
<Dialog 
  open={acceptDialogOpen} 
  onClose={() => !acceptMutation.isPending && setAcceptDialogOpen(false)} 
  maxWidth="sm" 
  fullWidth
>
  <DialogTitle>
    <Stack direction="row" alignItems="center" spacing={1}>
      <CheckCircle color="success" />
      <Typography variant="h6">Accept Invoice</Typography>
    </Stack>
  </DialogTitle>
  <DialogContent>
    <Typography variant="body1" gutterBottom sx={{ mt: 1 }}>
      Are you sure you want to accept invoice <strong>{invoice?.number}</strong>?
    </Typography>
    
    <Typography variant="body2" color="text.secondary" sx={{ mt: 1, mb: 2 }}>
      By accepting this invoice, you confirm that:
    </Typography>
    <Box component="ul" sx={{ pl: 2, mt: 0 }}>
      <Typography component="li" variant="body2" color="text.secondary">
        All items and amounts are correct
      </Typography>
      <Typography component="li" variant="body2" color="text.secondary">
        Services/products have been delivered as described
      </Typography>
      <Typography component="li" variant="body2" color="text.secondary">
        You approve this invoice for payment processing
      </Typography>
    </Box>
    
    <TextField
      label="Notes (Optional)"
      multiline
      rows={3}
      fullWidth
      value={notes}
      onChange={(e) => setNotes(e.target.value)}
      placeholder="Add any comments about your acceptance..."
      sx={{ mt: 2 }}
    />
  </DialogContent>
  <DialogActions sx={{ px: 3, pb: 2 }}>
    <Button 
      onClick={() => setAcceptDialogOpen(false)}
      disabled={acceptMutation.isPending}
    >
      Cancel
    </Button>
    <Button
      onClick={handleAccept}
      variant="contained"
      color="success"
      disabled={acceptMutation.isPending}
      startIcon={acceptMutation.isPending ? <CircularProgress size={20} /> : <CheckCircle />}
    >
      {acceptMutation.isPending ? 'Accepting...' : 'Confirm Acceptance'}
    </Button>
  </DialogActions>
</Dialog>
```

---

## Step 6: Add Dispute Dialog

Add this dialog component after the Accept dialog:

```typescript
{/* Dispute Invoice Dialog */}
<Dialog 
  open={disputeDialogOpen} 
  onClose={() => !disputeMutation.isPending && setDisputeDialogOpen(false)} 
  maxWidth="sm" 
  fullWidth
>
  <DialogTitle>
    <Stack direction="row" alignItems="center" spacing={1}>
      <Warning color="warning" />
      <Typography variant="h6">Dispute Invoice</Typography>
    </Stack>
  </DialogTitle>
  <DialogContent>
    <Typography variant="body1" gutterBottom sx={{ mt: 1 }}>
      Please provide details about the issue with invoice <strong>{invoice?.number}</strong>.
    </Typography>
    
    <Alert severity="info" sx={{ mt: 2, mb: 3 }}>
      <Typography variant="body2">
        Our team will review your dispute and contact you within 1-2 business days to resolve the issue.
      </Typography>
    </Alert>
    
    <FormControl fullWidth sx={{ mb: 2 }}>
      <InputLabel>Reason for Dispute *</InputLabel>
      <Select
        value={disputeReason}
        onChange={(e) => setDisputeReason(e.target.value)}
        label="Reason for Dispute *"
        required
      >
        <MenuItem value="INCORRECT_AMOUNT">Incorrect Amount</MenuItem>
        <MenuItem value="INCORRECT_ITEMS">Incorrect Items/Services</MenuItem>
        <MenuItem value="WORK_NOT_COMPLETED">Work Not Completed</MenuItem>
        <MenuItem value="QUALITY_ISSUES">Quality Issues</MenuItem>
        <MenuItem value="BILLING_ERROR">Billing Error</MenuItem>
        <MenuItem value="DUPLICATE_CHARGE">Duplicate Charge</MenuItem>
        <MenuItem value="OTHER">Other Issue</MenuItem>
      </Select>
    </FormControl>
    
    <TextField
      label="Detailed Explanation *"
      multiline
      rows={4}
      fullWidth
      value={notes}
      onChange={(e) => setNotes(e.target.value)}
      placeholder="Please explain the issue in detail so we can resolve it quickly..."
      required
      error={notes.length > 0 && notes.length < 10}
      helperText={notes.length > 0 && notes.length < 10 ? 'Please provide more details (at least 10 characters)' : ''}
    />
  </DialogContent>
  <DialogActions sx={{ px: 3, pb: 2 }}>
    <Button 
      onClick={() => {
        setDisputeDialogOpen(false);
        setNotes('');
        setDisputeReason('');
      }}
      disabled={disputeMutation.isPending}
    >
      Cancel
    </Button>
    <Button
      onClick={handleDispute}
      variant="contained"
      color="warning"
      disabled={disputeMutation.isPending || !disputeReason || notes.length < 10}
      startIcon={disputeMutation.isPending ? <CircularProgress size={20} /> : <Warning />}
    >
      {disputeMutation.isPending ? 'Submitting...' : 'Submit Dispute'}
    </Button>
  </DialogActions>
</Dialog>
```

---

## Step 7: Add History Dialog (Optional)

If you want to show acceptance/dispute history:

```typescript
{/* Invoice History Dialog */}
<Dialog 
  open={historyDialogOpen} 
  onClose={() => setHistoryDialogOpen(false)} 
  maxWidth="md" 
  fullWidth
>
  <DialogTitle>
    <Stack direction="row" alignItems="center" spacing={1}>
      <History color="primary" />
      <Typography variant="h6">Invoice History</Typography>
    </Stack>
  </DialogTitle>
  <DialogContent>
    <Typography variant="body2" color="text.secondary">
      Acceptance and dispute history will be displayed here.
    </Typography>
    {/* TODO: Fetch and display history */}
  </DialogContent>
  <DialogActions>
    <Button onClick={() => setHistoryDialogOpen(false)}>Close</Button>
  </DialogActions>
</Dialog>
```

---

## Step 8: Add History Button (Optional)

Add this button near the top of the page (in the header section):

```typescript
<Tooltip title="View History">
  <IconButton onClick={() => setHistoryDialogOpen(true)} color="primary">
    <History />
  </IconButton>
</Tooltip>
```

---

## Complete Code Structure

Your updated component should have this structure:

```typescript
export const PortalInvoiceDetailPage: React.FC = () => {
  // 1. useParams and navigation
  const { id } = useParams<{ id: string }>();
  const navigate = useNavigate();
  
  // 2. Existing state
  const [invoice, setInvoice] = useState<Invoice | null>(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<string | null>(null);
  
  // 3. NEW: Dialog state
  const [acceptDialogOpen, setAcceptDialogOpen] = useState(false);
  const [disputeDialogOpen, setDisputeDialogOpen] = useState(false);
  const [historyDialogOpen, setHistoryDialogOpen] = useState(false);
  const [notes, setNotes] = useState('');
  const [disputeReason, setDisputeReason] = useState('');
  
  // 4. NEW: React Query mutations
  const queryClient = useQueryClient();
  const acceptMutation = useMutation({ ... });
  const disputeMutation = useMutation({ ... });
  
  // 5. Existing useEffect for fetching invoice
  useEffect(() => { ... });
  
  // 6. Helper functions
  const getStatusColor = (status: string) => { ... };
  
  // 7. Render
  return (
    <Box>
      {/* Existing content */}
      
      {/* NEW: Action buttons */}
      {invoice.status === 'SENT' && ( ... )}
      
      {/* NEW: Status messages */}
      {invoice.status === 'ACCEPTED' && ( ... )}
      {invoice.status === 'DISPUTED' && ( ... )}
      
      {/* Existing invoice details */}
      
      {/* NEW: Dialogs */}
      <Dialog>Accept Dialog</Dialog>
      <Dialog>Dispute Dialog</Dialog>
      <Dialog>History Dialog</Dialog>
    </Box>
  );
};
```

---

## Testing Checklist

After implementing:

- [ ] Accept button only shows for SENT invoices
- [ ] Dispute button only shows for SENT invoices
- [ ] Accept dialog opens and closes correctly
- [ ] Dispute dialog opens and closes correctly
- [ ] Form validation prevents empty submissions
- [ ] Success messages appear after actions
- [ ] Invoice status updates immediately
- [ ] Error messages display for API failures
- [ ] Loading states prevent double-submissions
- [ ] Mobile responsive design works

---

## Notes

1. **Toast Notifications**: Assumes you have a toast/snackbar system (like `react-toastify` or Material-UI Snackbar)
2. **React Query**: Uses `@tanstack/react-query` for mutations
3. **API Instance**: Uses the existing `api` instance from `portal-axios-instance`
4. **Icons**: Uses Material-UI icons (CheckCircle, Warning, History)

---

**Implementation Time:** 2-3 hours for complete frontend changes
