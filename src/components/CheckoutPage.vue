<script setup>
import { ref, computed, onMounted } from 'vue';

// Import local image assets for correct Vue bundle resolution
import hoodieImg from '../assets/hoodie_black.png';
import tshirtImg from '../assets/tshirt_white.png';
import backpackImg from '../assets/backpack_grey.png';
import totebagImg from '../assets/totebag_canvas.png';

const imageMap = {
  "../assets/hoodie_black.png": hoodieImg.src || hoodieImg,
  "../assets/tshirt_white.png": tshirtImg.src || tshirtImg,
  "../assets/backpack_grey.png": backpackImg.src || backpackImg,
  "../assets/totebag_canvas.png": totebagImg.src || totebagImg
};

const cart = ref([]);

const name = ref('');
const phone = ref('');
const selectedCityType = ref('tanger');
const customCity = ref('');
const address = ref('');

const errors = ref({
  name: '',
  phone: '',
  city: '',
  address: ''
});

onMounted(() => {
  const savedCart = localStorage.getItem('modilo_cart');
  if (savedCart) {
    try {
      cart.value = JSON.parse(savedCart);
    } catch (e) {
      console.error(e);
    }
  }

  // Load customer details
  name.value = localStorage.getItem('modilo_cust_name') || '';
  phone.value = localStorage.getItem('modilo_cust_phone') || '';
  selectedCityType.value = localStorage.getItem('modilo_cust_city_type') || 'tanger';
  customCity.value = localStorage.getItem('modilo_cust_city_custom') || '';
  address.value = localStorage.getItem('modilo_cust_address') || '';
});

function saveCustomerInfo() {
  localStorage.setItem('modilo_cust_name', name.value);
  localStorage.setItem('modilo_cust_phone', phone.value);
  localStorage.setItem('modilo_cust_city_type', selectedCityType.value);
  localStorage.setItem('modilo_cust_city_custom', customCity.value);
  localStorage.setItem('modilo_cust_address', address.value);

  // Clear errors when typing/selecting
  if (name.value.trim()) errors.value.name = '';
  if (phone.value.trim()) errors.value.phone = '';
  if (selectedCityType.value === 'tanger' || customCity.value.trim()) errors.value.city = '';
  if (address.value.trim()) errors.value.address = '';
}

const cartItemCount = computed(() => {
  return cart.value.reduce((total, item) => total + item.quantity, 0);
});

const cartTotalPrice = computed(() => {
  return cart.value.reduce((total, item) => total + (item.price * item.quantity), 0);
});

const shippingFee = computed(() => {
  return selectedCityType.value === 'tanger' ? 0 : 45;
});

const totalWithShipping = computed(() => {
  return cartTotalPrice.value + shippingFee.value;
});

function validateForm() {
  let isValid = true;
  errors.value = { name: '', phone: '', city: '', address: '' };

  if (!name.value.trim()) {
    errors.value.name = 'Veuillez saisir votre nom complet.';
    isValid = false;
  }

  const trimmedPhone = phone.value.trim().replace(/\s/g, '');
  if (!trimmedPhone) {
    errors.value.phone = 'Veuillez saisir votre numéro de téléphone.';
    isValid = false;
  } else if (!/^(?:\+212|0)[5-7]\d{8}$/.test(trimmedPhone)) {
    errors.value.phone = 'Veuillez saisir un numéro de téléphone marocain valide (ex: 0612345678).';
    isValid = false;
  }

  if (selectedCityType.value === 'autre' && !customCity.value.trim()) {
    errors.value.city = 'Veuillez saisir le nom de votre ville.';
    isValid = false;
  }

  if (!address.value.trim()) {
    errors.value.address = 'Veuillez saisir votre adresse de livraison.';
    isValid = false;
  }

  return isValid;
}

