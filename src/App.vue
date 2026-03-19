<template>
  <div id="app">
    <nav class="navbar">
      <div class="nav-brand">
        <span class="brand-icon">🎓</span>
        <span class="brand-name">PartnerPath</span>
      </div>
      <div class="nav-actions">
        <template v-if="!walletAddress">
          <button class="btn-connect" @click="connectWallet">
            <span class="wallet-icon">🔗</span> Connect Wallet
          </button>
        </template>
        <template v-else>
          <div class="wallet-info" @click="currentPage = 'wallet'">
            <span class="wallet-points">⭐ {{ userPoints }} pts</span>
            <span class="wallet-addr">{{ shortAddress }}</span>
          </div>
          <button class="btn-disconnect" @click="disconnectWallet">✕</button>
        </template>
      </div>
    </nav>

    <div class="page-container">

      <!-- HOME PAGE -->
      <div v-if="currentPage === 'home'" class="page">
        <div class="hero">
          <h1 class="hero-title">Find Your <span class="accent">OJT</span> Opportunity</h1>
          <p class="hero-sub">Discover internships, earn rewards, build your career.</p>
          <div class="search-bar">
            <input v-model="searchQuery" placeholder="Search by title, company..." class="search-input" />
            <button class="btn-search" @click="filterOpportunities">🔍 Search</button>
          </div>
          <div class="filters">
            <select v-model="filterCategory" class="filter-select">
              <option value="">All Categories</option>
              <option>IT / Tech</option>
              <option>Business</option>
              <option>Engineering</option>
              <option>Education</option>
              <option>Healthcare</option>
            </select>
            <select v-model="filterLocation" class="filter-select">
              <option value="">All Locations</option>
              <option>Manila</option>
              <option>Cebu</option>
              <option>Davao</option>
              <option>Remote</option>
            </select>
            <select v-model="filterType" class="filter-select">
              <option value="">Paid & Unpaid</option>
              <option value="paid">Paid</option>
              <option value="unpaid">Unpaid</option>
            </select>
            <button class="btn-filter" @click="filterOpportunities">Apply</button>
          </div>
        </div>

        <div class="opportunities-grid">
          <div
            v-for="opp in filteredOpportunities"
            :key="opp.id"
            :class="['opp-card', opp.premium ? 'premium-card' : '']"
          >
            <div class="card-header">
              <span class="card-badge" :class="opp.paid ? 'paid' : 'unpaid'">
                {{ opp.paid ? '💰 Paid' : 'Unpaid' }}
              </span>
              <span v-if="opp.premium" class="premium-badge">⭐ Premium</span>
            </div>
            <h3 class="card-title">{{ opp.title }}</h3>
            <p class="card-company">🏢 {{ opp.company }}</p>
            <p class="card-location">📍 {{ opp.location }}</p>
            <p class="card-category">🏷️ {{ opp.category }}</p>
            <p class="card-desc">{{ opp.description }}</p>
            <div class="card-footer">
              <span class="points-reward">+10 pts on apply</span>
              <button v-if="!opp.premium || canAccessPremium" class="btn-apply" @click="openApply(opp)">Apply Now</button>
              <button v-else class="btn-locked" @click="currentPage = 'wallet'">🔒 Unlock (100 pts)</button>
            </div>
          </div>
        </div>
      </div>

      <!-- APPLY MODAL -->
      <div v-if="showApplyModal" class="modal-overlay" @click.self="showApplyModal = false">
        <div class="modal">
          <h2>Apply: {{ selectedOpp?.title }}</h2>
          <p class="modal-company">{{ selectedOpp?.company }}</p>
          <div class="form-group">
            <label>Full Name</label>
            <input v-model="applyForm.name" placeholder="Your full name" class="form-input" />
          </div>
          <div class="form-group">
            <label>Email</label>
            <input v-model="applyForm.email" placeholder="your@email.com" class="form-input" />
          </div>
          <div class="form-group">
            <label>Message</label>
            <textarea v-model="applyForm.message" placeholder="Why are you interested?" class="form-textarea"></textarea>
          </div>
          <div class="form-group">
            <label>Resume (PDF)</label>
            <input type="file" class="form-input" accept=".pdf" />
          </div>
          <div class="modal-actions">
            <button class="btn-cancel" @click="showApplyModal = false">Cancel</button>
            <button class="btn-submit" @click="submitApplication">Submit Application</button>
          </div>
        </div>
      </div>

      <!-- WALLET PAGE -->
      <div v-if="currentPage === 'wallet'" class="page wallet-page">
        <div class="wallet-hero">
          <div class="wallet-card">
            <div class="wallet-card-top">
              <span class="wallet-label">Your OJT Wallet</span>
              <span class="wallet-network">Base Network</span>
            </div>
            <div class="wallet-balance">
              <span class="balance-num">{{ userPoints }}</span>
              <span class="balance-label">POINTS</span>
            </div>
            <div class="wallet-address-display">{{ walletAddress || 'Not Connected' }}</div>
          </div>
        </div>

        <div class="section">
          <h2 class="section-title">How to Earn Points</h2>
          <div class="earn-grid">
            <div class="earn-card" v-for="earn in earnMethods" :key="earn.label">
              <span class="earn-icon">{{ earn.icon }}</span>
              <div>
                <p class="earn-label">{{ earn.label }}</p>
                <p class="earn-pts">+{{ earn.points }} pts</p>
              </div>
              <button v-if="earn.action" class="btn-earn" @click="claimReward(earn)">
                {{ earn.claimed ? '✓ Claimed' : 'Claim' }}
              </button>
            </div>
          </div>
        </div>

        <div class="section">
          <h2 class="section-title">Use Your Points</h2>
          <div class="use-grid">
            <div class="use-card" v-for="use in useOptions" :key="use.label">
              <span class="use-icon">{{ use.icon }}</span>
              <p class="use-label">{{ use.label }}</p>
              <p class="use-cost">{{ use.cost }} pts</p>
              <button class="btn-use" :disabled="userPoints < use.cost" @click="usePoints(use)">
                {{ userPoints >= use.cost ? 'Redeem' : 'Need More Points' }}
              </button>
            </div>
          </div>
        </div>

        <div class="section">
          <h2 class="section-title">Transaction History</h2>
          <div class="tx-list">
            <div class="tx-item" v-for="tx in transactions" :key="tx.id">
              <span class="tx-icon">{{ tx.icon }}</span>
              <div class="tx-info">
                <p class="tx-label">{{ tx.label }}</p>
                <p class="tx-date">{{ tx.date }}</p>
              </div>
              <span :class="['tx-amount', tx.type === 'earn' ? 'positive' : 'negative']">
                {{ tx.type === 'earn' ? '+' : '-' }}{{ tx.points }} pts
              </span>
            </div>
            <p v-if="transactions.length === 0" class="empty-state">No transactions yet. Start earning!</p>
          </div>
        </div>
      </div>

    </div>

    <nav class="bottom-nav">
      <button :class="['nav-btn', currentPage === 'home' ? 'active' : '']" @click="currentPage = 'home'">
        <span>🔍</span><small>Explore</small>
      </button>
      <button :class="['nav-btn', currentPage === 'wallet' ? 'active' : '']" @click="currentPage = 'wallet'">
        <span>👛</span><small>Wallet</small>
      </button>
    </nav>

    <div v-if="toast.show" class="toast" :class="toast.type">{{ toast.message }}</div>
  </div>
