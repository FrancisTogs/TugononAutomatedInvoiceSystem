<script lang="ts">

// 1. Import your new sub-components
  import ClientManager from './ClientManager.svelte';
  import OrderManager from './OrderManager.svelte';
  import QuotationForm from './QuotationForm.svelte';
  import InvoiceManager from './InvoiceManager.svelte';
  import PaymentStatus from './PaymentStatus.svelte';
  import OfficialReceipts from './OfficialReceipts.svelte';

  // --- Auth State ---
  type User = {
    id: number;
    name: string;
    email: string;
    role: 'ADMIN_SALES' | 'FINANCE';
  };

  let currentUser = $state<User | null>(null);
  let usernameInput = $state('Akeah Diez'); // Default pre-fill for fast testing
  let passwordInput = $state('password123');
  let selectedRoleInput = $state<'ADMIN_SALES' | 'FINANCE'>('ADMIN_SALES');
  let authError = $state('');

  // --- Active Tab State ---
  let activeTab = $state('dashboard');

  // --- Mock Login Action ---
  function handleLogin(e: Event) {
    e.preventDefault();
    authError = '';

    if (!usernameInput.trim() || !passwordInput.trim()) {
      authError = 'Please enter both username and password.';
      return;
    }

    // Simulating authentication response based on selected testing role
    currentUser = {
      id: 1,
      name: usernameInput,
      email: selectedRoleInput === 'ADMIN_SALES' ? 'admin@tugonon.com' : 'finance@tugonon.com',
      role: selectedRoleInput
    };

    activeTab = 'dashboard';
  }

  function handleLogout() {
    currentUser = null;
    usernameInput = '';
    passwordInput = '';
    authError = '';
  }
</script>