function handleCheckout() {
  if (!validateForm()) {
    // Scroll to the first error form-group with margin-top offset
    const firstErrorGroup = document.querySelector('.form-group:has(.input-error), .form-group:has(.error-text)');
    if (firstErrorGroup) {
      firstErrorGroup.scrollIntoView({ behavior: 'smooth', block: 'start' });
    } else {
      const firstErrorEl = document.querySelector('.error-text');
      if (firstErrorEl) {
        firstErrorEl.scrollIntoView({ behavior: 'smooth', block: 'center' });
      }
    }
    return;
  }

  const finalCity = selectedCityType.value === 'tanger' ? 'Tanger' : customCity.value.trim();
  const shippingText = selectedCityType.value === 'tanger' ? 'Gratuite' : '45 MAD';

  let message = `Bonjour Modilo, je souhaite commander les articles suivants :\n\n`;
  cart.value.forEach(item => {
    message += `• ${item.quantity}x ${item.title} (Taille: ${item.size}${item.color ? ', Couleur: ' + item.color : ''}) - ${item.price} MAD chacun\n`;
  });
  message += `\nSous-total : ${cartTotalPrice.value} MAD\n`;
  message += `Frais de livraison : ${shippingText}\n`;
  message += `Total : ${totalWithShipping.value} MAD\n\n`;
  message += `*Informations de livraison :*\n`;
  message += `• *Nom complet :* ${name.value.trim()}\n`;
  message += `• *Téléphone :* ${phone.value.trim()}\n`;
  message += `• *Ville :* ${finalCity}\n`;
  message += `• *Adresse :* ${address.value.trim()}\n\n`;
  message += `Merci de valider ma commande.`;

  const link = `https://wa.me/+212630257293?text=${encodeURIComponent(message)}`;
  window.open(link, '_blank');
}
</script>

