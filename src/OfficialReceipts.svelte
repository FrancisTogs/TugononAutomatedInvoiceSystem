<script lang="ts">
  type Receipt = {
    receiptNo: string;
    clientName: string;
    linkedInvoiceNo: string;
    amountPaid: number;
    amountInWords: string;
    paymentMethod: string;
    referenceNo: string;
    paymentDate: string;
    issuedBy: string;
  };

  // --- Initial Official Receipts Database ---
  let receipts = $state<Receipt[]>([
    {
      receiptNo: 'OR-2026-001',
      clientName: 'Cebu Construction Corp',
      linkedInvoiceNo: 'INV-2026-015',
      amountPaid: 168000,
      amountInWords: 'One Hundred Sixty-Eight Thousand Pesos Only',
      paymentMethod: 'Bank Transfer',
      referenceNo: 'TRN-9988231',
      paymentDate: '2026-09-18',
      issuedBy: 'Francis Virgilio Tugonon'
    },
    {
      receiptNo: 'OR-2026-002',
      clientName: 'Visayas Builders Inc',
      linkedInvoiceNo: 'INV-2026-016',
      amountPaid: 250000,
      amountInWords: 'Two Hundred Fifty Thousand Pesos Only',
      paymentMethod: 'Check',
      referenceNo: 'CHK-004412',
      paymentDate: '2026-09-22',
      issuedBy: 'Francis Virgilio Tugonon'
    }
  ]);

  let activeReceipt = $state<Receipt>(receipts[0]);
  let isHistoricModalOpen = $state(false);

  function selectReceipt(receipt: Receipt) {
    activeReceipt = receipt;
    isHistoricModalOpen = false;
  }

  function handlePrintReceipt() {
    window.print();
  }
</script>

<section class="card">
  <div class="card-header-flex">
    <div>
      <h2>🧾 Official Receipts Issuance & Preview</h2>
      <p class="subtitle">Official evidence of completed client service payments</p>
    </div>
    <button class="btn-secondary" onclick={() => isHistoricModalOpen = true}>
      📚 View Historic Receipts ({receipts.length})
    </button>
  </div>

  <!-- OFFICIAL RECEIPT CERTIFICATE DISPLAY FRAME -->
  <div class="receipt-frame-wrapper">
    <div class="official-receipt-card">
      <div class="or-watermark">OFFICIAL RECEIPT</div>

      <!-- OR Header -->
      <div class="or-header">
        <div class="or-brand">
          <span class="or-logo">🏗️</span>
          <div>
            <h1 class="or-company-name">Tugonon Construction Services</h1>
            <p class="or-subtext">Automated Billing & Transaction System | Cebu City</p>
          </div>
        </div>
        <div class="or-meta">
          <div class="or-number font-bold text-gold">{activeReceipt.receiptNo}</div>
          <div class="or-date">Date: <strong>{activeReceipt.paymentDate}</strong></div>
        </div>
      </div>

      <hr class="or-divider" />

      <!-- OR Body Content -->
      <div class="or-body">
        <div class="or-row">
          <span class="or-label">Received From Client:</span>
          <span class="or-value font-bold text-white">{activeReceipt.clientName}</span>
        </div>

        <div class="or-row">
          <span class="or-label">Linked Invoice Reference:</span>
          <span class="or-value text-blue">{activeReceipt.linkedInvoiceNo}</span>
        </div>

        <div class="or-row">
          <span class="or-label">Amount Received (in Words):</span>
          <span class="or-value text-words font-bold">{activeReceipt.amountInWords}</span>
        </div>

        <div class="or-row-highlight">
          <span class="or-label">Amount Received (in Figures):</span>
          <span class="or-amount-fig">₱{activeReceipt.amountPaid.toLocaleString()}</span>
        </div>

        <div class="grid-2 or-details-grid">
          <div>
            <span class="or-label block">Payment Details:</span>
            <ul class="or-list">
              <li>Method: <strong>{activeReceipt.paymentMethod}</strong></li>
              <li>Reference No: <strong>{activeReceipt.referenceNo}</strong></li>
            </ul>
          </div>
          
          <div class="signature-box">
            <div class="signature-line"></div>
            <span class="or-label block">Authorized Issued By:</span>
            <span class="font-bold text-white">{activeReceipt.issuedBy} (Finance Officer)</span>
          </div>
        </div>
      </div>
    </div>
  </div>

  <!-- Action Controls -->
  <div class="receipt-actions">
    <button class="btn-primary" onclick={handlePrintReceipt}>🖨️ Print / Download PDF Receipt</button>
  </div>
</section>

