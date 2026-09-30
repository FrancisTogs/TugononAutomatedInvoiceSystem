<script lang="ts">
  type ApprovedQuotation = {
    id: number;
    quotationNo: string;
    clientName: string;
    projectTitle: string;
    subtotal: number;
  };

  // --- Mock Approved Quotations Ready for Invoicing ---
  let approvedQuotations: ApprovedQuotation[] = [
    { id: 201, quotationNo: 'QUO-2026-088', clientName: 'Cebu Construction Corp', projectTitle: 'Commercial Warehouse Phase 1', subtotal: 150000 },
    { id: 202, quotationNo: 'QUO-2026-092', clientName: 'Visayas Builders Inc', projectTitle: 'Site Excavation & Clearing', subtotal: 450000 }
  ];

  // --- Invoice Form State ---
  let selectedQuotationId = $state<number | null>(201);
  let invoiceNumber = $state('INV-2026-015'); // Auto-generated sequence pattern
  let invoiceDate = $state(new Date().toISOString().split('T')[0]);
  let paymentTerms = $state('Net 30 Days');

  let discountRate = $state(0); // Discount Percentage (e.g., 5%)
  let withholdingTaxRate = $state(2); // Withholding Tax Percentage (e.g., 2% BIR 2307)

  // --- Selected Quotation Derived State ---
  let selectedQuotation = $derived(approvedQuotations.find(q => q.id === selectedQuotationId) || null);

  // --- Financial Calculations (Svelte 5 $derived) ---
  let baseAmount = $derived(selectedQuotation?.subtotal || 0);
  let discountAmount = $derived(baseAmount * (discountRate / 100));
  let taxableSubtotal = $derived(baseAmount - discountAmount);
  
  let withholdingTaxAmount = $derived(taxableSubtotal * (withholdingTaxRate / 100));
  let vatAmount = $derived(taxableSubtotal * 0.12);
  
  let grandTotalInvoice = $derived(taxableSubtotal + vatAmount - withholdingTaxAmount);

  function handleSaveInvoice(event: Event) {
    event.preventDefault();
    if (!selectedQuotation) return alert("Select an approved quotation to invoice!");

    const invoicePayload = {
      invoiceNumber,
      quotationId: selectedQuotation.id,
      clientName: selectedQuotation.clientName,
      invoiceDate,
      paymentTerms,
      baseAmount,
      discountAmount,
      withholdingTaxAmount,
      vatAmount,
      grandTotalInvoice,
      status: 'UNPAID'
    };

    console.log("Invoice Created:", invoicePayload);
    alert(`Invoice ${invoiceNumber} created and saved successfully!`);
  }
</script>

<section class="card">
  <div class="card-header">
    <h2>📄 Finance Invoice CRUD & Generation</h2>
  </div>

  <form onsubmit={handleSaveInvoice} class="form-container">
    <!-- 1. INVOICE HEADER & QUOTATION SELECTION -->
    <div class="form-section">
      <h3>1. Invoice Details & Linked Quotation</h3>
      <div class="grid-3">
        <div class="form-group">
          <label for="invNum">Sequential Invoice Number</label>
          <input type="text" id="invNum" bind:value={invoiceNumber} class="font-bold highlight-input" readonly />
        </div>

        <div class="form-group">
          <label for="invDate">Invoice Date</label>
          <input type="date" id="invDate" bind:value={invoiceDate} required />
        </div>

        <div class="form-group">
          <label for="terms">Payment Terms</label>
          <select id="terms" bind:value={paymentTerms}>
            <option>COD / Immediate</option>
            <option>Net 15 Days</option>
            <option>Net 30 Days</option>
            <option>Net 60 Days</option>
          </select>
        </div>
      </div>

      <div class="form-group" style="margin-top: 1rem;">
        <label for="quotationSelect">Select Approved Quotation to Bill</label>
        <select id="quotationSelect" bind:value={selectedQuotationId}>
          {#each approvedQuotations as q}
            <option value={q.id}>{q.quotationNo} - {q.clientName} ({q.projectTitle}) - ₱{q.subtotal.toLocaleString()}</option>
          {/each}
        </select>
      </div>
    </div>

    <!-- 2. TAX & DISCOUNT CALCULATOR -->
    <div class="form-section">
      <h3>2. Tax, Discount, and Adjustments</h3>
      <div class="grid-2">
        <div class="form-group">
          <label for="discount">Discount Rate (%)</label>
          <input type="number" id="discount" bind:value={discountRate} min="0" max="100" step="0.5" />
        </div>

        <div class="form-group">
          <label for="wTax">Withholding Tax / Creditable Tax Rate (%)</label>
          <select id="wTax" bind:value={withholdingTaxRate}>
            <option value={0}>0% - Exempt</option>
            <option value={1}>1% - Goods Purchase</option>
            <option value={2}>2% - Services / Construction Subcontractor</option>
            <option value={5}>5% - Real Property / Special Rate</option>
          </select>
        </div>
      </div>
    </div>

    <!-- 3. INVOICE COMPUTATION SUMMARY PANEL -->
    <div class="totals-panel">
      <div class="totals-summary">
        <div class="total-row"><span>Base Quotation Amount:</span> <strong>₱{baseAmount.toLocaleString()}</strong></div>
        {#if discountRate > 0}
          <div class="total-row text-discount"><span>Less Discount ({discountRate}%):</span> <strong>-₱{discountAmount.toLocaleString()}</strong></div>
        {/if}
        <div class="total-row"><span>Add 12% VAT:</span> <strong>+₱{vatAmount.toLocaleString()}</strong></div>
        {#if withholdingTaxRate > 0}
          <div class="total-row text-tax"><span>Less Withholding Tax ({withholdingTaxRate}%):</span> <strong>-₱{withholdingTaxAmount.toLocaleString()}</strong></div>
        {/if}
        <div class="total-row grand-total"><span>Total Payable Amount:</span> <strong>₱{grandTotalInvoice.toLocaleString()}</strong></div>
      </div>

      <div class="action-buttons">
        <button type="submit" class="btn-primary">Save & Issue Invoice</button>
      </div>
    </div>
  </form>
</section>

<style>
  .form-container { display: flex; flex-direction: column; gap: 1.5rem; }
  .form-section { background: #0f172a; padding: 1.2rem; border-radius: 8px; border: 1px solid #334155; }
  .form-section h3 { margin-top: 0; color: #38bdf8; font-size: 1rem; border-bottom: 1px solid #1e293b; padding-bottom: 0.5rem; }
  
  .grid-2 { display: grid; grid-template-columns: 1fr 1fr; gap: 1rem; }
  .grid-3 { display: grid; grid-template-columns: 1fr 1fr 1fr; gap: 1rem; }
  
  .highlight-input { color: #38bdf8; border-color: #0284c7; font-size: 1.1rem; }

  .totals-panel { display: flex; justify-content: space-between; align-items: flex-end; background: #1e293b; padding: 1.2rem; border-radius: 8px; border: 1px solid #334155; }
  .totals-summary { display: flex; flex-direction: column; gap: 0.5rem; min-width: 320px; }
  .total-row { display: flex; justify-content: space-between; color: #cbd5e1; font-size: 0.95rem; }
  
  .text-discount { color: #f87171; }
  .text-tax { color: #fbbf24; }
  
  .grand-total { font-size: 1.3rem; color: #4ade80; border-top: 1px solid #334155; padding-top: 0.5rem; margin-top: 0.2rem; }
</style>