<template>
  <div class="checkout-page-container">
    <div class="checkout-header">
      <h1 class="page-title">Livraison</h1>
      <div class="accent-line"></div>
    </div>

    <div v-if="cart.length === 0" class="checkout-empty-state">
      <h2 class="empty-title">Votre panier est vide</h2>
      <p class="empty-text">Veuillez ajouter des articles à votre panier avant de passer à la livraison.</p>
      <a href="/products" class="btn-shop">Découvrir la Boutique</a>
    </div>

    <div v-else class="checkout-grid">
      <!-- Left: Delivery Form -->
      <div class="checkout-form-section">
        <div class="delivery-info-card">
          <h3 class="delivery-info-title">Informations de livraison</h3>
          <p class="delivery-info-desc">Complétez vos coordonnées pour finaliser la commande sur WhatsApp.</p>
          
          <div class="form-group">
            <label for="fullname" class="form-label">Nom Complet *</label>
            <div class="input-icon-wrapper">
              <svg class="input-icon" viewBox="0 0 24 24" width="16" height="16" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
                <path d="M20 21v-2a4 4 0 0 0-4-4H8a4 4 0 0 0-4 4v2"></path>
                <circle cx="12" cy="7" r="4"></circle>
              </svg>
              <input 
                type="text" 
                id="fullname" 
                v-model="name" 
                @input="saveCustomerInfo" 
                placeholder="Ex: Mohammed Alami" 
                :class="['form-input', 'has-icon', {'input-error': errors.name}]" 
              />
            </div>
            <span v-if="errors.name" class="error-text">{{ errors.name }}</span>
          </div>

          <div class="form-group">
            <label for="phone" class="form-label">Téléphone *</label>
            <div class="input-icon-wrapper">
              <svg class="input-icon" viewBox="0 0 24 24" width="16" height="16" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
                <path d="M22 16.92v3a2 2 0 0 1-2.18 2 19.79 19.79 0 0 1-8.63-3.07 19.5 19.5 0 0 1-6-6 19.79 19.79 0 0 1-3.07-8.67A2 2 0 0 1 4.11 2h3a2 2 0 0 1 2 1.72 12.84 12.84 0 0 0 .7 2.81 2 2 0 0 1-.45 2.11L8.09 9.91a16 16 0 0 0 6 6l1.27-1.27a2 2 0 0 1 2.11-.45 12.84 12.84 0 0 0 2.81.7A2 2 0 0 1 22 16.92z"></path>
              </svg>
              <input 
                type="tel" 
                id="phone" 
                v-model="phone" 
                @input="saveCustomerInfo" 
                placeholder="Ex: 0612345678" 
                :class="['form-input', 'has-icon', {'input-error': errors.phone}]" 
              />
            </div>
            <span v-if="errors.phone" class="error-text">{{ errors.phone }}</span>
          </div>

          <div class="form-group">
            <label for="city-select" class="form-label">Ville *</label>
            <div class="input-icon-wrapper">
              <svg class="input-icon" viewBox="0 0 24 24" width="16" height="16" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
                <path d="M21 10c0 7-9 13-9 13s-9-6-9-13a9 9 0 0 1 18 0z"></path>
                <circle cx="12" cy="10" r="3"></circle>
              </svg>
              <select 
                id="city-select" 
                v-model="selectedCityType" 
                @change="saveCustomerInfo" 
                class="form-select has-icon"
              >
                <option value="tanger">Tanger (Livraison Gratuite)</option>
                <option value="autre">Autre ville du Maroc (Livraison: 45 MAD)</option>
              </select>
            </div>
          </div>

          <div v-if="selectedCityType === 'autre'" class="form-group slide-fade-in">
            <label for="custom-city" class="form-label">Nom de la ville *</label>
            <div class="input-icon-wrapper">
              <svg class="input-icon" viewBox="0 0 24 24" width="16" height="16" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
                <circle cx="12" cy="12" r="10"></circle>
                <line x1="2" y1="12" x2="22" y2="12"></line>
                <path d="M12 2a15.3 15.3 0 0 1 4 10 15.3 15.3 0 0 1-4 10 15.3 15.3 0 0 1-4-10 15.3 15.3 0 0 1 4-10z"></path>
              </svg>
              <input 
                type="text" 
                id="custom-city" 
                v-model="customCity" 
                @input="saveCustomerInfo" 
                placeholder="Ex: Casablanca, Rabat, Marrakech..." 
                :class="['form-input', 'has-icon', {'input-error': errors.city}]" 
              />
            </div>
            <span v-if="errors.city" class="error-text">{{ errors.city }}</span>
          </div>

          <div class="form-group">
            <label for="address" class="form-label">Adresse de livraison *</label>
            <div class="input-icon-wrapper">
              <svg class="input-icon" style="top: 14px;" viewBox="0 0 24 24" width="16" height="16" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
                <path d="M3 9l9-7 9 7v11a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2z"></path>
                <polyline points="9 22 9 12 15 12 15 22"></polyline>
              </svg>
              <textarea 
                id="address" 
                v-model="address" 
                @input="saveCustomerInfo" 
                placeholder="Ex: Boulevard Mohamed V, N° 12, Appt 4" 
                rows="3"
                :class="['form-input', 'has-icon', {'input-error': errors.address}]" 
              ></textarea>
            </div>
            <span v-if="errors.address" class="error-text">{{ errors.address }}</span>
          </div>
        </div>

        <a href="/cart" class="btn-back-to-cart">
          <svg viewBox="0 0 24 24" width="16" height="16" fill="none" stroke="currentColor" stroke-width="2">
            <line x1="19" y1="12" x2="5" y2="12"></line>
            <polyline points="12 19 5 12 12 5"></polyline>
          </svg>
          Retour au panier
        </a>
      </div>

      <!-- Right: Summary Card -->
      <div class="checkout-summary-section">
        <!-- Order Items Summary -->
        <div class="items-summary-card">
          <h3 class="summary-title">Articles</h3>
          <div class="items-list-compact">
            <div v-for="item in cart" :key="item.id + '-' + item.size + '-' + item.color" class="compact-item">
              <div class="compact-item-img">
                <img :src="imageMap[item.image] || item.image" :alt="item.title" :style="{ filter: item.cssFilter || 'none' }" />
                <span class="compact-item-qty">{{ item.quantity }}</span>
              </div>
              <div class="compact-item-details">
                <span class="compact-item-title">{{ item.title }}</span>
                <span class="compact-item-meta">{{ item.size }} <span v-if="item.color">/ {{ item.color }}</span></span>
              </div>
              <span class="compact-item-price">{{ item.price * item.quantity }} MAD</span>
            </div>
          </div>
        </div>

        <!-- Totals Card -->
        <div class="summary-card">
          <h3 class="summary-title">Résumé</h3>
          
          <div class="summary-row">
            <span>Sous-total</span>
            <span>{{ cartTotalPrice }} MAD</span>
          </div>
          
          <div class="summary-row">
            <span>Livraison</span>
            <span v-if="shippingFee === 0" class="free-shipping">Gratuite</span>
            <span v-else>{{ shippingFee }} MAD</span>
          </div>

          <div class="summary-divider"></div>

          <div class="summary-row total-row">
            <span>Total</span>
            <span class="total-price">{{ totalWithShipping }} MAD</span>
          </div>

          <button @click="handleCheckout" class="btn-checkout-whatsapp">
            <svg viewBox="0 0 24 24" width="20" height="20" fill="currentColor">
              <path d="M12.012 2c-5.506 0-9.989 4.478-9.99 9.984a9.96 9.96 0 0 0 1.333 4.982L2 22l5.233-1.371a9.936 9.936 0 0 0 4.779 1.21h.005c5.505 0 9.988-4.479 9.989-9.986 0-2.67-1.037-5.178-2.922-7.062A9.925 9.925 0 0 0 12.012 2zm5.835 14.165c-.32.9-.1.9-.1.9s-.427 1.099-1.637 1.328c-1.21.23-2.56.096-4.576-.718-2.016-.814-3.526-2.585-4.464-3.878-.938-1.293-1.579-2.738-1.579-4.22 0-1.48.718-2.203 1.026-2.502.308-.299.82-.455 1.077-.455.257 0 .492-.01.705.01.214.02.487-.08.761.564.274.645.94 2.274 1.017 2.43.077.157.12.338.017.532-.102.193-.154.314-.308.492-.154.177-.325.298-.462.46-.153.18-.316.378-.137.684.18.307.8 1.314 1.718 2.128.918.814 1.692 1.065 1.992 1.185.3.12.478.105.658-.1.18-.206.77-.895.974-1.2.206-.307.41-.258.693-.153.282.105 1.795.847 2.103.992.308.145.513.218.59.35.077.133.077.77-.243 1.67z"/>
            </svg>
            <span>Commander via WhatsApp</span>
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.checkout-page-container {
  max-width: 1200px;
  margin: 0 auto;
  padding: 140px 20px 80px;
  min-height: 70vh;
  color: var(--text-primary);
}

