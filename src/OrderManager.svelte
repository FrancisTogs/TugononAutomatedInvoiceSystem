<script lang="ts">
  type Order = {
    id: string;
    clientName: string;
    projectTitle: string;
    siteLocation: string;
    estimatedCost: number;
    requestDate: string;
    status: 'NEW' | 'QUOTATION_CREATED' | 'IN_PROGRESS' | 'COMPLETED';
  };

  // --- Initial Orders State ---
  let orders = $state<Order[]>([
    { id: "ORD-2026-001", clientName: "Cebu Construction Corp", projectTitle: "Commercial Warehouse Phase 1", siteLocation: "Subangdaku, Mandaue City", estimatedCost: 150000, requestDate: "2026-09-15", status: "QUOTATION_CREATED" },
    { id: "ORD-2026-002", clientName: "Visayas Builders Inc", projectTitle: "Site Excavation & Clearing", siteLocation: "IT Park, Cebu City", estimatedCost: 450000, requestDate: "2026-09-20", status: "NEW" },
    { id: "ORD-2026-003", clientName: "Metro Cebu Holdings", projectTitle: "Drainage System Installation", siteLocation: "Banilad, Cebu City", estimatedCost: 280000, requestDate: "2026-09-22", status: "IN_PROGRESS" }
  ]);

  let statusFilter = $state('ALL');
  let isNewOrderModalOpen = $state(false);

  // New Order Form Inputs
  let clientName = $state('');
  let projectTitle = $state('');
  let siteLocation = $state('');
  let estimatedCost = $state(0);

  // --- Derived Filtered List ---
  let filteredOrders = $derived(
    orders.filter(o => statusFilter === 'ALL' || o.status === statusFilter)
  );

  function handleCreateOrder(e: Event) {
    e.preventDefault();
    if (!clientName.trim() || !projectTitle.trim()) return;

    const newOrder: Order = {
      id: `ORD-2026-00${orders.length + 1}`,
      clientName,
      projectTitle,
      siteLocation,
      estimatedCost,
      requestDate: new Date().toISOString().split('T')[0],
      status: 'NEW'
    };

    orders = [newOrder, ...orders];
    closeModal();
  }

  function updateOrderStatus(id: string, newStatus: Order['status']) {
    orders = orders.map(o => o.id === id ? { ...o, status: newStatus } : o);
  }

  function closeModal() {
    isNewOrderModalOpen = false;
    clientName = ''; projectTitle = ''; siteLocation = ''; estimatedCost = 0;
  }
</script>