<!-- HISTORIC RECEIPTS LIST MODAL -->
{#if isHistoricModalOpen}
  <div class="modal-backdrop" onclick={() => isHistoricModalOpen = false} aria-hidden="true"></div>
  <div class="modal">
    <div class="modal-header">
      <h3>Historic Official Receipts</h3>
      <button class="close-btn" onclick={() => isHistoricModalOpen = false}>✕</button>
    </div>

    <div class="table-wrapper">
      <table class="data-table">
        <thead>
          <tr>
            <th>Receipt No</th>
            <th>Client Name</th>
            <th>Linked Invoice</th>
            <th>Amount Paid</th>
            <th>Payment Date</th>
            <th style="text-align: center;">Action</th>
          </tr>
        </thead>
        <tbody>
          {#each receipts as r}
            <tr>
              <td class="font-bold text-gold">{r.receiptNo}</td>
              <td class="font-bold text-white">{r.clientName}</td>
              <td class="text-blue">{r.linkedInvoiceNo}</td>
              <td class="text-green font-bold">₱{r.amountPaid.toLocaleString()}</td>
              <td>{r.paymentDate}</td>
              <td style="text-align: center;">
                <button class="btn-sm-action" onclick={() => selectReceipt(r)}>Load Preview</button>
              </td>
            </tr>
          {/each}
        </tbody>
      </table>
    </div>
  </div>
{/if}

<style>
  .card-header-flex { display: flex; justify-content: space-between; align-items: center; border-bottom: 1px solid #334155; padding-bottom: 0.8rem; margin-bottom: 1.5rem; }
  .subtitle { margin: 0.2rem 0 0 0; font-size: 0.85rem; color: #94a3b8; }

  /* Receipt Certificate Stylings */
  .receipt-frame-wrapper { display: flex; justify-content: center; }
  .official-receipt-card { background: #0f172a; border: 2px solid #38bdf8; padding: 2rem; border-radius: 12px; width: 100%; max-width: 750px; position: relative; box-shadow: 0 10px 30px rgba(0,0,0,0.5); }
  
  .or-watermark { position: absolute; top: 50%; left: 50%; transform: translate(-50%, -50%) rotate(-20deg); font-size: 3.5rem; font-weight: 900; color: rgba(255, 255, 255, 0.03); pointer-events: none; white-space: nowrap; }

  .or-header { display: flex; justify-content: space-between; align-items: flex-start; }
  .or-brand { display: flex; align-items: center; gap: 0.8rem; }
  .or-logo { font-size: 2rem; }
  .or-company-name { font-size: 1.2rem; margin: 0; color: #f8fafc; }
  .or-subtext { font-size: 0.75rem; color: #94a3b8; margin: 0.2rem 0 0 0; }

  .or-meta { text-align: right; }
  .or-number { font-size: 1.1rem; }
  .or-date { font-size: 0.85rem; color: #cbd5e1; margin-top: 0.2rem; }

  .or-divider { border: 0; border-top: 1px dashed #334155; margin: 1.2rem 0; }

  .or-body { display: flex; flex-direction: column; gap: 1rem; }
  .or-row { display: flex; justify-content: space-between; font-size: 0.95rem; border-bottom: 1px dotted #1e293b; padding-bottom: 0.4rem; }
  .or-row-highlight { display: flex; justify-content: space-between; align-items: center; background: #1e293b; padding: 0.8rem 1rem; border-radius: 8px; border: 1px solid #334155; }

  .or-label { color: #94a3b8; font-size: 0.85rem; }
  .or-value { color: #e2e8f0; }
  .text-words { color: #fde047; font-style: italic; }
  .or-amount-fig { font-size: 1.4rem; font-weight: bold; color: #4ade80; }

  .or-details-grid { margin-top: 0.5rem; }
  .or-list { margin: 0.4rem 0 0 0; padding-left: 1.2rem; color: #cbd5e1; font-size: 0.85rem; }
  
  .signature-box { display: flex; flex-direction: column; align-items: flex-end; justify-content: flex-end; }
  .signature-line { width: 180px; border-bottom: 1px solid #94a3b8; margin-bottom: 0.4rem; }

  .receipt-actions { display: flex; justify-content: flex-end; margin-top: 1.5rem; }

  /* Utility Styles */
  .grid-2 { display: grid; grid-template-columns: 1fr 1fr; gap: 1rem; }
  .font-bold { font-weight: bold; }
  .text-white { color: #f8fafc; }
  .text-blue { color: #38bdf8; }
  .text-green { color: #4ade80; }
  .text-gold { color: #f59e0b; }
  .block { display: block; }

  .btn-primary { background: #2563eb; color: white; border: none; padding: 0.8rem 1.4rem; border-radius: 6px; font-weight: bold; cursor: pointer; }
  .btn-secondary { background: #334155; color: white; border: none; padding: 0.6rem 1rem; border-radius: 6px; cursor: pointer; font-size: 0.85rem; }
  .btn-sm-action { background: #0284c7; color: white; border: none; padding: 0.3rem 0.6rem; border-radius: 4px; font-size: 0.8rem; cursor: pointer; }

  /* Modal Styling */
  .modal-backdrop { position: fixed; top: 0; left: 0; width: 100%; height: 100%; background: rgba(0,0,0,0.6); z-index: 1000; }
  .modal { position: fixed; top: 50%; left: 50%; transform: translate(-50%, -50%); background: #1e293b; border: 1px solid #334155; padding: 1.5rem; border-radius: 12px; z-index: 1001; width: 90%; max-width: 700px; box-shadow: 0 10px 25px rgba(0,0,0,0.5); }
  .modal-header { display: flex; justify-content: space-between; align-items: center; border-bottom: 1px solid #334155; padding-bottom: 0.8rem; margin-bottom: 1rem; }
  .modal-header h3 { margin: 0; color: #f8fafc; }
  .close-btn { background: none; border: none; color: #94a3b8; font-size: 1.2rem; cursor: pointer; }

  .table-wrapper { overflow-x: auto; border: 1px solid #334155; border-radius: 8px; }
  .data-table { width: 100%; border-collapse: collapse; text-align: left; font-size: 0.85rem; }
  .data-table th { background: #0f172a; color: #94a3b8; padding: 0.7rem; border-bottom: 1px solid #334155; }
  .data-table td { padding: 0.7rem; border-bottom: 1px solid #1e293b; vertical-align: middle; }
</style>