</template>

<script>
export default {
  name: 'App',
  data() {
    return {
      currentPage: 'home',
      walletAddress: null,
      userPoints: 0,
      searchQuery: '',
      filterCategory: '',
      filterLocation: '',
      filterType: '',
      showApplyModal: false,
      selectedOpp: null,
      applyForm: { name: '', email: '', message: '' },
      toast: { show: false, message: '', type: '' },
      transactions: [],

      opportunities: [
        { id: 1, title: 'Frontend Developer Intern', company: 'TechPH Inc.', location: 'Remote', category: 'IT / Tech', paid: true, premium: false, description: 'Work on Vue.js projects, learn real-world dev workflows.' },
        { id: 2, title: 'Business Analyst OJT', company: 'GlobalBPO', location: 'Manila', category: 'Business', paid: false, premium: false, description: 'Support analytics team, prepare reports and presentations.' },
        { id: 3, title: 'Full Stack Engineer Intern', company: 'StartupMNL', location: 'Cebu', category: 'IT / Tech', paid: true, premium: true, description: '🌟 Premium opportunity with mentorship and allowance.' },
        { id: 4, title: 'Civil Engineering OJT', company: 'BuildRight PH', location: 'Davao', category: 'Engineering', paid: true, premium: false, description: 'On-site training with licensed engineers.' },
        { id: 5, title: 'HR Recruitment Intern', company: 'PeopleFirst Corp', location: 'Manila', category: 'Business', paid: false, premium: false, description: 'Learn recruitment, onboarding, and HR operations.' },
        { id: 6, title: 'AI Research Intern', company: 'DeepTech Labs', location: 'Remote', category: 'IT / Tech', paid: true, premium: true, description: '🌟 Premium – Work on cutting-edge AI/ML projects.' },
      ],

      earnMethods: [
        { icon: '📅', label: 'Daily Login', points: 5, action: true, claimed: false },
        { icon: '👤', label: 'Complete Profile', points: 20, action: true, claimed: false },
        { icon: '📨', label: 'Apply to Opportunity', points: 10, action: false, claimed: false },
        { icon: '👥', label: 'Refer a Friend', points: 30, action: true, claimed: false },
        { icon: '✅', label: 'Complete Volunteer Task', points: 50, action: false, claimed: false },
      ],

      useOptions: [
        { icon: '⭐', label: 'Unlock Premium Opportunities', cost: 100 },
        { icon: '🚀', label: 'Highlight Profile to Companies', cost: 50 },
        { icon: '🏅', label: 'Get Digital Certificate', cost: 200 },
      ],
    }
  },
  computed: {
    shortAddress() {
      if (!this.walletAddress) return ''
      return this.walletAddress.slice(0, 6) + '...' + this.walletAddress.slice(-4)
    },
    canAccessPremium() {
      return this.userPoints >= 100
    },
    filteredOpportunities() {
      return this.opportunities.filter(o => {
        const matchSearch = !this.searchQuery ||
          o.title.toLowerCase().includes(this.searchQuery.toLowerCase()) ||
          o.company.toLowerCase().includes(this.searchQuery.toLowerCase())
        const matchCat = !this.filterCategory || o.category === this.filterCategory
        const matchLoc = !this.filterLocation || o.location === this.filterLocation
        const matchType = !this.filterType ||
          (this.filterType === 'paid' && o.paid) ||
          (this.filterType === 'unpaid' && !o.paid)
        return matchSearch && matchCat && matchLoc && matchType
      })
    }
  },
  methods: {
    async connectWallet() {
      if (typeof window.ethereum !== 'undefined') {
        try {
          const accounts = await window.ethereum.request({ method: 'eth_requestAccounts' })
          this.walletAddress = accounts[0]
          this.showToast('✅ Wallet connected!', 'success')
          this.checkDailyLogin()
        } catch (err) {
          this.showToast('❌ Connection failed', 'error')
        }
      } else {
        this.walletAddress = '0xDEMO...1234'
        this.showToast('👛 Demo mode active (install MetaMask/Coinbase Wallet for real)', 'info')
        this.checkDailyLogin()
      }
    },
    disconnectWallet() {
      this.walletAddress = null
      this.showToast('Wallet disconnected', 'info')
    },
    checkDailyLogin() {
      const today = new Date().toDateString()
      const lastLogin = localStorage.getItem('lastLogin')
      if (lastLogin !== today) {
        localStorage.setItem('lastLogin', today)
        this.addPoints(5, '📅 Daily Login Reward')
        this.earnMethods[0].claimed = true
      }
    },
    openApply(opp) {
      if (!this.walletAddress) {
        this.showToast('Please connect your wallet first!', 'error')
        return
      }
      this.selectedOpp = opp
      this.showApplyModal = true
    },
    submitApplication() {
      if (!this.applyForm.name || !this.applyForm.email) {
        this.showToast('Please fill in required fields', 'error')
        return
      }
      this.showApplyModal = false
      this.addPoints(10, `📨 Applied to ${this.selectedOpp.title}`)
      this.showToast('🎉 Application submitted! +10 pts earned!', 'success')
      this.applyForm = { name: '', email: '', message: '' }
    },
    claimReward(earn) {
      if (earn.claimed) return
      earn.claimed = true
      this.addPoints(earn.points, earn.icon + ' ' + earn.label)
      this.showToast(`+${earn.points} points earned!`, 'success')
    },
    usePoints(use) {
      if (this.userPoints < use.cost) return
      this.userPoints -= use.cost
      this.transactions.unshift({
        id: Date.now(), icon: use.icon, label: use.label,
        date: new Date().toLocaleDateString(), type: 'spend', points: use.cost
      })
      this.showToast(`✅ ${use.label} unlocked!`, 'success')
    },
    // FIX: Removed unused 'key' parameter (was causing the ESLint error)
    addPoints(pts, label) {
      this.userPoints += pts
      this.transactions.unshift({
        id: Date.now(), icon: '⭐', label,
        date: new Date().toLocaleDateString(), type: 'earn', points: pts
      })
    },
    filterOpportunities() { /* computed handles it */ },
    showToast(message, type = 'info') {
      this.toast = { show: true, message, type }
      setTimeout(() => { this.toast.show = false }, 3000)
    }
  },
  mounted() {
    const saved = localStorage.getItem('ojtPoints')
    if (saved) this.userPoints = parseInt(saved)
  },
  watch: {
    userPoints(val) { localStorage.setItem('ojtPoints', val) }
  }
}
</script>