<section class="card">
  <div class="card-header-flex">
    <div>
      <h2>📋 Client Orders & Service Requests</h2>
      <p class="subtitle">Track incoming construction requests and project proposals</p>
    </div>
    <button class="btn-primary" onclick={() => isNewOrderModalOpen = true}>+ Create New Order Request</button>
  </div>

  <!-- Status Filter Tabs -->
  <div class="filter-tabs">
    <button class="filter-btn {statusFilter === 'ALL' ? 'active' : ''}" onclick={() => statusFilter = 'ALL'}>All Orders ({orders.length})</button>
    <button class="filter-btn {statusFilter === 'NEW' ? 'active' : ''}" onclick={() => statusFilter = 'NEW'}>New Requests</button>
    <button class="filter-btn {statusFilter === 'QUOTATION_CREATED' ? 'active' : ''}" onclick={() => statusFilter = 'QUOTATION_CREATED'}>Quotation Created</button>
    <button class="filter-btn {statusFilter === 'IN_PROGRESS' ? 'active' : ''}" onclick={() => statusFilter = 'IN_PROGRESS'}>In Progress</button>
  </div>

  <!-- Orders Table -->
  <div class="table-wrapper">
    <table class="data-table">
      <thead>
        <tr>
          <th>Order ID</th>
          <th>Client Name</th>
          <th>Project Title & Location</th>
          <th>Est. Budget</th>
          <th>Date Requested</th>
          <th>Status</th>
          <th style="text-align: center;">Change Status</th>
        </tr>
      </thead>
      <tbody>
        {#if filteredOrders.length === 0}
          <tr>
            <td colspan="7" class="empty-cell">No orders found for the selected filter.</td>
          </tr>
        {:else}
          {#each filteredOrders as order}
            <tr>
              <td class="font-bold text-blue">{order.id}</td>
              <td class="font-bold text-white">{order.clientName}</td>
              <td>
                <div class="project-stack">
                  <span>{order.projectTitle}</span>
                  <span class="text-dim text-sm">📍 {order.siteLocation}</span>
                </div>
              </td>
              <td>₱{order.estimatedCost.toLocaleString()}</td>
              <td class="text-dim">{order.requestDate}</td>
              <td>
                <span class="status-badge status-{order.status.toLowerCase()}">
                  {order.status.replace('_', ' ')}
                </span>
              </td>
              <td style="text-align: center;">
                <select 
                  class="status-select" 
                  value={order.status} 
                  onchange={(e) => updateOrderStatus(order.id, (e.target as HTMLSelectElement).value as Order['status'])}
                >
                  <option value="NEW">NEW</option>
                  <option value="QUOTATION_CREATED">QUOTATION CREATED</option>
                  <option value="IN_PROGRESS">IN PROGRESS</option>
                  <option value="COMPLETED">COMPLETED</option>
                </select>
              </td>
            </tr>
          {/each}
        {/if}
      </tbody>
    </table>
  </div>
</section>

<!-- CREATE ORDER MODAL -->
{#if isNewOrderModalOpen}
  <div class="modal-backdrop" onclick={closeModal} aria-hidden="true"></div>
  <div class="modal">
    <div class="modal-header">
      <h3>Record New Order Request</h3>
      <button class="close-btn" onclick={closeModal}>✕</button>
    </div>
    <form onsubmit={handleCreateOrder} class="modal-form">
      <div class="form-group">
        <label for="oClient">Client / Company Name</label>
        <input type="text" id="oClient" bind:value={clientName} required placeholder="e.g. Cebu Construction Corp" />
      </div>
      <div class="form-group">
        <label for="oTitle">Project Title</label>
        <input type="text" id="oTitle" bind:value={projectTitle} required placeholder="e.g. Warehouse Concrete Slab Construction" />
      </div>
      <div class="grid-2">
        <div class="form-group">
          <label for="oLoc">Site Location</label>
          <input type="text" id="oLoc" bind:value={siteLocation} required placeholder="Mandaue City, Cebu" />
        </div>
        <div class="form-group">
          <label for="oCost">Estimated Budget (PHP)</label>
          <input type="number" id="oCost" bind:value={estimatedCost} min="0" step="5000" required />
        </div>
      </div>
      <div class="modal-footer">
        <button type="button" class="btn-secondary" onclick={closeModal}>Cancel</button>
        <button type="submit" class="btn-primary">Create Order</button>
      </div>
    </form>
  </div>
{/if}

<style>
  .card-header-flex { display: flex; justify-content: space-between; align-items: center; border-bottom: 1px solid #334155; padding-bottom: 1rem; margin-bottom: 1rem; }
  .subtitle { margin: 0.2rem 0 0 0; font-size: 0.85rem; color: #94a3b8; }
  
  .filter-tabs { display: flex; gap: 0.5rem; margin-bottom: 1rem; overflow-x: auto; }
  .filter-btn { background: #0f172a; border: 1px solid #334155; color: #94a3b8; padding: 0.4rem 0.8rem; border-radius: 6px; font-size: 0.85rem; cursor: pointer; }
  .filter-btn.active { background: #2563eb; color: white; border-color: #3b82f6; }

  .table-wrapper { overflow-x: auto; border: 1px solid #334155; border-radius: 8px; }
  .data-table { width: 100%; border-collapse: collapse; text-align: left; font-size: 0.9rem; }
  .data-table th { background: #0f172a; color: #94a3b8; padding: 0.8rem 1rem; border-bottom: 1px solid #334155; }
  .data-table td { padding: 0.8rem 1rem; border-bottom: 1px solid #1e293b; vertical-align: middle; }
  .empty-cell { text-align: center; color: #64748b; padding: 2rem; }

  .project-stack { display: flex; flex-direction: column; gap: 0.2rem; }
  .text-sm { font-size: 0.8rem; }
  .text-dim { color: #94a3b8; }
  .font-bold { font-weight: bold; }
  .text-white { color: #f8fafc; }
  .text-blue { color: #38bdf8; }

  .status-badge { font-size: 0.75rem; font-weight: bold; padding: 0.2rem 0.6rem; border-radius: 12px; display: inline-block; text-transform: uppercase; }
  .status-new { background: #1e3a8a; color: #93c5fd; }
  .status-quotation_created { background: #78350f; color: #fde047; }
  .status-in_progress { background: #065f46; color: #6ee7b7; }
  .status-completed { background: #14532d; color: #86efac; }

  .status-select { background: #0f172a; color: #e2e8f0; border: 1px solid #334155; padding: 0.3rem 0.5rem; border-radius: 4px; font-size: 0.8rem; cursor: pointer; }

  /* Modal Styling */
  .modal-backdrop { position: fixed; top: 0; left: 0; width: 100%; height: 100%; background: rgba(0,0,0,0.6); z-index: 1000; }
  .modal { position: fixed; top: 50%; left: 50%; transform: translate(-50%, -50%); background: #1e293b; border: 1px solid #334155; padding: 1.5rem; border-radius: 12px; z-index: 1001; width: 90%; max-width: 500px; box-shadow: 0 10px 25px rgba(0,0,0,0.5); }
  .modal-header { display: flex; justify-content: space-between; align-items: center; border-bottom: 1px solid #334155; padding-bottom: 0.8rem; margin-bottom: 1rem; }
  .modal-header h3 { margin: 0; color: #f8fafc; }
  .close-btn { background: none; border: none; color: #94a3b8; font-size: 1.2rem; cursor: pointer; }
  
  .modal-form { display: flex; flex-direction: column; gap: 1rem; }
  .grid-2 { display: grid; grid-template-columns: 1fr 1fr; gap: 0.8rem; }
  .form-group { display: flex; flex-direction: column; gap: 0.3rem; }
  .form-group label { font-size: 0.8rem; color: #cbd5e1; font-weight: 600; }
  
  .modal-footer { display: flex; justify-content: flex-end; gap: 0.8rem; margin-top: 1rem; border-top: 1px solid #334155; padding-top: 1rem; }
  .btn-primary { background: #2563eb; color: white; border: none; padding: 0.6rem 1.2rem; border-radius: 6px; font-weight: bold; cursor: pointer; }
  .btn-secondary { background: #334155; color: white; border: none; padding: 0.6rem 1rem; border-radius: 6px; cursor: pointer; }
</style>