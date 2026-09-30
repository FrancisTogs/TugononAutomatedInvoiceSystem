<script lang="ts">
  type InvoicePayment = {
    invoiceNo: string;
    clientName: string;
    totalAmount: number;
    amountPaid: number;
    status: 'PAID' | 'PARTIAL' | 'UNPAID' | 'OVERDUE';
  };

  // --- Initial Payment Status Records ---
  let invoices = $state<InvoicePayment[]>([
    { invoiceNo: 'INV-2026-015', clientName: 'Cebu Construction Corp', totalAmount: 168000, amountPaid: 168000, status: 'PAID' },
    { invoiceNo: 'INV-2026-016', clientName: 'Visayas Builders Inc', totalAmount: 504000, amountPaid: 250000, status: 'PARTIAL' },
    { invoiceNo: 'INV-2026-017', clientName: 'Metro Cebu Holdings', totalAmount: 313600, amountPaid: 0, status: 'UNPAID' },
    { invoiceNo: 'INV-2026-018', clientName: 'Subangdaku Logistics', totalAmount: 85000, amountPaid: 0, status: 'OVERDUE' }
  ]);

  let statusFilter = $state('ALL');
  let selectedInvoiceNo = $state<string | null>(null);

  // --- New Payment Entry Inputs ---
  let paymentAmount = $state<number>(0);
  let paymentMethod = $state('Bank Transfer');
  let paymentDate = $state(new Date().toISOString().split('T')[0]);
  let referenceNo = $state('');

  // --- Derived State ---
  let filteredInvoices = $derived(
    invoices.filter(inv => statusFilter === 'ALL' || inv.status === statusFilter)
  );

  let selectedInvoice = $derived(
    invoices.find(inv => inv.invoiceNo === selectedInvoiceNo) || null
  );

  let remainingBalance = $derived(
    selectedInvoice ? selectedInvoice.totalAmount - selectedInvoice.amountPaid : 0
  );

  // --- Actions ---
  function selectInvoiceForPayment(inv: InvoicePayment) {
    if (inv.status === 'PAID') {
      alert("This invoice has already been fully paid!");
      return;
    }
    selectedInvoiceNo = inv.invoiceNo;
    paymentAmount = inv.totalAmount - inv.amountPaid;
    referenceNo = `REF-${Math.floor(100000 + Math.random() * 900000)}`;
  }

  function handleProcessPayment(e: Event) {
    e.preventDefault();
    if (!selectedInvoice) return alert("Select an invoice to record payment.");
    if (paymentAmount <= 0) return alert("Please enter a valid payment amount.");
    if (paymentAmount > remainingBalance) return alert("Payment amount cannot exceed remaining balance!");

    const newAmountPaid = selectedInvoice.amountPaid + paymentAmount;
    const newStatus: InvoicePayment['status'] = newAmountPaid >= selectedInvoice.totalAmount ? 'PAID' : 'PARTIAL';

    invoices = invoices.map(inv => 
      inv.invoiceNo === selectedInvoiceNo 
        ? { ...inv, amountPaid: newAmountPaid, status: newStatus } 
        : inv
    );

    alert(`Payment of ₱${paymentAmount.toLocaleString()} recorded for ${selectedInvoice.invoiceNo}. Ref: ${referenceNo}`);
    selectedInvoiceNo = null;
    paymentAmount = 0;
    referenceNo = '';
  }
</script>