.checkout-header {
  text-align: center;
  margin-bottom: 50px;
}

.page-title {
  font-family: var(--font-heading);
  font-size: 2.5rem;
  font-weight: 800;
  letter-spacing: 0.5px;
}

.accent-line {
  width: 60px;
  height: 4px;
  background: var(--accent-primary);
  margin: 15px auto 0;
  border-radius: 99px;
}

.checkout-empty-state {
  text-align: center;
  max-width: 500px;
  margin: 80px auto;
  padding: 40px;
  background: var(--bg-secondary);
  border: 1px solid var(--card-border);
  border-radius: 16px;
  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.03);
}

.empty-title {
  font-family: var(--font-heading);
  font-size: 1.5rem;
  font-weight: 700;
  margin-bottom: 12px;
}

.empty-text {
  color: var(--text-secondary);
  font-size: 0.95rem;
  line-height: 1.6;
  margin-bottom: 30px;
}

.btn-shop {
  display: inline-block;
  background: var(--accent-primary);
  color: #ffffff;
  text-decoration: none;
  font-weight: 700;
  font-size: 0.95rem;
  padding: 14px 30px;
  border-radius: 99px;
  transition: all 0.25s;
}

.btn-shop:hover {
  background: var(--accent-primary-hover);
  transform: translateY(-1px);
}