<style>
@import url('https://fonts.googleapis.com/css2?family=Space+Grotesk:wght@400;500;600;700&family=DM+Mono:wght@400;500&display=swap');

* { margin: 0; padding: 0; box-sizing: border-box; }

:root {
  --bg: #0a0a0f; --surface: #13131a; --surface2: #1c1c27;
  --accent: #6366f1; --accent2: #8b5cf6; --green: #22c55e;
  --yellow: #eab308; --red: #ef4444; --text: #f1f5f9;
  --text-muted: #64748b; --border: #2a2a3a; --radius: 14px;
}

body { background: var(--bg); color: var(--text); font-family: 'Space Grotesk', sans-serif; }
#app { min-height: 100vh; padding-bottom: 80px; }

.navbar { display: flex; justify-content: space-between; align-items: center; padding: 16px 20px; background: var(--surface); border-bottom: 1px solid var(--border); position: sticky; top: 0; z-index: 100; }
.nav-brand { display: flex; align-items: center; gap: 8px; font-weight: 700; font-size: 1.2rem; }
.brand-icon { font-size: 1.4rem; }
.nav-actions { display: flex; align-items: center; gap: 8px; }
.btn-connect { background: var(--accent); color: white; border: none; padding: 8px 16px; border-radius: 999px; cursor: pointer; font-weight: 600; font-size: 0.85rem; display: flex; align-items: center; gap: 6px; }
.wallet-info { background: var(--surface2); border: 1px solid var(--border); padding: 8px 14px; border-radius: 999px; cursor: pointer; display: flex; align-items: center; gap: 10px; font-size: 0.85rem; }
.wallet-points { color: var(--yellow); font-weight: 700; }
.wallet-addr { color: var(--text-muted); font-family: 'DM Mono', monospace; }
.btn-disconnect { background: none; border: 1px solid var(--border); color: var(--text-muted); padding: 8px 10px; border-radius: 8px; cursor: pointer; }