<section class="card">
  <div class="card-header-flex">
    <div>
      <h2>💳 Payment Status & Monitoring</h2>
      <p class="subtitle">Monitor client billing balances and process incoming payments</p>
    </div>
  </div>

  <!-- Status Filter Tabs -->
  <div class="filter-tabs">
    <button class="filter-btn {statusFilter === 'ALL' ? 'active' : ''}" onclick={() => statusFilter = 'ALL'}>All Invoices ({invoices.length})</button>
    <button class="filter-btn {statusFilter === 'UNPAID' ? 'active' : ''}" onclick={() => statusFilter = 'UNPAID'}>Unpaid</button>
    <button class="filter-btn {statusFilter === 'PARTIAL' ? 'active' : ''}" onclick={() => statusFilter = 'PARTIAL'}>Partial</button>
    <button class="filter-btn {statusFilter === 'OVERDUE' ? 'active' : ''}" onclick={() => statusFilter = 'OVERDUE'}>Overdue</button>
    <button class="filter-btn {statusFilter === 'PAID' ? 'active' : ''}" onclick={() => statusFilter = 'PAID'}>Paid</button>
  </div>

  <!-- Invoice Payment Status List Table -->
  <div class="table-wrapper">
    <table class="data-table">
      <thead>
        <tr>
          <th>Invoice No</th>
          <th>Client Name</th>
          <th>Total Amount</th>
          <th>Amount Paid</th>
          <th>Remaining Balance</th>
          <th>Status</th>
          <th style="text-align: center;">Action</th>
        </tr>
      </thead>
      <tbody>
        {#each filteredInvoices as inv}
          {@const balance = inv.totalAmount - inv.amountPaid}
          <tr class={selectedInvoiceNo === inv.invoiceNo ? 'selected-row' : ''}>
            <td class="font-bold text-blue">{inv.invoiceNo}</td>
            <td class="font-bold text-white">{inv.clientName}</td>
            <td>₱{inv.totalAmount.toLocaleString()}</td>
            <td class="text-green">₱{inv.amountPaid.toLocaleString()}</td>
            <td class="font-bold {balance > 0 ? 'text-warn' : 'text-dim'}">₱{balance.toLocaleString()}</td>
            <td>
              <span class="status-badge status-{inv.status.toLowerCase()}">{inv.status}</span>
            </td>
            <td style="text-align: center;">
              {#if inv.status !== 'PAID'}
                <button class="btn-sm-action" onclick={() => selectInvoiceForPayment(inv)}>Record Payment</button>
              {:else}
                <span class="text-dim text-sm">✔ Cleared</span>
              {/if}
            </td>
          </tr>
        {/each}
      </tbody>
    </table>
  </div>

  <!-- RECORD PAYMENT PANEL -->
  {#if selectedInvoice}
    <div class="payment-entry-panel">
      <h3>Record Payment for <span class="text-blue">{selectedInvoice.invoiceNo}</span> ({selectedInvoice.clientName})</h3>
      
      <form onsubmit={handleProcessPayment} class="form-container">
        <div class="grid-3">
          <div class="form-group">
            <label for="pAmount">Amount Received (PHP)</label>
            <input type="number" id="pAmount" bind:value={paymentAmount} min="1" max={remainingBalance} required />
            <span class="input-hint">Max Balance: ₱{remainingBalance.toLocaleString()}</span>
          </div>

          <div class="form-group">
            <label for="pMethod">Payment Method</label>
            <select id="pMethod" bind:value={paymentMethod}>
              <option>Cash</option>
              <option>Check</option>
              <option>Bank Transfer</option>
              <option>Online Payment</option>
            </select>
          </div>

          <div class="form-group">
            <label for="pDate">Payment Date</label>
            <input type="date" id="pDate" bind:value={paymentDate} required />
          </div>
        </div>

        <div class="grid-2" style="margin-top: 0.8rem;">
          <div class="form-group">
            <label for="pRef">Reference / Check / Deposit No.</label>
            <input type="text" id="pRef" bind:value={referenceNo} placeholder="e.g. TRN-9988231" required />
          </div>

          <div class="action-buttons-end">
            <button type="button" class="btn-secondary" onclick={() => selectedInvoiceNo = null}>Cancel</button>
            <button type="submit" class="btn-primary">Process & Confirm Payment</button>
          </div>
        </div>
      </form>
    </div>
  {/if}
</section>

<style>
  .card-header-flex { border-bottom: 1px solid #334155; padding-bottom: 0.8rem; margin-bottom: 1rem; }
  .subtitle { margin: 0.2rem 0 0 0; font-size: 0.85rem; color: #94a3b8; }

  .filter-tabs { display: flex; gap: 0.5rem; margin-bottom: 1rem; overflow-x: auto; }
  .filter-btn { background: #0f172a; border: 1px solid #334155; color: #94a3b8; padding: 0.4rem 0.8rem; border-radius: 6px; font-size: 0.85rem; cursor: pointer; }
  .filter-btn.active { background: #2563eb; color: white; border-color: #3b82f6; }

  .table-wrapper { overflow-x: auto; border: 1px solid #334155; border-radius: 8px; }
  .data-table { width: 100%; border-collapse: collapse; text-align: left; font-size: 0.9rem; }
  .data-table th { background: #0f172a; color: #94a3b8; padding: 0.8rem 1rem; border-bottom: 1px solid #334155; }
  .data-table td { padding: 0.8rem 1rem; border-bottom: 1px solid #1e293b; vertical-align: middle; }
  .selected-row { background: #1e293b; border-left: 3px solid #38bdf8; }

  .font-bold { font-weight: bold; }
  .text-white { color: #f8fafc; }
  .text-blue { color: #38bdf8; }
  .text-green { color: #4ade80; }
  .text-warn { color: #fbbf24; }
  .text-dim { color: #64748b; }
  .text-sm { font-size: 0.8rem; }

  .status-badge { font-size: 0.75rem; font-weight: bold; padding: 0.2rem 0.6rem; border-radius: 12px; display: inline-block; }
  .status-paid { background: #14532d; color: #86efac; }
  .status-partial { background: #1e3a8a; color: #93c5fd; }
  .status-unpaid { background: #78350f; color: #fde047; }
  .status-overdue { background: #7f1d1d; color: #fca5a5; }

  .btn-sm-action { background: #2563eb; color: white; border: none; padding: 0.35rem 0.7rem; border-radius: 4px; font-size: 0.8rem; font-weight: bold; cursor: pointer; }

  .payment-entry-panel { background: #0f172a; border: 1px solid #38bdf8; padding: 1.2rem; border-radius: 8px; margin-top: 1.5rem; }
  .payment-entry-panel h3 { margin-top: 0; color: #f8fafc; font-size: 1rem; border-bottom: 1px solid #334155; padding-bottom: 0.5rem; }

  .form-container { display: flex; flex-direction: column; gap: 1rem; margin-top: 0.8rem; }
  .grid-2 { display: grid; grid-template-columns: 1fr 1fr; gap: 1rem; }
  .grid-3 { display: grid; grid-template-columns: 1fr 1fr 1fr; gap: 1rem; }
  .form-group { display: flex; flex-direction: column; gap: 0.3rem; }
  .form-group label { font-size: 0.8rem; color: #cbd5e1; font-weight: 600; }
  .input-hint { font-size: 0.75rem; color: #94a3b8; }

  .action-buttons-end { display: flex; justify-content: flex-end; align-items: flex-end; gap: 0.8rem; }
  .btn-primary { background: #10b981; color: white; border: none; padding: 0.8rem 1.2rem; border-radius: 6px; font-weight: bold; cursor: pointer; }
  .btn-secondary { background: #334155; color: white; border: none; padding: 0.8rem 1rem; border-radius: 6px; cursor: pointer; }
</style>