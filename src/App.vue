<template>
  <div id="app">

    <!-- ===================== AUTH PAGES ===================== -->
    <div v-if="!currentUser" class="auth-wrapper">
      <div class="auth-card">
        <div class="auth-brand">
          <span>🤝</span>
          <h1>PartnerPath</h1>
          <p>Your partner in every OJT step</p>
        </div>

        <!-- LOGIN -->
        <div v-if="authPage === 'login'">
          <h2 class="auth-title">Welcome back!</h2>
          <div class="form-group">
            <label>Email</label>
            <input v-model="authForm.email" type="email" placeholder="your@email.com" class="form-input" />
          </div>
          <div class="form-group">
            <label>Password</label>
            <input v-model="authForm.password" type="password" placeholder="••••••••" class="form-input" />
          </div>
          <button class="btn-auth" @click="login" :disabled="authLoading">
            {{ authLoading ? 'Signing in...' : 'Sign In' }}
          </button>
          <p class="auth-switch">
            Wala pang account?
            <span @click="authPage = 'signup'">Sign Up</span>
          </p>
        </div>

        <!-- SIGN UP -->
        <div v-if="authPage === 'signup'">
          <h2 class="auth-title">Create Account</h2>
          <div class="form-group">
            <label>Full Name</label>
            <input v-model="authForm.name" placeholder="Juan Dela Cruz" class="form-input" />
          </div>
          <div class="form-group">
            <label>Email</label>
            <input v-model="authForm.email" type="email" placeholder="your@email.com" class="form-input" />
          </div>
          <div class="form-group">
            <label>Password</label>
            <input v-model="authForm.password" type="password" placeholder="Min. 6 characters" class="form-input" />
          </div>
          <div class="form-group">
            <label>School</label>
            <input v-model="authForm.school" placeholder="University / College" class="form-input" />
          </div>
          <div class="form-group">
            <label>Course</label>
            <select v-model="authForm.course" class="form-input">
              <option value="">Select Course</option>
              <option>BS Information Technology</option>
              <option>BS Computer Science</option>
              <option>BS Business Administration</option>
              <option>BS Civil Engineering</option>
              <option>BS Nursing</option>
              <option>BS Education</option>
              <option>Other</option>
            </select>
          </div>
          <div class="form-group">
            <label>Year Level</label>
            <select v-model="authForm.year" class="form-input">
              <option value="">Select Year</option>
              <option>3rd Year</option>
              <option>4th Year</option>
              <option>Graduate</option>
            </select>
          </div>
          <button class="btn-auth" @click="signup" :disabled="authLoading">
            {{ authLoading ? 'Creating account...' : 'Sign Up' }}
          </button>
          <p class="auth-switch">
            May account na?
            <span @click="authPage = 'login'">Sign In</span>
          </p>
        </div>

        <p v-if="authError" class="auth-error">{{ authError }}</p>
      </div>
    </div>

    <!-- ===================== MAIN APP ===================== -->
    <div v-else>
      <nav class="navbar">
        <div class="nav-brand">
          <span class="brand-icon">🤝</span>
          <span class="brand-name">PartnerPath</span>
        </div>
        <div class="nav-actions">
          <template v-if="!walletAddress">
            <button class="btn-connect" @click="connectWallet">
              <span>🔗</span> Connect Wallet
            </button>
          </template>
          <template v-else>
            <div class="wallet-info" @click="currentPage = 'wallet'">
              <span class="wallet-points">⭐ {{ userPoints }} pts</span>
              <span class="wallet-addr">{{ shortAddress }}</span>
            </div>
          </template>
          <div class="user-menu" @click="showUserMenu = !showUserMenu">
            <span class="user-avatar">{{ userInitial }}</span>
            <div v-if="showUserMenu" class="user-dropdown">
              <p class="dropdown-name">{{ currentUser.name }}</p>
              <p class="dropdown-email">{{ currentUser.email }}</p>
              <hr />
              <button @click="currentPage = 'profile'">👤 Profile</button>
              <button @click="logout" class="logout-btn">🚪 Sign Out</button>
            </div>
          </div>
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
              <button class="btn-search">🔍 Search</button>
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
            </div>
          </div>

          <div class="opportunities-grid">
            <div v-for="opp in filteredOpportunities" :key="opp.id" :class="['opp-card', opp.premium ? 'premium-card' : '']">
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

        <!-- PROFILE PAGE -->
        <div v-if="currentPage === 'profile'" class="page profile-page">
          <h2 class="section-title">My Profile</h2>
          <div class="profile-card">
            <div class="profile-avatar">{{ userInitial }}</div>
            <div class="profile-info">
              <h3>{{ currentUser.name }}</h3>
              <p>{{ currentUser.email }}</p>
              <p>📚 {{ currentUser.course || 'No course set' }}</p>
              <p>🏫 {{ currentUser.school || 'No school set' }}</p>
              <p>📅 {{ currentUser.year || 'No year set' }}</p>
            </div>
          </div>
          <div class="profile-stats">
            <div class="stat-box">
              <span class="stat-num">{{ userPoints }}</span>
              <span class="stat-label">Points</span>
            </div>
            <div class="stat-box">
              <span class="stat-num">{{ transactions.filter(t => t.type === 'earn').length }}</span>
              <span class="stat-label">Activities</span>
            </div>
            <div class="stat-box">
              <span class="stat-num">{{ applications.length }}</span>
              <span class="stat-label">Applications</span>
            </div>
          </div>

          <!-- Edit Profile -->
          <div class="section">
            <h2 class="section-title">Edit Profile</h2>
            <div class="form-group">
              <label>Full Name</label>
              <input v-model="editForm.name" class="form-input" />
            </div>
            <div class="form-group">
              <label>School</label>
              <input v-model="editForm.school" class="form-input" />
            </div>
            <div class="form-group">
              <label>Course</label>
              <select v-model="editForm.course" class="form-input">
                <option>BS Information Technology</option>
                <option>BS Computer Science</option>
                <option>BS Business Administration</option>
                <option>BS Civil Engineering</option>
                <option>BS Nursing</option>
                <option>BS Education</option>
                <option>Other</option>
              </select>
            </div>
            <div class="form-group">
              <label>Year Level</label>
              <select v-model="editForm.year" class="form-input">
                <option>3rd Year</option>
                <option>4th Year</option>
                <option>Graduate</option>
              </select>
            </div>
            <button class="btn-auth" @click="saveProfile">💾 Save Profile</button>
          </div>
        </div>

        <!-- WALLET PAGE -->
        <div v-if="currentPage === 'wallet'" class="page wallet-page">
          <div class="wallet-hero">
            <div class="wallet-card">
              <div class="wallet-card-top">
                <span>Your PartnerPath Wallet</span>
                <span>Base Network</span>
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
                  {{ userPoints >= use.cost ? 'Redeem' : 'Need More' }}
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

      <!-- APPLY MODAL -->
      <div v-if="showApplyModal" class="modal-overlay" @click.self="showApplyModal = false">
        <div class="modal">
          <h2>Apply: {{ selectedOpp?.title }}</h2>
          <p class="modal-company">{{ selectedOpp?.company }}</p>
          <div class="form-group">
            <label>Full Name</label>
            <input v-model="applyForm.name" class="form-input" />
          </div>
          <div class="form-group">
            <label>Email</label>
            <input v-model="applyForm.email" class="form-input" />
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
            <button class="btn-submit" @click="submitApplication">Submit</button>
          </div>
        </div>
      </div>

      <!-- BOTTOM NAV -->
      <nav class="bottom-nav">
        <button :class="['nav-btn', currentPage === 'home' ? 'active' : '']" @click="currentPage = 'home'">
          <span>🔍</span><small>Explore</small>
        </button>
        <button :class="['nav-btn', currentPage === 'wallet' ? 'active' : '']" @click="currentPage = 'wallet'">
          <span>👛</span><small>Wallet</small>
        </button>
        <button :class="['nav-btn', currentPage === 'profile' ? 'active' : '']" @click="currentPage = 'profile'">
          <span>👤</span><small>Profile</small>
        </button>
      </nav>
    </div>

    <div v-if="toast.show" class="toast" :class="toast.type">{{ toast.message }}</div>
  </div>
