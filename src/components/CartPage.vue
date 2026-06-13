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

onMounted(() => {
  const savedCart = localStorage.getItem('modilo_cart');
  if (savedCart) {
    try {
      cart.value = JSON.parse(savedCart);
    } catch (e) {
      console.error(e);
    }
  }
});

function saveCart() {
  localStorage.setItem('modilo_cart', JSON.stringify(cart.value));
  window.dispatchEvent(new CustomEvent('cart-updated'));
}

function updateQuantity(item, amount) {
  item.quantity += amount;
  if (item.quantity <= 0) {
    cart.value = cart.value.filter(i => i !== item);
  }
  saveCart();
}

function removeFromCart(item) {
  cart.value = cart.value.filter(i => i !== item);
  saveCart();
}

const cartItemCount = computed(() => {
  return cart.value.reduce((total, item) => total + item.quantity, 0);
});

const cartTotalPrice = computed(() => {
  return cart.value.reduce((total, item) => total + (item.price * item.quantity), 0);
});

function getWhatsAppCheckoutLink() {
  let message = `Bonjour Modilo, je souhaite commander les articles suivants :\n\n`;
  cart.value.forEach(item => {
    message += `• ${item.quantity}x ${item.title} (Taille: ${item.size}${item.color ? ', Couleur: ' + item.color : ''}) - ${item.price} MAD chacun\n`;
  });
  message += `\nTotal : ${cartTotalPrice.value} MAD\n\nMerci de valider ma commande.`;
  return `https://wa.me/+212630257293?text=${encodeURIComponent(message)}`;
}
</script>

<template>
  <div class="cart-page-container">
    <div class="cart-header">
      <h1 class="page-title">Votre Panier</h1>
      <div class="accent-line"></div>
    </div>

    <div v-if="cart.length === 0" class="cart-empty-state">
      <div class="empty-icon-wrapper">
        <svg viewBox="0 0 24 24" width="64" height="64" fill="none" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round">
          <circle cx="9" cy="21" r="1"></circle>
          <circle cx="20" cy="21" r="1"></circle>
          <path d="M1 1h4l2.68 13.39a2 2 0 0 0 2 1.61h9.72a2 2 0 0 0 2-1.61L23 6H6"></path>
        </svg>
      </div>
      <h2 class="empty-title">Votre panier est vide</h2>
      <p class="empty-text">Découvrez notre collection de vêtements premium conçus exclusivement pour l'homme d'action.</p>
      <a href="/produits" class="btn-shop">Découvrir la Boutique</a>
    </div>

    <div v-else class="cart-grid">
      <!-- Items List -->
      <div class="cart-items-section">
        <div v-for="item in cart" :key="item.id + '-' + item.size + '-' + item.color" class="cart-page-item">
          <div class="item-img-wrapper">
            <img :src="imageMap[item.image] || item.image" :alt="item.title" :style="{ filter: item.cssFilter || 'none' }" />
          </div>
          
          <div class="item-details">
            <h3 class="item-title">{{ item.title }}</h3>
            <p class="item-meta">
              <span>Taille: <strong>{{ item.size }}</strong></span>
              <span v-if="item.color" class="meta-divider">|</span>
              <span v-if="item.color">Couleur: <strong>{{ item.color }}</strong></span>
            </p>
            <p class="item-unit-price">{{ item.price }} MAD</p>
          </div>

          <div class="item-qty-and-subtotal">
            <div class="qty-control">
              <button @click="updateQuantity(item, -1)" class="qty-btn" aria-label="Diminuer">-</button>
              <span class="qty-val">{{ item.quantity }}</span>
              <button @click="updateQuantity(item, 1)" class="qty-btn" aria-label="Augmenter">+</button>
            </div>
            
            <div class="subtotal-and-delete">
              <span class="item-subtotal">{{ item.price * item.quantity }} MAD</span>
              <button @click="removeFromCart(item)" class="delete-btn" aria-label="Supprimer cet article">
                <svg viewBox="0 0 24 24" width="18" height="18" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
                  <polyline points="3 6 5 6 21 6"></polyline>
                  <path d="M19 6v14a2 2 0 0 1-2 2H7a2 2 0 0 1-2-2V6m3 0V4a2 2 0 0 1 2-2h4a2 2 0 0 1 2 2v2"></path>
                </svg>
              </button>
            </div>
          </div>
        </div>
      </div>

      <!-- Checkout Summary Card -->
      <div class="cart-summary-section">
        <!-- Order Summary Card -->
        <div class="summary-card">
          <h3 class="summary-title">Résumé de la commande</h3>
          
          <div class="summary-row">
            <span>Articles ({{ cartItemCount }})</span>
            <span>{{ cartTotalPrice }} MAD</span>
          </div>
          
          <div class="summary-row">
            <span>Livraison</span>
            <span class="free-shipping">Gratuite</span>
          </div>

          <div class="summary-divider"></div>

          <div class="summary-row total-row">
            <span>Total</span>
            <span class="total-price">{{ cartTotalPrice }} MAD</span>
          </div>

          <a href="/commande" class="btn-checkout-next">
            <span>Passer à la livraison</span>
            <svg viewBox="0 0 24 24" width="20" height="20" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
              <line x1="5" y1="12" x2="19" y2="12"></line>
              <polyline points="12 5 19 12 12 19"></polyline>
            </svg>
          </a>

          <a href="/produits" class="btn-continue-shopping">
            Continuer mes achats
          </a>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.cart-page-container {
  max-width: 1200px;
  margin: 0 auto;
  padding: 140px 20px 80px;
  min-height: 70vh;
  color: var(--text-primary);
}

