<script lang="ts">
  type Client = {
    id: number;
    name: string;
    contactPerson: string;
    email: string;
    phone: string;
    address: string;
    totalOrders: number;
  };

  // --- Initial Client State ---
  let clients = $state<Client[]>([
    { id: 101, name: "Cebu Construction Corp", contactPerson: "Juan Dela Cruz", email: "juan@cebuconst.ph", phone: "09171234567", address: "Mandaue City, Cebu", totalOrders: 5 },
    { id: 102, name: "Visayas Builders Inc", contactPerson: "Maria Santos", email: "maria@visayasbuilders.com", phone: "09189876543", address: "IT Park, Cebu City", totalOrders: 3 }
  ]);

  // --- Form & Modal State ---
  let searchQuery = $state('');
  let isAddModalOpen = $state(false);
  let isEditModalOpen = $state(false);

  // New Client Inputs
  let newName = $state('');
  let newContactPerson = $state('');
  let newEmail = $state('');
  let newPhone = $state('');
  let newAddress = $state('');

  // Edit Client State
  let editId = $state<number | null>(null);
  let editName = $state('');
  let editContactPerson = $state('');
  let editEmail = $state('');
  let editPhone = $state('');
  let editAddress = $state('');

  // --- Filtered Clients Derived State ---
  let filteredClients = $derived(
    clients.filter(c => 
      c.name.toLowerCase().includes(searchQuery.toLowerCase()) ||
      c.contactPerson.toLowerCase().includes(searchQuery.toLowerCase()) ||
      c.email.toLowerCase().includes(searchQuery.toLowerCase())
    )
  );

  // --- Actions ---
  function handleAddClient(e: Event) {
    e.preventDefault();
    if (!newName.trim()) return;

    const newClient: Client = {
      id: Date.now(),
      name: newName,
      contactPerson: newContactPerson,
      email: newEmail,
      phone: newPhone,
      address: newAddress,
      totalOrders: 0
    };

    clients = [newClient, ...clients];
    closeAddModal();
  }

  function openEditModal(client: Client) {
    editId = client.id;
    editName = client.name;
    editContactPerson = client.contactPerson;
    editEmail = client.email;
    editPhone = client.phone;
    editAddress = client.address;
    isEditModalOpen = true;
  }

  function handleSaveEdit(e: Event) {
    e.preventDefault();
    if (editId === null) return;

    clients = clients.map(c => c.id === editId ? {
      ...c,
      name: editName,
      contactPerson: editContactPerson,
      email: editEmail,
      phone: editPhone,
      address: editAddress
    } : c);

    closeEditModal();
  }

  function handleDeleteClient(id: number) {
    if (confirm("Are you sure you want to delete this client record?")) {
      clients = clients.filter(c => c.id !== id);
    }
  }

  function closeAddModal() {
    isAddModalOpen = false;
    newName = ''; newContactPerson = ''; newEmail = ''; newPhone = ''; newAddress = '';
  }

  function closeEditModal() {
    isEditModalOpen = false;
    editId = null;
  }
</script>