</template>

<script>
// ============================================================
// SUPABASE SETUP
// Palitan ng iyong sariling URL at KEY mula sa supabase.com
// Settings → API → Project URL & anon public key
// ============================================================
const SUPABASE_URL = 'https://kjbsomtffnpabcmlkrdd.supabase.co'
const SUPABASE_KEY = 'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6ImtqYnNvbXRmZm5wYWJjbWxrcmRkIiwicm9sZSI6ImFub24iLCJpYXQiOjE3NzM5MDU3MDIsImV4cCI6MjA4OTQ4MTcwMn0.OPqic14AV9porM4rOTv8W0_oXCSpaMgD3ZmkJNlfF08'

async function supabase(method, path, body = null, token = null) {
  const headers = {
    'Content-Type': 'application/json',
    'apikey': SUPABASE_KEY,
    'Authorization': `Bearer ${token || SUPABASE_KEY}`
  }
  const res = await fetch(`${SUPABASE_URL}${path}`, {
    method,
    headers,
    body: body ? JSON.stringify(body) : null
  })
  return res.json()
}

export default {
  name: 'App',
  data() {
    return {
      // Auth
      authPage: 'login',
      authLoading: false,
      authError: '',
      authForm: { name: '', email: '', password: '', school: '', course: '', year: '' },
      currentUser: null,
      userToken: null,

      // App
      currentPage: 'home',
      walletAddress: null,
      userPoints: 0,
      showUserMenu: false,
      applications: [],

      // Search
      searchQuery: '',
      filterCategory: '',
      filterLocation: '',
      filterType: '',

      // Apply
      showApplyModal: false,
      selectedOpp: null,
      applyForm: { name: '', email: '', message: '' },

      // Edit Profile
      editForm: { name: '', school: '', course: '', year: '' },

      // Toast
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
    userInitial() {
      return this.currentUser?.name?.charAt(0).toUpperCase() || '?'
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
    // ========== AUTH ==========
    async signup() {
      if (!this.authForm.name || !this.authForm.email || !this.authForm.password) {
        this.authError = 'Please fill in all required fields.'
        return
      }
      this.authLoading = true
      this.authError = ''
      try {
        // 1. Create auth user
        const authRes = await supabase('POST', '/auth/v1/signup', {
          email: this.authForm.email,
          password: this.authForm.password
        })
        if (authRes.error) throw new Error(authRes.error.message)

        // 2. Save profile to students table
        await supabase('POST', '/rest/v1/students', {
          id: authRes.user.id,
          name: this.authForm.name,
          email: this.authForm.email,
          school: this.authForm.school,
          course: this.authForm.course,
          year: this.authForm.year,
          points: 0
        }, authRes.access_token)

        this.currentUser = {
          id: authRes.user.id,
          name: this.authForm.name,
          email: this.authForm.email,
          school: this.authForm.school,
          course: this.authForm.course,
          year: this.authForm.year
        }
        this.userToken = authRes.access_token
        this.editForm = { ...this.currentUser }
        localStorage.setItem('pp_user', JSON.stringify(this.currentUser))
        localStorage.setItem('pp_token', this.userToken)
        this.showToast('🎉 Account created! Welcome!', 'success')
        this.checkDailyLogin()
      } catch (err) {
        this.authError = err.message || 'Signup failed. Try again.'
      }
      this.authLoading = false
    },

    async login() {
      if (!this.authForm.email || !this.authForm.password) {
        this.authError = 'Please enter email and password.'
        return
      }
      this.authLoading = true
      this.authError = ''
      try {
        const res = await supabase('POST', '/auth/v1/token?grant_type=password', {
          email: this.authForm.email,
          password: this.authForm.password
        })
        if (res.error) throw new Error(res.error.message)

        // Get profile
        const profile = await supabase('GET', `/rest/v1/students?id=eq.${res.user.id}&select=*`, null, res.access_token)

        this.currentUser = profile[0] || { id: res.user.id, name: res.user.email, email: res.user.email }
        this.userToken = res.access_token
        this.userPoints = this.currentUser.points || 0
        this.editForm = { ...this.currentUser }
        localStorage.setItem('pp_user', JSON.stringify(this.currentUser))
        localStorage.setItem('pp_token', this.userToken)
        this.showToast('✅ Welcome back!', 'success')
        this.checkDailyLogin()
      } catch (err) {
        this.authError = err.message || 'Login failed. Check your credentials.'
      }
      this.authLoading = false
    },

    logout() {
      this.currentUser = null
      this.userToken = null
      this.walletAddress = null
      this.userPoints = 0
      this.transactions = []
      this.showUserMenu = false
      localStorage.removeItem('pp_user')
      localStorage.removeItem('pp_token')
      this.showToast('Signed out', 'info')
    },

    // ========== PROFILE ==========
    async saveProfile() {
      if (!this.currentUser) return
      try {
        await supabase('PATCH', `/rest/v1/students?id=eq.${this.currentUser.id}`, {
          name: this.editForm.name,
          school: this.editForm.school,
          course: this.editForm.course,
          year: this.editForm.year
        }, this.userToken)
        this.currentUser = { ...this.currentUser, ...this.editForm }
        localStorage.setItem('pp_user', JSON.stringify(this.currentUser))
        this.showToast('✅ Profile saved! +20 pts', 'success')
        this.addPoints(20, '👤 Profile Completed')
        this.earnMethods[1].claimed = true
      } catch (err) {
        this.showToast('Failed to save profile', 'error')
      }
    },

    // ========== WALLET ==========
    async connectWallet() {
      if (typeof window.ethereum !== 'undefined') {
        try {
          const accounts = await window.ethereum.request({ method: 'eth_requestAccounts' })
          this.walletAddress = accounts[0]
          this.showToast('✅ Wallet connected!', 'success')
        } catch (err) {
          this.showToast('❌ Connection failed', 'error')
        }
      } else {
        this.walletAddress = '0xDEMO...1234'
        this.showToast('👛 Demo mode active', 'info')
      }
    },

    // ========== APPLY ==========
    openApply(opp) {
      this.selectedOpp = opp
      this.applyForm.name = this.currentUser.name
      this.applyForm.email = this.currentUser.email
      this.showApplyModal = true
    },
    async submitApplication() {
      if (!this.applyForm.name || !this.applyForm.email) {
        this.showToast('Please fill in required fields', 'error')
        return
      }
      // Save application to Supabase
      try {
        await supabase('POST', '/rest/v1/applications', {
          student_id: this.currentUser.id,
          opportunity_title: this.selectedOpp.title,
          company: this.selectedOpp.company,
          student_name: this.applyForm.name,
          student_email: this.applyForm.email,
          message: this.applyForm.message
        }, this.userToken)
      } catch (err) {
        console.log('Supabase not set up yet, running in demo mode')
      }
      this.applications.push(this.selectedOpp)
      this.showApplyModal = false
      this.addPoints(10, `📨 Applied to ${this.selectedOpp.title}`)
      this.showToast('🎉 Application submitted! +10 pts earned!', 'success')
      this.applyForm = { name: '', email: '', message: '' }
    },

    // ========== POINTS ==========
    checkDailyLogin() {
      const today = new Date().toDateString()
      const lastLogin = localStorage.getItem('pp_lastLogin')
      if (lastLogin !== today) {
        localStorage.setItem('pp_lastLogin', today)
        this.addPoints(5, '📅 Daily Login Reward')
        this.earnMethods[0].claimed = true
      }
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
    addPoints(pts, label) {
      this.userPoints += pts
      this.transactions.unshift({
        id: Date.now(), icon: '⭐', label,
        date: new Date().toLocaleDateString(), type: 'earn', points: pts
      })
      localStorage.setItem('pp_points', this.userPoints)
    },

    showToast(message, type = 'info') {
      this.toast = { show: true, message, type }
      setTimeout(() => { this.toast.show = false }, 3000)
    }
  },

  mounted() {
    // Auto-login if saved session
    const savedUser = localStorage.getItem('pp_user')
    const savedToken = localStorage.getItem('pp_token')
    const savedPoints = localStorage.getItem('pp_points')
    if (savedUser && savedToken) {
      this.currentUser = JSON.parse(savedUser)
      this.userToken = savedToken
      this.userPoints = parseInt(savedPoints || 0)
      this.editForm = { ...this.currentUser }
      this.checkDailyLogin()
    }
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
#app { min-height: 100vh; }

/* ===== AUTH ===== */
.auth-wrapper { min-height: 100vh; display: flex; align-items: center; justify-content: center; padding: 20px; background: radial-gradient(ellipse at top, #1c1c3a, var(--bg)); }
.auth-card { background: var(--surface); border: 1px solid var(--border); border-radius: 20px; padding: 32px; width: 100%; max-width: 400px; }
.auth-brand { text-align: center; margin-bottom: 28px; }
.auth-brand span { font-size: 2.5rem; display: block; margin-bottom: 8px; }
.auth-brand h1 { font-size: 1.6rem; font-weight: 700; color: var(--accent); }
.auth-brand p { color: var(--text-muted); font-size: 0.85rem; margin-top: 4px; }
.auth-title { font-size: 1.2rem; font-weight: 700; margin-bottom: 20px; }
.btn-auth { width: 100%; background: var(--accent); color: white; border: none; padding: 14px; border-radius: var(--radius); cursor: pointer; font-weight: 700; font-size: 1rem; margin-top: 8px; transition: opacity 0.2s; }
.btn-auth:disabled { opacity: 0.6; cursor: not-allowed; }
.auth-switch { text-align: center; margin-top: 16px; font-size: 0.85rem; color: var(--text-muted); }
.auth-switch span { color: var(--accent); cursor: pointer; font-weight: 600; }
.auth-error { color: var(--red); font-size: 0.85rem; text-align: center; margin-top: 12px; }

/* ===== NAVBAR ===== */
.navbar { display: flex; justify-content: space-between; align-items: center; padding: 14px 20px; background: var(--surface); border-bottom: 1px solid var(--border); position: sticky; top: 0; z-index: 100; }
.nav-brand { display: flex; align-items: center; gap: 8px; font-weight: 700; font-size: 1.2rem; }
.brand-icon { font-size: 1.4rem; }
.nav-actions { display: flex; align-items: center; gap: 10px; }
.btn-connect { background: var(--accent); color: white; border: none; padding: 8px 14px; border-radius: 999px; cursor: pointer; font-weight: 600; font-size: 0.82rem; display: flex; align-items: center; gap: 6px; }
.wallet-info { background: var(--surface2); border: 1px solid var(--border); padding: 6px 12px; border-radius: 999px; cursor: pointer; display: flex; align-items: center; gap: 8px; font-size: 0.82rem; }
.wallet-points { color: var(--yellow); font-weight: 700; }
.wallet-addr { color: var(--text-muted); font-family: 'DM Mono', monospace; font-size: 0.75rem; }

/* User Menu */
.user-menu { position: relative; cursor: pointer; }
.user-avatar { width: 36px; height: 36px; border-radius: 50%; background: var(--accent); display: flex; align-items: center; justify-content: center; font-weight: 700; font-size: 0.9rem; }
.user-dropdown { position: absolute; right: 0; top: 44px; background: var(--surface); border: 1px solid var(--border); border-radius: var(--radius); padding: 12px; min-width: 180px; z-index: 200; }
.dropdown-name { font-weight: 700; font-size: 0.9rem; }
.dropdown-email { font-size: 0.75rem; color: var(--text-muted); margin-bottom: 10px; }
.user-dropdown hr { border-color: var(--border); margin-bottom: 8px; }
.user-dropdown button { display: block; width: 100%; text-align: left; background: none; border: none; color: var(--text); padding: 8px; border-radius: 8px; cursor: pointer; font-size: 0.85rem; }
.user-dropdown button:hover { background: var(--surface2); }
.logout-btn { color: var(--red) !important; }

/* ===== PAGE ===== */
.page-container { padding: 20px; max-width: 900px; margin: 0 auto; padding-bottom: 100px; }
.hero { text-align: center; padding: 40px 0 30px; }
.hero-title { font-size: 2rem; font-weight: 700; margin-bottom: 10px; }
.accent { color: var(--accent); }
.hero-sub { color: var(--text-muted); margin-bottom: 24px; }
.search-bar { display: flex; gap: 8px; max-width: 500px; margin: 0 auto 16px; }
.search-input { flex: 1; background: var(--surface2); border: 1px solid var(--border); color: var(--text); padding: 12px 16px; border-radius: var(--radius); font-size: 0.95rem; }
.btn-search { background: var(--accent); color: white; border: none; padding: 12px 20px; border-radius: var(--radius); cursor: pointer; font-weight: 600; }
.filters { display: flex; gap: 8px; flex-wrap: wrap; justify-content: center; }
.filter-select { background: var(--surface2); border: 1px solid var(--border); color: var(--text); padding: 10px 14px; border-radius: var(--radius); font-size: 0.85rem; cursor: pointer; }

/* Cards */
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

/* Profile */
.profile-page { max-width: 600px; margin: 0 auto; }
.profile-card { background: var(--surface); border: 1px solid var(--border); border-radius: var(--radius); padding: 20px; display: flex; gap: 16px; align-items: center; margin-bottom: 20px; }
.profile-avatar { width: 60px; height: 60px; border-radius: 50%; background: var(--accent); display: flex; align-items: center; justify-content: center; font-size: 1.5rem; font-weight: 700; flex-shrink: 0; }
.profile-info h3 { font-size: 1.1rem; font-weight: 700; }
.profile-info p { font-size: 0.82rem; color: var(--text-muted); margin-top: 3px; }
.profile-stats { display: grid; grid-template-columns: repeat(3, 1fr); gap: 12px; margin-bottom: 24px; }
.stat-box { background: var(--surface); border: 1px solid var(--border); border-radius: var(--radius); padding: 16px; text-align: center; }
.stat-num { display: block; font-size: 1.8rem; font-weight: 700; color: var(--accent); }
.stat-label { font-size: 0.75rem; color: var(--text-muted); }

/* Modal */
.modal-overlay { position: fixed; inset: 0; background: rgba(0,0,0,0.7); display: flex; align-items: center; justify-content: center; z-index: 200; padding: 20px; }
.modal { background: var(--surface); border: 1px solid var(--border); border-radius: var(--radius); padding: 24px; width: 100%; max-width: 440px; }
.modal h2 { font-size: 1.2rem; margin-bottom: 4px; }
.modal-company { color: var(--text-muted); font-size: 0.85rem; margin-bottom: 16px; }
.modal-actions { display: flex; gap: 10px; justify-content: flex-end; margin-top: 16px; }
.btn-cancel { background: none; border: 1px solid var(--border); color: var(--text-muted); padding: 10px 18px; border-radius: 10px; cursor: pointer; }
.btn-submit { background: var(--accent); color: white; border: none; padding: 10px 20px; border-radius: 10px; cursor: pointer; font-weight: 600; }

/* Form */
.form-group { margin-bottom: 14px; }
.form-group label { display: block; font-size: 0.85rem; margin-bottom: 6px; color: var(--text-muted); }
.form-input { width: 100%; background: var(--surface2); border: 1px solid var(--border); color: var(--text); padding: 10px 14px; border-radius: 10px; font-size: 0.9rem; font-family: 'Space Grotesk', sans-serif; }
.form-textarea { width: 100%; background: var(--surface2); border: 1px solid var(--border); color: var(--text); padding: 10px 14px; border-radius: 10px; font-size: 0.9rem; height: 80px; resize: vertical; font-family: 'Space Grotesk', sans-serif; }
input:focus, select:focus, textarea:focus { outline: 2px solid var(--accent); border-color: var(--accent); }

/* Wallet */
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

/* Bottom Nav */
.bottom-nav { position: fixed; bottom: 0; left: 0; right: 0; background: var(--surface); border-top: 1px solid var(--border); display: flex; z-index: 100; }
.nav-btn { flex: 1; background: none; border: none; color: var(--text-muted); padding: 12px; cursor: pointer; display: flex; flex-direction: column; align-items: center; gap: 2px; font-size: 1.3rem; transition: color 0.2s; }
.nav-btn small { font-size: 0.7rem; font-family: 'Space Grotesk', sans-serif; }
.nav-btn.active { color: var(--accent); }

/* Toast */
.toast { position: fixed; bottom: 90px; left: 50%; transform: translateX(-50%); padding: 12px 24px; border-radius: 999px; font-weight: 600; font-size: 0.9rem; z-index: 300; animation: slideUp 0.3s ease; white-space: nowrap; }
.toast.success { background: var(--green); color: white; }
.toast.error { background: var(--red); color: white; }
.toast.info { background: var(--accent); color: white; }
@keyframes slideUp { from { opacity: 0; transform: translateX(-50%) translateY(20px); } to { opacity: 1; transform: translateX(-50%) translateY(0); } }
</style>