.cart-header {
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
  background: var(--accent-indigo);
  margin: 15px auto 0;
  border-radius: 99px;
}

/* Empty State */
.cart-empty-state {
  text-align: center;
  max-width: 500px;
  margin: 80px auto;
  padding: 40px;
  background: var(--bg-secondary);
  border: 1px solid var(--card-border);
  border-radius: 16px;
  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.03);
}

.empty-icon-wrapper {
  color: var(--text-muted);
  margin-bottom: 24px;
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
  background: var(--accent-indigo);
  color: #ffffff;
  text-decoration: none;
  font-weight: 700;
  font-size: 0.95rem;
  padding: 14px 30px;
  border-radius: 99px;
  transition: all 0.25s;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
}

.btn-shop:hover {
  background: var(--accent-primary-hover);
  transform: translateY(-1px);
  box-shadow: 0 6px 16px rgba(0, 0, 0, 0.15);
}

/* Grid Layout */
.cart-grid {
  display: grid;
  grid-template-columns: 1fr 380px;
  gap: 40px;
  align-items: start;
}

@media (max-width: 992px) {
  .cart-grid {
    grid-template-columns: 1fr;
    gap: 30px;
  }
}

/* Items List */
.cart-items-section {
  display: flex;
  flex-direction: column;
  gap: 20px;
}

.cart-page-item {
  display: grid;
  grid-template-columns: 100px 1fr auto;
  gap: 25px;
  align-items: center;
  background: var(--bg-secondary);
  border: 1px solid var(--card-border);
  border-radius: 14px;
  padding: 20px;
  box-shadow: 0 4px 15px rgba(0, 0, 0, 0.02);
}

@media (max-width: 600px) {
  .cart-page-item {
    grid-template-columns: 80px 1fr;
    gap: 15px;
    padding: 15px;
  }
}

.item-img-wrapper {
  aspect-ratio: 1;
  background-color: var(--bg-primary);
  border: 1px solid var(--card-border);
  border-radius: 8px;
  overflow: hidden;
  display: flex;
  align-items: center;
  justify-content: center;
}

.item-img-wrapper img {
  width: 100%;
  height: 100%;
  object-fit: cover;
}

.item-details {
  display: flex;
  flex-direction: column;
  gap: 6px;
}

.item-title {
  font-size: 1.1rem;
  font-weight: 600;
  line-height: 1.4;
}