.page-container { padding: 20px; max-width: 900px; margin: 0 auto; }
.hero { text-align: center; padding: 40px 0 30px; }
.hero-title { font-size: 2.2rem; font-weight: 700; margin-bottom: 10px; }
.accent { color: var(--accent); }
.hero-sub { color: var(--text-muted); margin-bottom: 24px; }
.search-bar { display: flex; gap: 8px; max-width: 500px; margin: 0 auto 16px; }
.search-input { flex: 1; background: var(--surface2); border: 1px solid var(--border); color: var(--text); padding: 12px 16px; border-radius: var(--radius); font-size: 0.95rem; }
.btn-search { background: var(--accent); color: white; border: none; padding: 12px 20px; border-radius: var(--radius); cursor: pointer; font-weight: 600; }
.filters { display: flex; gap: 8px; flex-wrap: wrap; justify-content: center; }
.filter-select { background: var(--surface2); border: 1px solid var(--border); color: var(--text); padding: 10px 14px; border-radius: var(--radius); font-size: 0.85rem; cursor: pointer; }
.btn-filter { background: var(--accent2); color: white; border: none; padding: 10px 18px; border-radius: var(--radius); cursor: pointer; font-weight: 600; }

.opportunities-grid { display: grid; grid-template-columns: repeat(auto-fill, minmax(280px, 1fr)); gap: 16px; margin-top: 20px; }
.opp-card { background: var(--surface); border: 1px solid var(--border); border-radius: var(--radius); padding: 18px; transition: transform 0.2s, border-color 0.2s; }
.opp-card:hover { transform: translateY(-3px); border-color: var(--accent); }
.premium-card { border-color: var(--yellow); background: linear-gradient(135deg, #13131a, #1c1a0a); }
.card-header { display: flex; justify-content: space-between; margin-bottom: 10px; }
.card-badge { font-size: 0.75rem; padding: 3px 8px; border-radius: 999px; }
.card-badge.paid { background: rgba(34,197,94,0.15); color: var(--green); }
.card-badge.unpaid { background: rgba(100,116,139,0.15); color: var(--text-muted); }
.premium-badge { font-size: 0.75rem; color: var(--yellow); }
.card-title { font-size: 1rem; font-weight: 700; margin-bottom: 6px; }
.card-company, .card-location, .card-category { font-size: 0.82rem; color: var(--text-muted); margin-bottom: 3px; }
.card-desc { font-size: 0.82rem; color: var(--text-muted); margin: 10px 0; line-height: 1.4; }
.card-footer { display: flex; justify-content: space-between; align-items: center; margin-top: 12px; }
.points-reward { font-size: 0.75rem; color: var(--yellow); }
.btn-apply { background: var(--accent); color: white; border: none; padding: 8px 16px; border-radius: 8px; cursor: pointer; font-weight: 600; font-size: 0.85rem; }
.btn-locked { background: var(--surface2); color: var(--yellow); border: 1px solid var(--yellow); padding: 8px 14px; border-radius: 8px; cursor: pointer; font-size: 0.82rem; }

.modal-overlay { position: fixed; inset: 0; background: rgba(0,0,0,0.7); display: flex; align-items: center; justify-content: center; z-index: 200; padding: 20px; }
.modal { background: var(--surface); border: 1px solid var(--border); border-radius: var(--radius); padding: 24px; width: 100%; max-width: 440px; }
.modal h2 { font-size: 1.2rem; margin-bottom: 4px; }
.modal-company { color: var(--text-muted); font-size: 0.85rem; margin-bottom: 16px; }
.form-group { margin-bottom: 14px; }
.form-group label { display: block; font-size: 0.85rem; margin-bottom: 6px; color: var(--text-muted); }
.form-input, .form-textarea { width: 100%; background: var(--surface2); border: 1px solid var(--border); color: var(--text); padding: 10px 14px; border-radius: 10px; font-size: 0.9rem; }
.form-textarea { height: 80px; resize: vertical; }
.modal-actions { display: flex; gap: 10px; justify-content: flex-end; margin-top: 16px; }
.btn-cancel { background: none; border: 1px solid var(--border); color: var(--text-muted); padding: 10px 18px; border-radius: 10px; cursor: pointer; }
.btn-submit { background: var(--accent); color: white; border: none; padding: 10px 20px; border-radius: 10px; cursor: pointer; font-weight: 600; }

.wallet-page { max-width: 600px; margin: 0 auto; }
.wallet-hero { margin-bottom: 30px; }
.wallet-card { background: linear-gradient(135deg, var(--accent), var(--accent2)); border-radius: 20px; padding: 24px; color: white; }
.wallet-card-top { display: flex; justify-content: space-between; margin-bottom: 16px; font-size: 0.85rem; opacity: 0.8; }
.wallet-balance { display: flex; align-items: baseline; gap: 8px; margin-bottom: 16px; }
.balance-num { font-size: 3rem; font-weight: 700; }
.balance-label { font-size: 0.9rem; opacity: 0.7; }
.wallet-address-display { font-family: 'DM Mono', monospace; font-size: 0.75rem; opacity: 0.6; }

.section { margin-bottom: 30px; }
.section-title { font-size: 1.1rem; font-weight: 700; margin-bottom: 14px; border-left: 3px solid var(--accent); padding-left: 10px; }
.earn-grid { display: flex; flex-direction: column; gap: 10px; }
.earn-card { background: var(--surface); border: 1px solid var(--border); border-radius: var(--radius); padding: 14px 16px; display: flex; align-items: center; gap: 14px; }
.earn-icon { font-size: 1.5rem; }
.earn-label { font-size: 0.9rem; font-weight: 600; }
.earn-pts { font-size: 0.82rem; color: var(--yellow); }
.btn-earn { margin-left: auto; background: rgba(99,102,241,0.2); color: var(--accent); border: 1px solid var(--accent); padding: 6px 14px; border-radius: 8px; cursor: pointer; font-size: 0.82rem; }

.use-grid { display: grid; grid-template-columns: repeat(auto-fill, minmax(160px, 1fr)); gap: 12px; }
.use-card { background: var(--surface); border: 1px solid var(--border); border-radius: var(--radius); padding: 16px; text-align: center; }
.use-icon { font-size: 2rem; display: block; margin-bottom: 8px; }
.use-label { font-size: 0.82rem; font-weight: 600; margin-bottom: 4px; }
.use-cost { font-size: 0.8rem; color: var(--yellow); margin-bottom: 10px; }
.btn-use { width: 100%; background: var(--accent); color: white; border: none; padding: 8px; border-radius: 8px; cursor: pointer; font-size: 0.8rem; font-weight: 600; }
.btn-use:disabled { background: var(--surface2); color: var(--text-muted); cursor: not-allowed; }

.tx-list { display: flex; flex-direction: column; gap: 8px; }
.tx-item { background: var(--surface); border: 1px solid var(--border); border-radius: 10px; padding: 12px 16px; display: flex; align-items: center; gap: 12px; }
.tx-icon { font-size: 1.2rem; }
.tx-info { flex: 1; }
.tx-label { font-size: 0.85rem; font-weight: 600; }
.tx-date { font-size: 0.75rem; color: var(--text-muted); }
.tx-amount { font-weight: 700; font-size: 0.9rem; }
.positive { color: var(--green); }
.negative { color: var(--red); }
.empty-state { color: var(--text-muted); text-align: center; padding: 20px; }

.bottom-nav { position: fixed; bottom: 0; left: 0; right: 0; background: var(--surface); border-top: 1px solid var(--border); display: flex; z-index: 100; }
.nav-btn { flex: 1; background: none; border: none; color: var(--text-muted); padding: 12px; cursor: pointer; display: flex; flex-direction: column; align-items: center; gap: 2px; font-size: 1.3rem; transition: color 0.2s; }
.nav-btn small { font-size: 0.7rem; font-family: 'Space Grotesk', sans-serif; }
.nav-btn.active { color: var(--accent); }

.toast { position: fixed; bottom: 90px; left: 50%; transform: translateX(-50%); padding: 12px 24px; border-radius: 999px; font-weight: 600; font-size: 0.9rem; z-index: 300; animation: slideUp 0.3s ease; }
.toast.success { background: var(--green); color: white; }
.toast.error { background: var(--red); color: white; }
.toast.info { background: var(--accent); color: white; }

@keyframes slideUp { from { opacity: 0; transform: translateX(-50%) translateY(20px); } to { opacity: 1; transform: translateX(-50%) translateY(0); } }
input:focus, select:focus, textarea:focus { outline: 2px solid var(--accent); border-color: var(--accent); }
</style>