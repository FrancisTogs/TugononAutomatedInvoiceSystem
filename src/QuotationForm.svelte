<script lang="ts">
  // --- Data Types ---
  type Client = { id: number; name: string; contactPerson: string; email: string; phone: string; address: string };
  type LineItem = { id: number; description: string; qty: number; unitCost: number };

  // --- Mock Client Database for Lookup ---
  let clientsList: Client[] = [
    { id: 101, name: "Cebu Construction Corp", contactPerson: "Juan Dela Cruz", email: "juan@cebuconst.ph", phone: "09171234567", address: "Mandaue City, Cebu" },
    { id: 102, name: "Visayas Builders Inc", contactPerson: "Maria Santos", email: "maria@visayasbuilders.com", phone: "09189876543", address: "IT Park, Cebu City" }
  ];

  // --- Form State ---
  let selectedClientId = $state<number | null>(null);
  let searchQuery = $state('');
  
  let projectTitle = $state('');
  let siteLocation = $state('');
  let targetStartDate = $state('');
  let targetEndDate = $state('');

  let lineItems = $state<LineItem[]>([
    { id: Date.now(), description: 'Foundation & Concrete Pouring', qty: 1, unitCost: 150000 }
  ]);

  // --- Client Lookup Derived State ---
  let selectedClient = $derived(clientsList.find(c => c.id === selectedClientId) || null);

  // --- Calculations (Svelte 5 $derived) ---
  let subtotal = $derived(lineItems.reduce((acc, item) => acc + (item.qty * item.unitCost), 0));
  let estimatedTax = $derived(subtotal * 0.12); // 12% VAT estimate
  let grandTotal = $derived(subtotal + estimatedTax);

  // --- Actions ---
  function addLineItem() {
    lineItems = [...lineItems, { id: Date.now(), description: '', qty: 1, unitCost: 0 }];
  }

  function removeLineItem(id: number) {
    if (lineItems.length > 1) {
      lineItems = lineItems.filter(item => item.id !== id);
    }
  }

  function handleSubmitQuotation(event: Event) {
    event.preventDefault();
    if (!selectedClientId) return alert("Please select or lookup a client!");
    
    const quotationPayload = {
      clientId: selectedClientId,
      projectTitle,
      siteLocation,
      targetStartDate,
      targetEndDate,
      items: lineItems,
      subtotal,
      estimatedTax,
      grandTotal,
      status: 'PENDING_APPROVAL'
    };

    console.log("Submitting Quotation:", quotationPayload);
    alert(`Quotation for "${projectTitle}" generated successfully! Sent for Finance review.`);
  }
</script>