.checkout-grid {
  display: grid;
  grid-template-columns: 1fr 380px;
  gap: 40px;
  align-items: start;
}

@media (max-width: 992px) {
  .checkout-grid {
    grid-template-columns: 1fr;
    gap: 30px;
  }
}

.checkout-form-section {
  display: flex;
  flex-direction: column;
  gap: 20px;
}

.delivery-info-card {
  background: var(--bg-secondary);
  border: 1px solid var(--card-border);
  border-radius: 20px;
  padding: 40px;
  box-shadow: 0 20px 40px rgba(0, 0, 0, 0.03);
  position: relative;
  overflow: hidden;
}

.delivery-info-card::before {
  content: '';
  position: absolute;
  top: 0;
  left: 0;
  width: 4px;
  height: 100%;
  background: var(--accent-primary);
}

.delivery-info-title {
  font-family: var(--font-heading);
  font-size: 1.45rem;
  font-weight: 800;
  margin-bottom: 8px;
  letter-spacing: -0.3px;
}

.delivery-info-desc {
  font-size: 0.9rem;
  color: var(--text-secondary);
  margin-bottom: 30px;
  line-height: 1.5;
}

.form-group {
  margin-bottom: 20px;
  display: flex;
  flex-direction: column;
  gap: 8px;
  scroll-margin-top: 140px; /* Account for sticky header offset */
}

.form-group:last-child {
  margin-bottom: 0;
}

.form-label {
  font-size: 0.78rem;
  font-weight: 700;
  color: var(--text-primary);
  text-transform: uppercase;
  letter-spacing: 0.8px;
}

.input-icon-wrapper {
  position: relative;
  display: flex;
  align-items: center;
  width: 100%;
}

.input-icon {
  position: absolute;
  left: 16px;
  color: var(--text-secondary);
  pointer-events: none;
  transition: color 0.2s;
}

.form-input.has-icon, .form-select.has-icon {
  padding-left: 46px !important;
}

.input-icon-wrapper:focus-within .input-icon {
  color: var(--text-primary);
}

.form-input {
  background: var(--bg-primary);
  border: 1px solid var(--card-border);
  color: var(--text-primary);
  padding: 14px 16px;
  border-radius: 10px;
  font-family: inherit;
  font-size: 0.92rem;
  transition: border-color 0.2s, box-shadow 0.2s;
  width: 100%;
  outline: none;
  resize: vertical;
}

.form-input:focus {
  border-color: var(--text-primary);
  box-shadow: 0 0 0 3px rgba(0, 0, 0, 0.03);
}

.form-input.input-error {
  border-color: #c92a2a;
  background: rgba(199, 42, 42, 0.02);
}

.form-select {
  background: var(--bg-primary);
  border: 1px solid var(--card-border);
  color: var(--text-primary);
  padding: 14px 16px;
  border-radius: 10px;
  font-family: inherit;
  font-size: 0.92rem;
  transition: border-color 0.2s, box-shadow 0.2s;
  width: 100%;
  outline: none;
  cursor: pointer;
  appearance: none;
  background-image: url("data:image/svg+xml;charset=UTF-8,%3csvg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 24 24' fill='none' stroke='currentColor' stroke-width='2' stroke-linecap='round' stroke-linejoin='round'%3e%3cpolyline points='6 9 12 15 18 9'%3e%3c/polyline%3e%3c/svg%3e");
  background-repeat: no-repeat;
  background-position: right 16px center;
  background-size: 16px;
}