<main class="app-root">
  {#if !currentUser}
    <!-- ================= LOGIN SCREEN ================= -->
    <div class="login-wrapper">
      <div class="login-card">
        <div class="brand-header">
          <div class="logo-icon">🏗️</div>
          <h2>Tugonon Construction Services</h2>
          <p class="subtitle">Automated Invoicing & Transaction System</p>
        </div>

        {#if authError}
          <div class="alert-error">{authError}</div>
        {/if}

        <form onsubmit={handleLogin} class="login-form">
          <div class="form-group">
            <label for="username">Username / Email</label>
            <input 
              type="text" 
              id="username" 
              bind:value={usernameInput} 
              placeholder="Enter username" 
              required 
            />
          </div>

          <div class="form-group">
            <label for="password">Password</label>
            <input 
              type="password" 
              id="password" 
              bind:value={passwordInput} 
              placeholder="••••••••" 
              required 
            />
          </div>

          <!-- Role Selector for testing -->
          <div class="form-group">
            <label for="role">Select Role (Testing Switch)</label>
            <select id="role" bind:value={selectedRoleInput}>
              <option value="ADMIN_SALES">Admin / Sales Officer</option>
              <option value="FINANCE">Finance / Accounting Officer</option>
            </select>
          </div>

          <button type="submit" class="btn-primary full-width">LOGIN TO SYSTEM</button>
        </form>
      </div>
    </div>

  {:else}
    <!-- ================= AUTHENTICATED DASHBOARD SHELL ================= -->
    <header class="top-bar">
      <div class="brand-title">
        <span class="brand-icon">🏗️</span>
        <div>
          <span class="company-name">Tugonon Construction Services</span>
          <span class="system-tag">Invoice & Tracking System</span>
        </div>
      </div>

      <!-- Logged-in User Profile Widget -->
      <div class="user-profile-widget">
        <div class="user-info">
          <span class="user-name">{currentUser.name}</span>
          <span class="role-badge {currentUser.role === 'ADMIN_SALES' ? 'badge-admin' : 'badge-finance'}">
            {currentUser.role === 'ADMIN_SALES' ? 'ADMIN / SALES' : 'FINANCE / ACCOUNTING'}
          </span>
        </div>
        <button class="btn-logout" onclick={handleLogout}>Logout</button>
      </div>
    </header>

    <!-- Dynamic Role-Based Navigation Header -->
    <nav class="main-navigation">
      {#if currentUser.role === 'ADMIN_SALES'}
        <!-- Admin/Sales Tabs -->
        <button class="nav-btn {activeTab === 'dashboard' ? 'active' : ''}" onclick={() => activeTab = 'dashboard'}>
          📊 Admin Dashboard
        </button>
        <button class="nav-btn {activeTab === 'clients' ? 'active' : ''}" onclick={() => activeTab = 'clients'}>
          👥 View Clients
        </button>
        <button class="nav-btn {activeTab === 'orders' ? 'active' : ''}" onclick={() => activeTab = 'orders'}>
          📋 Orders & Requests
        </button>
        <button class="nav-btn {activeTab === 'quotations' ? 'active' : ''}" onclick={() => activeTab = 'quotations'}>
          📝 Project Quotations
        </button>

      {:else if currentUser.role === 'FINANCE'}
        <!-- Finance Tabs -->
        <button class="nav-btn {activeTab === 'dashboard' ? 'active' : ''}" onclick={() => activeTab = 'dashboard'}>
          📈 Finance Dashboard
        </button>
        <button class="nav-btn {activeTab === 'invoices' ? 'active' : ''}" onclick={() => activeTab = 'invoices'}>
          📄 Invoice Management
        </button>
        <button class="nav-btn {activeTab === 'payments' ? 'active' : ''}" onclick={() => activeTab = 'payments'}>
          💳 Payment Status
        </button>
        <button class="nav-btn {activeTab === 'receipts' ? 'active' : ''}" onclick={() => activeTab = 'receipts'}>
          🧾 Official Receipts
        </button>
      {/if}
    </nav>

    <!-- Main Workspace Container -->
    <main class="workspace-container">
      {#if currentUser.role === 'ADMIN_SALES'}
        {#if activeTab === 'dashboard'}
          <section class="card">
            <h2>Admin / Sales Overview</h2>
            <div class="stats-grid">
              <div class="stat-box"><span class="stat-num">25</span><span class="stat-label">Total Clients</span></div>
              <div class="stat-box"><span class="stat-num">8</span><span class="stat-label">New Orders</span></div>
              <div class="stat-box"><span class="stat-num">30</span><span class="stat-label">Total Quotations</span></div>
              <div class="stat-box"><span class="stat-num">22</span><span class="stat-label">Approved Quotations</span></div>
            </div>
          </section>

       {:else if activeTab === 'clients'}
          <ClientManager />

        {:else if activeTab === 'orders'}
          <OrderManager />
        {:else if activeTab === 'quotations'}
          <!-- 2. Plugged in the Quotation Form here -->
          <QuotationForm />
        {/if}

      {:else if currentUser.role === 'FINANCE'}
        {#if activeTab === 'dashboard'}
          <section class="card">
            <h2>Finance Overview</h2>
            <div class="stats-grid">
              <div class="stat-box"><span class="stat-num">₱1,200,000</span><span class="stat-label">Total Revenue</span></div>
              <div class="stat-box"><span class="stat-num">110</span><span class="stat-label">Total Invoices</span></div>
              <div class="stat-box"><span class="stat-num">18</span><span class="stat-label">Unpaid Invoices</span></div>
              <div class="stat-box"><span class="stat-num">₱15,000</span><span class="stat-label">Payments Today</span></div>
            </div>
          </section>

        {:else if activeTab === 'invoices'}
          <InvoiceManager />
        {:else if activeTab === 'payments'}
        <PaymentStatus />
        {:else if activeTab === 'receipts'}
        <OfficialReceipts />
        {/if}
      {/if}
    </main>
  {/if}
</main>

<style>
  :global(body) {
    margin: 0;
    font-family: system-ui, -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
    background-color: #121824;
    color: #e2e8f0;
  }

  .app-root { min-height: 100vh; display: flex; flex-direction: column; }

  /* Login Styling */
  .login-wrapper { display: flex; align-items: center; justify-content: center; min-height: 100vh; padding: 1.5rem; }
  .login-card { background: #1e293b; border: 1px solid #334155; padding: 2.5rem; border-radius: 12px; width: 100%; max-width: 420px; box-shadow: 0 10px 25px rgba(0, 0, 0, 0.4); }
  .brand-header { text-align: center; margin-bottom: 2rem; }
  .logo-icon { font-size: 2.5rem; margin-bottom: 0.5rem; }
  .brand-header h2 { margin: 0; font-size: 1.3rem; color: #f8fafc; }
  .subtitle { font-size: 0.85rem; color: #94a3b8; margin-top: 0.4rem; }

  .login-form { display: flex; flex-direction: column; gap: 1.2rem; }
  .form-group { display: flex; flex-direction: column; gap: 0.4rem; text-align: left; }
  .form-group label { font-size: 0.85rem; font-weight: 600; color: #cbd5e1; }
  
  input, select { background: #0f172a; border: 1px solid #334155; color: white; padding: 0.8rem; border-radius: 6px; font-size: 1rem; width: 100%; box-sizing: border-box; }
  input:focus, select:focus { outline: none; border-color: #3b82f6; }

  .btn-primary { background: #2563eb; color: white; border: none; padding: 0.8rem 1.2rem; border-radius: 6px; font-weight: bold; cursor: pointer; transition: background 0.2s; }
  .btn-primary:hover { background: #1d4ed8; }
  .full-width { width: 100%; }

  .alert-error { background: #7f1d1d; border: 1px solid #f87171; color: #fca5a5; padding: 0.8rem; border-radius: 6px; font-size: 0.85rem; margin-bottom: 1rem; text-align: center; }

  /* Top Bar & Header */
  .top-bar { background: #1e293b; border-bottom: 1px solid #334155; padding: 1rem 2rem; display: flex; justify-content: space-between; align-items: center; }
  .brand-title { display: flex; align-items: center; gap: 0.8rem; }
  .brand-icon { font-size: 1.8rem; }
  .company-name { display: block; font-weight: bold; color: #f8fafc; font-size: 1.1rem; }
  .system-tag { display: block; font-size: 0.75rem; color: #94a3b8; }

  .user-profile-widget { display: flex; align-items: center; gap: 1.2rem; }
  .user-info { display: flex; flex-direction: column; align-items: flex-end; gap: 0.2rem; }
  .user-name { font-weight: bold; font-size: 0.95rem; color: #f1f5f9; }
  
  .role-badge { font-size: 0.7rem; font-weight: bold; padding: 0.2rem 0.6rem; border-radius: 12px; }
  .badge-admin { background: #3b82f6; color: white; }
  .badge-finance { background: #10b981; color: white; }

  .btn-logout { background: #dc2626; color: white; border: none; padding: 0.5rem 1rem; border-radius: 6px; font-weight: bold; font-size: 0.85rem; cursor: pointer; }
  .btn-logout:hover { background: #b91c1c; }

  /* Navigation Bar */
  .main-navigation { background: #0f172a; padding: 0.8rem 2rem; display: flex; gap: 1rem; border-bottom: 1px solid #1e293b; }
  .nav-btn { background: transparent; border: 1px solid #334155; color: #94a3b8; padding: 0.6rem 1.2rem; border-radius: 6px; font-weight: 600; cursor: pointer; transition: all 0.2s; }
  .nav-btn:hover { background: #1e293b; color: white; }
  .nav-btn.active { background: #2563eb; color: white; border-color: #3b82f6; }

  /* Workspace */
  .workspace-container { max-width: 1200px; width: 100%; margin: 2rem auto; padding: 0 1.5rem; box-sizing: border-box; }
  .card { background: #1e293b; border: 1px solid #334155; padding: 1.8rem; border-radius: 10px; box-shadow: 0 4px 6px rgba(0,0,0,0.2); }
  .card h2 { margin-top: 0; color: #f8fafc; border-bottom: 1px solid #334155; padding-bottom: 0.8rem; }

  .stats-grid { display: grid; grid-template-columns: repeat(auto-fit, minmax(200px, 1fr)); gap: 1rem; margin-top: 1.5rem; }
  .stat-box { background: #0f172a; border: 1px solid #334155; padding: 1.5rem; border-radius: 8px; text-align: center; display: flex; flex-direction: column; gap: 0.5rem; }
  .stat-num { font-size: 1.8rem; font-weight: bold; color: #38bdf8; }
  .stat-label { font-size: 0.85rem; color: #94a3b8; }
</style>