<section class="card">
  <div class="card-header">
    <h2>📋 Create Order & Project Quotation</h2>
  </div>

  <form onsubmit={handleSubmitQuotation} class="form-container">
    <!-- 1. CLIENT LOOKUP & INFORMATION -->
    <div class="form-section">
      <h3>1. Client Information</h3>
      <div class="grid-2">
        <div class="form-group">
          <label for="clientSelect">Client Lookup</label>
          <select id="clientSelect" bind:value={selectedClientId}>
            <option value={null}>-- Select Registered Client --</option>
            {#each clientsList as client}
              <option value={client.id}>{client.name} ({client.contactPerson})</option>
            {/each}
          </select>
        </div>

        <div class="form-group">
          <label for="contactPerson">Contact Person</label>
          <input type="text" id="contactPerson" value={selectedClient?.contactPerson || ''} readonly placeholder="Auto-filled" />
        </div>
      </div>

      <div class="grid-3" style="margin-top: 0.8rem;">
        <div class="form-group">
          <label for="clientEmail">Email</label>
          <input type="text" id="clientEmail" value={selectedClient?.email || ''} readonly placeholder="Auto-filled" />
        </div>
        <div class="form-group">
          <label for="clientPhone">Phone</label>
          <input type="text" id="clientPhone" value={selectedClient?.phone || ''} readonly placeholder="Auto-filled" />
        </div>
        <div class="form-group">
          <label for="billingAddress">Billing Address</label>
          <input type="text" id="billingAddress" value={selectedClient?.address || ''} readonly placeholder="Auto-filled" />
        </div>
      </div>
    </div>

    <!-- 2. PROJECT DETAILS -->
    <div class="form-section">
      <h3>2. Project Details</h3>
      <div class="grid-2">
        <div class="form-group">
          <label for="projTitle">Project Title</label>
          <input type="text" id="projTitle" bind:value={projectTitle} placeholder="e.g. Commercial Warehouse Phase 1" required />
        </div>
        <div class="form-group">
          <label for="siteLoc">Site Location</label>
          <input type="text" id="siteLoc" bind:value={siteLocation} placeholder="e.g. Subangdaku, Mandaue City" required />
        </div>
      </div>

      <div class="grid-2" style="margin-top: 0.8rem;">
        <div class="form-group">
          <label for="startDate">Target Start Date</label>
          <input type="date" id="startDate" bind:value={targetStartDate} required />
        </div>
        <div class="form-group">
          <label for="endDate">Target End Date / Completion</label>
          <input type="date" id="endDate" bind:value={targetEndDate} required />
        </div>
      </div>
    </div>

    <!-- 3. DYNAMIC QUOTATION ITEMS CALCULATOR -->
    <div class="form-section">
      <div class="section-header-flex">
        <h3>3. Quotation Items & Scope</h3>
        <button type="button" class="btn-secondary" onclick={addLineItem}>+ Add Line Item</button>
      </div>

      <table class="data-table">
        <thead>
          <tr>
            <th>Description / Scope</th>
            <th style="width: 100px;">Qty</th>
            <th style="width: 160px;">Unit Cost (PHP)</th>
            <th style="width: 160px;">Line Total (PHP)</th>
            <th style="width: 50px;"></th>
          </tr>
        </thead>
        <tbody>
          {#each lineItems as item, index}
            <tr>
              <td>
                <input type="text" bind:value={item.description} placeholder="Item description or service..." required />
              </td>
              <td>
                <input type="number" bind:value={item.qty} min="1" required />
              </td>
              <td>
                <input type="number" bind:value={item.unitCost} min="0" step="100" required />
              </td>
              <td class="text-right font-bold">
                ₱{(item.qty * item.unitCost).toLocaleString()}
              </td>
              <td class="text-center">
                {#if lineItems.length > 1}
                  <button type="button" class="btn-icon-danger" onclick={() => removeLineItem(item.id)}>✕</button>
                {/if}
              </td>
            </tr>
          {/each}
        </tbody>
      </table>
    </div>

    <!-- 4. TOTALS & SUBMISSION -->
    <div class="totals-panel">
      <div class="totals-summary">
        <div class="total-row"><span>Subtotal:</span> <strong>₱{subtotal.toLocaleString()}</strong></div>
        <div class="total-row"><span>Est. Tax (12% VAT):</span> <strong>₱{estimatedTax.toLocaleString()}</strong></div>
        <div class="total-row grand-total"><span>Grand Total:</span> <strong>₱{grandTotal.toLocaleString()}</strong></div>
      </div>
      
      <div class="action-buttons">
        <button type="submit" class="btn-primary">Generate & Save Quotation</button>
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
  
  .section-header-flex { display: flex; justify-content: space-between; align-items: center; margin-bottom: 0.8rem; }
  
  .data-table { width: 100%; border-collapse: collapse; margin-top: 0.5rem; }
  .data-table th { background: #1e293b; color: #94a3b8; padding: 0.6rem; text-align: left; font-size: 0.85rem; border: 1px solid #334155; }
  .data-table td { padding: 0.5rem; border: 1px solid #334155; vertical-align: middle; }
  
  .text-right { text-align: right; }
  .text-center { text-align: center; }
  .font-bold { font-weight: bold; color: #f8fafc; }

  .totals-panel { display: flex; justify-content: space-between; align-items: flex-end; background: #1e293b; padding: 1.2rem; border-radius: 8px; border: 1px solid #334155; }
  .totals-summary { display: flex; flex-direction: column; gap: 0.4rem; min-width: 280px; }
  .total-row { display: flex; justify-content: space-between; color: #cbd5e1; font-size: 0.95rem; }
  .grand-total { font-size: 1.2rem; color: #4ade80; border-top: 1px solid #334155; padding-top: 0.4rem; margin-top: 0.2rem; }

  .btn-secondary { background: #334155; color: white; border: none; padding: 0.5rem 1rem; border-radius: 6px; cursor: pointer; font-weight: 600; }
  .btn-secondary:hover { background: #475569; }
  .btn-icon-danger { background: transparent; border: none; color: #f87171; font-size: 1.1rem; cursor: pointer; }
</style>