.form-select:focus {
  border-color: var(--text-primary);
  box-shadow: 0 0 0 3px rgba(0, 0, 0, 0.03);
}

.slide-fade-in {
  animation: slideFadeIn 0.3s ease-out forwards;
}

@keyframes slideFadeIn {
  from {
    opacity: 0;
    transform: translateY(-8px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

.error-text {
  font-size: 0.78rem;
  color: #c92a2a;
  font-weight: 500;
  margin-top: 2px;
}

.btn-back-to-cart {
  display: inline-flex;
  align-items: center;
  gap: 8px;
  color: var(--text-secondary);
  text-decoration: none;
  font-size: 0.9rem;
  font-weight: 600;
  transition: color 0.2s;
  align-self: flex-start;
}

.btn-back-to-cart:hover {
  color: var(--text-primary);
}

.checkout-summary-section {
  display: flex;
  flex-direction: column;
  gap: 20px;
}

.items-summary-card {
  background: var(--bg-secondary);
  border: 1px solid var(--card-border);
  border-radius: 16px;
  padding: 30px;
  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.02);
}

.items-list-compact {
  display: flex;
  flex-direction: column;
  gap: 15px;
}

.compact-item {
  display: grid;
  grid-template-columns: 50px 1fr auto;
  gap: 15px;
  align-items: center;
}

.compact-item-img {
  width: 50px;
  height: 50px;
  background-color: var(--bg-primary);
  border: 1px solid var(--card-border);
  border-radius: 6px;
  position: relative;
  display: flex;
  align-items: center;
  justify-content: center;
}

.compact-item-img img {
  width: 100%;
  height: 100%;
  object-fit: cover;
  border-radius: 5px;
}

.compact-item-qty {
  position: absolute;
  top: -6px;
  right: -6px;
  background: var(--accent-primary);
  color: #ffffff;
  font-size: 0.65rem;
  font-weight: 700;
  border-radius: 99px;
  width: 16px;
  height: 16px;
  display: flex;
  align-items: center;
  justify-content: center;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
  z-index: 2;
}

.compact-item-details {
  display: flex;
  flex-direction: column;
  gap: 2px;
}

.compact-item-title {
  font-size: 0.88rem;
  font-weight: 600;
  line-height: 1.3;
}

.compact-item-meta {
  font-size: 0.75rem;
  color: var(--text-secondary);
}

.compact-item-price {
  font-size: 0.88rem;
  font-weight: 700;
}

.summary-card {
  background: var(--bg-secondary);
  border: 1px solid var(--card-border);
  border-radius: 16px;
  padding: 30px;
  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.02);
}

.summary-title {
  font-family: var(--font-heading);
  font-size: 1.3rem;
  font-weight: 700;
  margin-bottom: 25px;
}

.summary-row {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 15px;
  font-size: 0.95rem;
  color: var(--text-secondary);
}

.free-shipping {
  color: #2b8a3e;
  font-weight: 600;
}

.summary-divider {
  height: 1px;
  background: var(--card-border);
  margin: 20px 0;
}

.total-row {
  font-size: 1.15rem;
  font-weight: 700;
  color: var(--text-primary);
  margin-bottom: 25px;
}

.total-price {
  font-size: 1.4rem;
  font-weight: 800;
  color: var(--accent-primary);
}

.btn-checkout-whatsapp {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 10px;
  background: #25d366;
  color: #ffffff;
  width: 100%;
  font-weight: 700;
  font-size: 1rem;
  padding: 15px;
  border-radius: 99px;
  text-align: center;
  transition: all 0.25s;
  box-shadow: 0 4px 16px rgba(37, 211, 102, 0.2);
  border: none;
  cursor: pointer;
  text-decoration: none;
}

.btn-checkout-whatsapp:hover {
  background: #20ba59;
  transform: translateY(-1px);
  box-shadow: 0 6px 22px rgba(37, 211, 102, 0.3);
}
</style>