<section class="card">
  <div class="card-header-flex">
    <div>
      <h2>👥 Client Management</h2>
      <p class="subtitle">View, add, and manage company client accounts</p>
    </div>
    <button class="btn-primary" onclick={() => isAddModalOpen = true}>+ Add New Client</button>
  </div>

  <!-- Search Filter -->
  <div class="filter-bar">
    <input 
      type="text" 
      bind:value={searchQuery} 
      placeholder="🔍 Search clients by company, contact person, or email..." 
    />
  </div>

  <!-- Clients Table -->
  <div class="table-wrapper">
    <table class="data-table">
      <thead>
        <tr>
          <th>Company / Client Name</th>
          <th>Contact Person</th>
          <th>Contact Details</th>
          <th>Billing Address</th>
          <th style="text-align: center;">Orders</th>
          <th style="text-align: center;">Actions</th>
        </tr>
      </thead>
      <tbody>
        {#if filteredClients.length === 0}
          <tr>
            <td colspan="6" class="empty-cell">No matching client records found.</td>
          </tr>
        {:else}
          {#each filteredClients as client}
            <tr>
              <td>
                <span class="font-bold text-white">{client.name}</span>
              </td>
              <td>{client.contactPerson}</td>
              <td>
                <div class="contact-stack">
                  <span class="text-sm">📧 {client.email}</span>
                  <span class="text-sm text-dim">📞 {client.phone}</span>
                </div>
              </td>
              <td class="text-dim">{client.address}</td>
              <td style="text-align: center;">
                <span class="badge-count">{client.totalOrders}</span>
              </td>
              <td style="text-align: center;">
                <div class="action-btn-group">
                  <button class="btn-sm-edit" onclick={() => openEditModal(client)}>Edit</button>
                  <button class="btn-sm-danger" onclick={() => handleDeleteClient(client.id)}>Delete</button>
                </div>
              </td>
            </tr>
          {/each}
        {/if}
      </tbody>
    </table>
  </div>
</section>

<!-- ADD CLIENT MODAL -->
{#if isAddModalOpen}
  <div class="modal-backdrop" onclick={closeAddModal} aria-hidden="true"></div>
  <div class="modal">
    <div class="modal-header">
      <h3>Register New Client</h3>
      <button class="close-btn" onclick={closeAddModal}>✕</button>
    </div>
    <form onsubmit={handleAddClient} class="modal-form">
      <div class="form-group">
        <label for="cName">Company / Client Name</label>
        <input type="text" id="cName" bind:value={newName} required placeholder="e.g. Metro Builders Inc." />
      </div>
      <div class="grid-2">
        <div class="form-group">
          <label for="cPerson">Contact Person</label>
          <input type="text" id="cPerson" bind:value={newContactPerson} required placeholder="e.g. John Doe" />
        </div>
        <div class="form-group">
          <label for="cPhone">Phone Number</label>
          <input type="text" id="cPhone" bind:value={newPhone} required placeholder="0917XXXXXXX" />
        </div>
      </div>
      <div class="form-group">
        <label for="cEmail">Email Address</label>
        <input type="email" id="cEmail" bind:value={newEmail} required placeholder="client@company.ph" />
      </div>
      <div class="form-group">
        <label for="cAddress">Billing Address</label>
        <input type="text" id="cAddress" bind:value={newAddress} required placeholder="City, Province" />
      </div>
      <div class="modal-footer">
        <button type="button" class="btn-secondary" onclick={closeAddModal}>Cancel</button>
        <button type="submit" class="btn-primary">Save Client Profile</button>
      </div>
    </form>
  </div>
{/if}

<!-- EDIT CLIENT MODAL -->
{#if isEditModalOpen}
  <div class="modal-backdrop" onclick={closeEditModal} aria-hidden="true"></div>
  <div class="modal">
    <div class="modal-header">
      <h3>Edit Client Information</h3>
      <button class="close-btn" onclick={closeEditModal}>✕</button>
    </div>
    <form onsubmit={handleSaveEdit} class="modal-form">
      <div class="form-group">
        <label for="editCName">Company / Client Name</label>
        <input type="text" id="editCName" bind:value={editName} required />
      </div>
      <div class="grid-2">
        <div class="form-group">
          <label for="editCPerson">Contact Person</label>
          <input type="text" id="editCPerson" bind:value={editContactPerson} required />
        </div>
        <div class="form-group">
          <label for="editCPhone">Phone Number</label>
          <input type="text" id="editCPhone" bind:value={editPhone} required />
        </div>
      </div>
      <div class="form-group">
        <label for="editCEmail">Email Address</label>
        <input type="email" id="editCEmail" bind:value={editEmail} required />
      </div>
      <div class="form-group">
        <label for="editCAddress">Billing Address</label>
        <input type="text" id="editCAddress" bind:value={editAddress} required />
      </div>
      <div class="modal-footer">
        <button type="button" class="btn-secondary" onclick={closeEditModal}>Cancel</button>
        <button type="submit" class="btn-primary">Update Client</button>
      </div>
    </form>
  </div>
{/if}

<style>
  .card-header-flex { display: flex; justify-content: space-between; align-items: center; border-bottom: 1px solid #334155; padding-bottom: 1rem; margin-bottom: 1rem; }
  .subtitle { margin: 0.2rem 0 0 0; font-size: 0.85rem; color: #94a3b8; }
  .filter-bar { margin-bottom: 1rem; }
  
  .table-wrapper { overflow-x: auto; border: 1px solid #334155; border-radius: 8px; }
  .data-table { width: 100%; border-collapse: collapse; text-align: left; font-size: 0.9rem; }
  .data-table th { background: #0f172a; color: #94a3b8; padding: 0.8rem 1rem; border-bottom: 1px solid #334155; }
  .data-table td { padding: 0.8rem 1rem; border-bottom: 1px solid #1e293b; vertical-align: middle; }
  .empty-cell { text-align: center; color: #64748b; padding: 2rem; }

  .contact-stack { display: flex; flex-direction: column; gap: 0.2rem; }
  .text-sm { font-size: 0.85rem; }
  .text-dim { color: #94a3b8; }
  .font-bold { font-weight: bold; }
  .text-white { color: #f8fafc; }

  .badge-count { background: #334155; color: #38bdf8; font-size: 0.8rem; padding: 0.2rem 0.6rem; border-radius: 12px; font-weight: bold; }
  
  .action-btn-group { display: flex; justify-content: center; gap: 0.4rem; }
  .btn-sm-edit { background: #d97706; color: white; border: none; padding: 0.3rem 0.6rem; border-radius: 4px; cursor: pointer; font-size: 0.8rem; }
  .btn-sm-danger { background: #dc2626; color: white; border: none; padding: 0.3rem 0.6rem; border-radius: 4px; cursor: pointer; font-size: 0.8rem; }

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