.item-meta {
  font-size: 0.85rem;
  color: var(--text-secondary);
}

.meta-divider {
  margin: 0 8px;
  color: var(--text-muted);
}

.item-unit-price {
  font-size: 0.95rem;
  font-weight: 700;
  color: var(--text-secondary);
}

.item-qty-and-subtotal {
  display: flex;
  align-items: center;
  gap: 30px;
}

@media (max-width: 600px) {
  .item-qty-and-subtotal {
    grid-column: 1 / -1;
    justify-content: space-between;
    border-top: 1px solid var(--card-border);
    padding-top: 12px;
    margin-top: 5px;
  }
}

.qty-control {
  display: flex;
  align-items: center;
  background: var(--bg-primary);
  border: 1px solid var(--card-border);
  border-radius: 8px;
  overflow: hidden;
}

.qty-btn {
  background: none;
  border: none;
  width: 32px;
  height: 32px;
  cursor: pointer;
  font-weight: 600;
  color: var(--text-primary);
  transition: background-color 0.2s;
}

.qty-btn:hover {
  background-color: rgba(0, 0, 0, 0.05);
}

.qty-val {
  width: 35px;
  text-align: center;
  font-size: 0.9rem;
  font-weight: 600;
}

.subtotal-and-delete {
  display: flex;
  align-items: center;
  gap: 20px;
}

.item-subtotal {
  font-size: 1.15rem;
  font-weight: 700;
  min-width: 90px;
  text-align: right;
}

.delete-btn {
  background: none;
  border: none;
  color: var(--text-muted);
  cursor: pointer;
  padding: 6px;
  transition: color 0.2s;
}

.delete-btn:hover {
  color: #c92a2a;
}

/* Delivery Info Form */
.delivery-info-card {
  background: var(--bg-secondary);
  border: 1px solid var(--card-border);
  border-radius: 16px;
  padding: 30px;
  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.02);
  margin-bottom: 20px;
}

.delivery-info-title {
  font-family: var(--font-heading);
  font-size: 1.3rem;
  font-weight: 700;
  margin-bottom: 8px;
}

.delivery-info-desc {
  font-size: 0.85rem;
  color: var(--text-secondary);
  margin-bottom: 20px;
  line-height: 1.4;
}

.form-group {
  margin-bottom: 18px;
  display: flex;
  flex-direction: column;
  gap: 6px;
}

.form-group:last-child {
  margin-bottom: 0;
}

.form-label {
  font-size: 0.82rem;
  font-weight: 600;
  color: var(--text-primary);
  text-transform: uppercase;
  letter-spacing: 0.5px;
}

.form-input {
  background: var(--bg-primary);
  border: 1px solid var(--card-border);
  color: var(--text-primary);
  padding: 12px 16px;
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
  box-shadow: 0 0 0 2px rgba(0, 0, 0, 0.04);
}

.form-input.input-error {
  border-color: #c92a2a;
}

.error-text {
  font-size: 0.78rem;
  color: #c92a2a;
  font-weight: 500;
  margin-top: 2px;
}

/* Summary Card */
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

.btn-checkout-next {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 10px;
  background: var(--accent-primary);
  color: #ffffff;
  text-decoration: none;
  font-weight: 700;
  font-size: 1rem;
  padding: 15px;
  border-radius: 99px;
  text-align: center;
  transition: all 0.25s;
  box-shadow: 0 4px 16px rgba(0, 0, 0, 0.1);
  margin-bottom: 15px;
  border: none;
  cursor: pointer;
}

.btn-checkout-next:hover {
  background: var(--accent-primary-hover);
  transform: translateY(-1px);
  box-shadow: 0 6px 22px rgba(0, 0, 0, 0.15);
}

.btn-continue-shopping {
  display: block;
  text-align: center;
  color: var(--text-secondary);
  text-decoration: none;
  font-size: 0.9rem;
  font-weight: 600;
  transition: color 0.2s;
}

.btn-continue-shopping:hover {
  color: var(--accent-indigo);
  text-decoration: underline;
}
</style>
