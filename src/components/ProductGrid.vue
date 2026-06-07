<script setup>
import { ref, computed, onMounted } from 'vue';
import productsData from '../data/products.json';

// Import local image assets for correct Vue bundle resolution
import hoodieImg from '../assets/hoodie_black.png';
import tshirtImg from '../assets/tshirt_white.png';
import backpackImg from '../assets/backpack_grey.png';
import totebagImg from '../assets/totebag_canvas.png';

// Map product image paths to resolved static assets
const imageMap = {
  "../assets/hoodie_black.png": hoodieImg.src || hoodieImg,
  "../assets/tshirt_white.png": tshirtImg.src || tshirtImg,
  "../assets/backpack_grey.png": backpackImg.src || backpackImg,
  "../assets/totebag_canvas.png": totebagImg.src || totebagImg
};

const products = productsData.map(p => ({
  ...p,
  resolvedImage: imageMap[p.image] || p.image
}));

const props = defineProps({
  showOnlyFeatured: {
    type: Boolean,
    default: false
  }
});

const categories = ['all', 'hoodie', 't-shirt'];
const categoryLabels = {
  all: 'Tout',
  hoodie: 'Sweats',
  't-shirt': 'T-Shirts'
};
const selectedCategory = ref('all');
const activeQuickView = ref(null);
const selectedSize = ref('');
const selectedColor = ref('');
const cart = ref([]);

const filteredProducts = computed(() => {
  if (selectedCategory.value === 'all') {
    return products;
  }
  return products.filter(p => p.category === selectedCategory.value);
});

const topSellingProducts = computed(() => {
  return products.filter(p => p.topSelling).slice(0, 5);
});

const tshirtProducts = computed(() => {
  return products.filter(p => p.category === 't-shirt').slice(0, 9);
});

onMounted(() => {
  const urlParams = new URLSearchParams(window.location.search);
  const catParam = urlParams.get('cat');
  if (catParam && categories.includes(catParam)) {
    selectedCategory.value = catParam;
  }
  
  // Load Cart
  const savedCart = localStorage.getItem('modilo_cart');
  if (savedCart) {
    try {
      cart.value = JSON.parse(savedCart);
    } catch (e) {
      console.error(e);
    }
  }

  // Expose global cart page redirect
  window.__openModiloCart = () => {
    window.location.href = '/cart';
  };

  // Dispatch loaded cart event to navbar
  window.dispatchEvent(new CustomEvent('cart-updated'));

  // Redirect to cart page if requested from URL
  if (urlParams.get('open_cart') === 'true') {
    window.location.href = '/cart';
  }
});

function selectCategory(cat) {
  selectedCategory.value = cat;
  const url = new URL(window.location);
  if (cat === 'all') {
    url.searchParams.delete('cat');
  } else {
    url.searchParams.set('cat', cat);
  }
  window.history.replaceState({}, '', url);
}

function openQuickView(product) {
  activeQuickView.value = product;
  selectedSize.value = product.variations && product.variations.sizes ? product.variations.sizes[0] : '';
  selectedColor.value = product.variations && product.variations.colors ? product.variations.colors[0] : '';
  document.body.style.overflow = 'hidden';
}

function closeQuickView() {
  activeQuickView.value = null;
  document.body.style.overflow = 'auto';
}

function saveCart() {
  localStorage.setItem('modilo_cart', JSON.stringify(cart.value));
  window.dispatchEvent(new CustomEvent('cart-updated'));
}

function addToCart(product) {
  const size = selectedSize.value || (product.variations && product.variations.sizes ? product.variations.sizes[0] : '');
  const color = selectedColor.value || (product.variations && product.variations.colors ? product.variations.colors[0] : '');
  
  const existingItem = cart.value.find(item => 
    item.id === product.id && item.size === size && item.color === color
  );
  
  if (existingItem) {
    existingItem.quantity += 1;
  } else {
    cart.value.push({
      id: product.id,
      title: product.title,
      price: product.price,
      size,
      color,
      quantity: 1,
      image: product.resolvedImage,
      cssFilter: product.cssFilter
    });
  }
  saveCart();
  closeQuickView();
  window.location.href = '/cart';
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

// Generate custom WhatsApp message link
function getWhatsAppLink(product) {
  const message = `Bonjour Modilo, je suis intéressé(e) par ${product.title} (${product.price} MAD). Pourriez-vous m’en dire plus ?`;
  return `https://wa.me/+212630257293?text=${encodeURIComponent(message)}`;
}

function quickAddToCart(product) {
  selectedSize.value = '';
  selectedColor.value = '';
  addToCart(product);
}
</script>

<template>
  <div class="product-section">
    <!-- T-Shirt Dedicated Section with Small Cards (New Arrivals) -->
    <div v-if="props.showOnlyFeatured" class="container margin-bottom tshirts-section">
      <div class="section-header">
        <h2 class="section-title">Nouveautés</h2>
        <div class="accent-line"></div>
      </div>
      
      <div class="small-grid">
        <div 
          v-for="product in tshirtProducts" 
          :key="'tshirt-' + product.id"
          class="small-card"
          @click="openQuickView(product)"
        >
          <div class="card-image-wrapper">
            <img :src="product.resolvedImage" :alt="product.title" class="card-image" :style="{ filter: product.cssFilter || 'none' }" />
            <div class="card-overlay">
              <span class="view-label">Aperçu Rapide</span>
            </div>
          </div>
          <div class="card-info">
            <h3 class="card-title">{{ product.title }}</h3>
            <div class="card-footer-row">
              <p class="card-price">{{ product.price }} MAD</p>
              <button class="card-add-icon-btn" @click.stop="quickAddToCart(product)" aria-label="Ajouter au panier">
                <span class="btn-icons">
                  <span class="plus-sign">+</span>
                  <svg viewBox="0 0 24 24" width="14" height="14" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
                    <circle cx="9" cy="21" r="1"></circle>
                    <circle cx="20" cy="21" r="1"></circle>
                    <path d="M1 1h4l2.68 13.39a2 2 0 0 0 2 1.61h9.72a2 2 0 0 0 2-1.61L23 6H6"></path>
                  </svg>
                </span>
              </button>
            </div>
          </div>
        </div>
      </div>
    </div>

    <!-- Main Shop / Categories Section -->
    <div v-if="!props.showOnlyFeatured" class="container">

      <!-- Category Filter Tabs -->
      <div class="tabs">
        <button 
          v-for="cat in categories" 
          :key="cat"
          :class="['tab-btn', { active: selectedCategory === cat }]"
          @click="selectCategory(cat)"
        >
          {{ categoryLabels[cat] || cat }}
        </button>
      </div>

      <!-- Products Grid -->
      <TransitionGroup name="fade-grid" tag="div" class="grid">
        <div 
          v-for="product in filteredProducts" 
          :key="product.id"
          class="card"
          @click="openQuickView(product)"
        >
          <div class="card-image-wrapper">
            <img :src="product.resolvedImage" :alt="product.title" class="card-image" :style="{ filter: product.cssFilter || 'none' }" />
            <div class="card-overlay">
              <button class="btn btn-secondary">Aperçu Rapide</button>
            </div>
          </div>
          <div class="card-info">
            <span class="card-category">{{ product.category }}</span>
            <h3 class="card-title">{{ product.title }}</h3>
            <div class="card-footer-row">
              <p class="card-price">{{ product.price }} MAD</p>
              <button class="card-add-btn" @click.stop="quickAddToCart(product)" aria-label="Ajouter au panier">
                <span class="btn-icons">
                  <span class="plus-sign">+</span>
                  <svg viewBox="0 0 24 24" width="14" height="14" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
                    <circle cx="9" cy="21" r="1"></circle>
                    <circle cx="20" cy="21" r="1"></circle>
                    <path d="M1 1h4l2.68 13.39a2 2 0 0 0 2 1.61h9.72a2 2 0 0 0 2-1.61L23 6H6"></path>
                  </svg>
                </span>
                <span>Ajouter</span>
              </button>
            </div>
          </div>
        </div>
      </TransitionGroup>
      
      <div v-if="filteredProducts.length === 0" class="empty-state">
        <p>Aucun produit trouvé dans cette catégorie.</p>
      </div>
    </div>

    <!-- Quick View Modal -->
    <Transition name="modal-fade">
      <div v-if="activeQuickView" class="modal-overlay" @click.self="closeQuickView">
        <div class="modal-content">
          <button class="close-btn" @click="closeQuickView">&times;</button>
          
          <div class="modal-body">
            <div class="modal-image-pane">
              <img :src="activeQuickView.resolvedImage" :alt="activeQuickView.title" :style="{ filter: activeQuickView.cssFilter || 'none' }" />
            </div>
            
            <div class="modal-info-pane">
              <span class="modal-category">{{ activeQuickView.category }}</span>
              <h2 class="modal-title">{{ activeQuickView.title }}</h2>
              <p class="modal-price">{{ activeQuickView.price }} MAD</p>
              
              <div class="divider"></div>
              
              <p class="modal-desc">{{ activeQuickView.description }}</p>
              
              <!-- Variations -->
              <div class="modal-variations">
                <div v-if="activeQuickView.variations.sizes" class="var-group">
                  <span class="var-label">Taille :</span>
                  <div class="var-options">
                    <button 
                      v-for="size in activeQuickView.variations.sizes" 
                      :key="size" 
                      type="button"
                      :class="['option-btn', { active: selectedSize === size }]"
                      @click="selectedSize = size"
                    >
                      {{ size }}
                    </button>
                  </div>
                </div>
                
                <div v-if="activeQuickView.variations.colors" class="var-group">
                  <span class="var-label">Couleur :</span>
                  <div class="var-options">
                    <button 
                      v-for="color in activeQuickView.variations.colors" 
                      :key="color" 
                      type="button"
                      :class="['option-btn', { active: selectedColor === color }]"
                      @click="selectedColor = color"
                    >
                      {{ color }}
                    </button>
                  </div>
                </div>
              </div>

              <!-- Specifications -->
              <div class="modal-specs">
                <h4 class="specs-heading">Spécifications</h4>
                <table class="specs-table">
                  <tbody>
                    <tr v-for="spec in activeQuickView.specifications" :key="spec.name">
                      <td class="spec-name">{{ spec.name }}</td>
                      <td class="spec-value">{{ spec.value }}</td>
                    </tr>
                  </tbody>
                </table>
              </div>
              
              <div class="modal-actions-row">
                <button type="button" @click="addToCart(activeQuickView)" class="add-to-cart-btn">
                  <span>Ajouter au Panier</span>
                  <svg viewBox="0 0 24 24" width="18" height="18" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
                    <circle cx="9" cy="21" r="1"></circle>
                    <circle cx="20" cy="21" r="1"></circle>
                    <path d="M1 1h4l2.68 13.39a2 2 0 0 0 2 1.61h9.72a2 2 0 0 0 2-1.61L23 6H6"></path>
                  </svg>
                </button>
                
                <a :href="getWhatsAppLink(activeQuickView)" target="_blank" class="buy-whatsapp-btn-secondary">
                  <span>Inquiéter via WhatsApp</span>
                </a>
              </div>
            </div>
          </div>
        </div>
      </div>
    </Transition>

    <!-- Floating Cart Toggle Button -->
    <a v-if="cartItemCount > 0" class="floating-cart-btn" href="/cart" aria-label="Voir le Panier">
      <svg viewBox="0 0 24 24" width="24" height="24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
        <circle cx="9" cy="21" r="1"></circle>
        <circle cx="20" cy="21" r="1"></circle>
        <path d="M1 1h4l2.68 13.39a2 2 0 0 0 2 1.61h9.72a2 2 0 0 0 2-1.61L23 6H6"></path>
      </svg>
      <span class="cart-badge">{{ cartItemCount }}</span>
    </a>
  </div>
</template>

<style scoped>
.product-section {
  padding: 60px 0 100px;
  color: var(--text-primary);
}

.container {
  max-width: 1200px;
  margin: 0 auto;
  padding: 0 20px;
}

.margin-bottom {
  margin-bottom: 90px;
}

.section-header {
  text-align: center;
  margin-bottom: 50px;
}

.subtitle {
  text-transform: uppercase;
  font-size: 0.8rem;
  letter-spacing: 3px;
  color: var(--text-secondary);
  font-weight: 600;
  display: block;
  margin-bottom: 10px;
}

.section-title {
  font-size: 2.5rem;
  font-weight: 700;
  letter-spacing: 1px;
  color: var(--text-primary);
  font-family: 'Outfit', system-ui, sans-serif;
}

.accent-line {
  width: 50px;
  height: 3px;
  background: linear-gradient(90deg, #10044E, #3b1fa8);
  margin: 15px auto 0;
  border-radius: 99px;
}

/* Category Tabs */
.tabs {
  display: flex;
  justify-content: center;
  gap: 15px;
  margin-bottom: 45px;
  flex-wrap: wrap;
}

.tab-btn {
  background: var(--bg-secondary);
  border: 1px solid var(--card-border);
  color: var(--text-secondary);
  padding: 10px 20px;
  border-radius: 8px;
  cursor: pointer;
  font-family: inherit;
  font-weight: 500;
  font-size: 0.9rem;
  text-transform: capitalize;
  transition: all 0.2s ease;
}

.tab-btn:hover {
  background: var(--accent-indigo-soft);
  color: var(--accent-indigo);
  border-color: #c5bfe8;
}

.tab-btn.active {
  background: var(--accent-indigo);
  border-color: var(--accent-indigo);
  color: #ffffff;
  box-shadow: 0 4px 14px rgba(16,4,78,0.20);
}

/* Product Grid */
.grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(280px, 1fr));
  gap: 30px;
}

.card {
  background: var(--bg-secondary);
  border: 1px solid var(--card-border);
  border-radius: 14px;
  overflow: hidden;
  cursor: pointer;
  transition: border-color 0.25s ease, box-shadow 0.25s ease, transform 0.25s ease;
}

.card:hover {
  border-color: var(--accent-indigo);
  box-shadow: 0 8px 32px rgba(16,4,78,0.10);
  transform: translateY(-3px);
}

.card-image-wrapper {
  position: relative;
  aspect-ratio: 1;
  background-color: var(--bg-primary);
  display: flex;
  align-items: center;
  justify-content: center;
  overflow: hidden;
  border-bottom: 1px solid var(--card-border);
}

.card-image {
  width: 100%;
  height: 100%;
  object-fit: cover;
}

.badge {
  position: absolute;
  top: 15px;
  left: 15px;
  background: var(--accent-primary);
  color: white;
  padding: 5px 11px;
  font-size: 0.68rem;
  font-weight: 700;
  border-radius: 99px;
  text-transform: uppercase;
  letter-spacing: 1px;
  box-shadow: 0 2px 8px rgba(16,4,78,0.30);
}

.card-overlay {
  position: absolute;
  inset: 0;
  background: rgba(246,245,250,0.93);
  opacity: 0;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: opacity 0.25s ease;
}

.card:hover .card-overlay {
  opacity: 1;
}

.card-info {
  padding: 20px;
}

.card-category {
  font-size: 0.75rem;
  color: var(--text-secondary);
  text-transform: uppercase;
  letter-spacing: 1px;
  font-weight: 600;
  display: block;
  margin-bottom: 6px;
}

.card-title {
  font-size: 1.05rem;
  font-weight: 600;
  margin-bottom: 8px;
  color: var(--text-primary);
  line-height: 1.4;
}

.card-price {
  font-size: 1.15rem;
  font-weight: 700;
  color: var(--accent-primary);
}

.empty-state {
  text-align: center;
  padding: 60px 0;
  color: var(--text-secondary);
  font-size: 1.1rem;
}

/* Buttons */
.btn {
  padding: 10px 20px;
  border-radius: 6px;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.2s;
  font-family: inherit;
  outline: none;
}

.btn-secondary {
  background: var(--accent-indigo);
  border: none;
  color: #ffffff;
  border-radius: 99px;
  letter-spacing: 0.3px;
}

.btn-secondary:hover {
  background: #1e0b7a;
}

/* Transitions */
.fade-grid-enter-active,
.fade-grid-leave-active {
  transition: all 0.4s ease;
}
.fade-grid-enter-from,
.fade-grid-leave-to {
  opacity: 0;
  transform: translateY(20px);
}

/* Quick View Modal */
.modal-overlay {
  position: fixed;
  inset: 0;
  background: rgba(0, 0, 0, 0.5);
  z-index: 9999;
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 20px;
}

.modal-content {
  background: var(--bg-secondary);
  border: 1px solid var(--card-border);
  width: 100%;
  max-width: 900px;
  max-height: 90vh;
  border-radius: 12px;
  position: relative;
  overflow-y: auto;
  box-shadow: 0 4px 30px rgba(0, 0, 0, 0.05);
  animation: modal-enter 0.2s ease-out;
  color: var(--text-primary);
}

@keyframes modal-enter {
  from {
    opacity: 0;
    transform: translateY(10px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

.close-btn {
  position: absolute;
  top: 15px;
  right: 20px;
  background: none;
  border: none;
  color: var(--text-secondary);
  font-size: 1.8rem;
  cursor: pointer;
  transition: color 0.2s;
  z-index: 10;
}

.close-btn:hover {
  color: var(--text-primary);
}

.modal-body {
  display: grid;
  grid-template-columns: 1fr 1.1fr;
  gap: 30px;
  padding: 30px;
}

@media (max-width: 768px) {
  .modal-body {
    grid-template-columns: 1fr;
    padding: 20px;
    gap: 20px;
  }
}

.modal-image-pane {
  background-color: var(--bg-primary);
  border: 1px solid var(--card-border);
  border-radius: 8px;
  overflow: hidden;
  display: flex;
  align-items: center;
  justify-content: center;
  aspect-ratio: 1;
}

.modal-image-pane img {
  width: 100%;
  height: 100%;
  object-fit: cover;
}

.modal-info-pane {
  display: flex;
  flex-direction: column;
}

.modal-category {
  font-size: 0.8rem;
  text-transform: uppercase;
  letter-spacing: 2px;
  color: var(--text-secondary);
  font-weight: 600;
  margin-bottom: 8px;
}

.modal-title {
  font-size: 1.8rem;
  font-weight: 700;
  line-height: 1.2;
  margin-bottom: 8px;
  font-family: 'Outfit', system-ui, sans-serif;
  color: var(--text-primary);
}

.modal-price {
  font-size: 1.4rem;
  font-weight: 700;
  color: var(--accent-primary);
}

.divider {
  height: 1px;
  background: var(--card-border);
  margin: 15px 0;
}

.modal-desc {
  color: var(--text-secondary);
  line-height: 1.6;
  font-size: 0.95rem;
  margin-bottom: 20px;
}

.modal-variations {
  display: flex;
  flex-direction: column;
  gap: 12px;
  margin-bottom: 20px;
}

.var-group {
  display: flex;
  align-items: center;
  gap: 15px;
}

.var-label {
  font-size: 0.85rem;
  font-weight: 600;
  color: var(--text-secondary);
  min-width: 100px;
}

.var-options {
  display: flex;
  gap: 6px;
  flex-wrap: wrap;
}

.option-tag {
  background: var(--bg-primary);
  border: 1px solid var(--card-border);
  color: var(--text-primary);
  padding: 4px 10px;
  border-radius: 4px;
  font-size: 0.75rem;
  font-weight: 500;
}

.modal-specs {
  margin-top: 10px;
  margin-bottom: 25px;
}

.specs-heading {
  font-size: 0.85rem;
  font-weight: 600;
  text-transform: uppercase;
  letter-spacing: 1px;
  color: var(--text-secondary);
  margin-bottom: 10px;
}

.specs-table {
  width: 100%;
  border-collapse: collapse;
}

.specs-table td {
  padding: 6px 10px;
  font-size: 0.85rem;
  border-bottom: 1px solid var(--card-border);
}

.spec-name {
  color: var(--text-secondary);
  font-weight: 500;
  width: 35%;
}

.spec-value {
  color: var(--text-primary);
}

.buy-whatsapp-btn {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 10px;
  background: linear-gradient(135deg, var(--accent-primary), #1a0870);
  color: white;
  text-decoration: none;
  font-weight: 700;
  font-size: 0.95rem;
  padding: 13px 24px;
  border-radius: 99px;
  text-align: center;
  transition: all 0.25s;
  margin-top: auto;
  box-shadow: 0 4px 16px rgba(16,4,78,0.28);
}

.buy-whatsapp-btn:hover {
  background: linear-gradient(135deg, var(--accent-primary-hover), #0d0340);
  box-shadow: 0 6px 22px rgba(16,4,78,0.38);
  transform: translateY(-1px);
}

/* Modal Fade transition */
.modal-fade-enter-active,
.modal-fade-leave-active {
  transition: opacity 0.3s ease;
}
.modal-fade-enter-from,
.modal-fade-leave-to {
  opacity: 0;
}

/* T-Shirts Section with Small Cards */
.tshirts-section {
  border-bottom: 1px solid var(--card-border);
  padding-bottom: 90px;
}

.small-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(180px, 1fr));
  gap: 20px;
}

.small-card {
  background: var(--bg-secondary);
  border: 1px solid var(--card-border);
  border-radius: 10px;
  overflow: hidden;
  cursor: pointer;
  transition: border-color 0.25s ease, box-shadow 0.25s ease, transform 0.25s ease;
}

.small-card:hover {
  border-color: var(--accent-indigo);
  box-shadow: 0 6px 20px rgba(16,4,78,0.10);
  transform: translateY(-2px);
}

.small-card .card-image-wrapper {
  aspect-ratio: 1;
  background-color: var(--bg-primary);
  display: flex;
  align-items: center;
  justify-content: center;
  overflow: hidden;
  border-bottom: 1px solid var(--card-border);
}

.small-card .card-image {
  width: 100%;
  height: 100%;
  object-fit: cover;
}

.small-card .card-info {
  padding: 12px;
}

.small-card .card-category {
  font-size: 0.7rem;
  color: var(--text-secondary);
  text-transform: uppercase;
  letter-spacing: 0.5px;
  font-weight: 600;
  display: block;
  margin-bottom: 2px;
}

.small-card .card-title {
  font-size: 0.9rem;
  font-weight: 600;
  margin-bottom: 4px;
  color: var(--text-primary);
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

.small-card .card-price {
  font-size: 0.9rem;
  font-weight: 700;
  color: var(--accent-primary);
}

.view-label {
  font-size: 0.75rem;
  font-weight: 600;
  color: var(--text-primary);
  text-transform: uppercase;
  letter-spacing: 0.5px;
}

/* Interactive Option Buttons */
.option-btn {
  background: var(--bg-primary);
  border: 1px solid var(--card-border);
  color: var(--text-primary);
  padding: 6px 14px;
  border-radius: 6px;
  cursor: pointer;
  font-family: inherit;
  font-weight: 600;
  font-size: 0.8rem;
  transition: all 0.2s ease;
}

.option-btn:hover {
  border-color: var(--accent-primary);
  color: var(--accent-primary);
}

.option-btn.active {
  background: var(--accent-primary);
  border-color: var(--accent-primary);
  color: #ffffff;
  box-shadow: 0 2px 8px rgba(16, 4, 78, 0.25);
}

/* Modal Actions Row */
.modal-actions-row {
  display: flex;
  gap: 15px;
  margin-top: 15px;
}

.add-to-cart-btn {
  flex: 1.2;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 10px;
  background: var(--accent-indigo);
  color: #ffffff;
  border: none;
  font-weight: 700;
  font-size: 0.95rem;
  padding: 13px 24px;
  border-radius: 99px;
  cursor: pointer;
  transition: all 0.25s;
  box-shadow: 0 4px 16px rgba(16, 4, 78, 0.15);
}

.add-to-cart-btn:hover {
  background: #1e0b7a;
  box-shadow: 0 6px 22px rgba(16, 4, 78, 0.25);
  transform: translateY(-1px);
}

.buy-whatsapp-btn-secondary {
  flex: 0.8;
  display: flex;
  align-items: center;
  justify-content: center;
  background: #F1E6D3;
  color: #10044E;
  text-decoration: none;
  font-weight: 700;
  font-size: 0.9rem;
  padding: 13px 20px;
  border-radius: 99px;
  text-align: center;
  transition: all 0.25s;
}

.buy-whatsapp-btn-secondary:hover {
  background: #e8d8c0;
}

/* Floating Cart Button */
.floating-cart-btn {
  position: fixed;
  bottom: 30px;
  right: 30px;
  width: 60px;
  height: 60px;
  border-radius: 50%;
  background: #10044E;
  color: #F1E6D3;
  border: none;
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  box-shadow: 0 6px 24px rgba(16, 4, 78, 0.25);
  z-index: 999;
  transition: transform 0.25s ease, background-color 0.2s;
}

.floating-cart-btn:hover {
  transform: scale(1.05);
  background: #1e0b7a;
}

.cart-badge {
  position: absolute;
  top: -5px;
  right: -5px;
  background: #10044E;
  color: #ffffff;
  font-size: 0.75rem;
  font-weight: 700;
  padding: 3px 8px;
  border-radius: 99px;
  box-shadow: 0 2px 8px rgba(16, 4, 78, 0.35);
}

/* Cart Drawer */
.cart-drawer-overlay {
  position: fixed;
  inset: 0;
  background: rgba(0, 0, 0, 0.5);
  z-index: 9999;
  display: flex;
  justify-content: flex-end;
}

.cart-drawer {
  width: 100%;
  max-width: 420px;
  height: 100%;
  background: var(--bg-secondary);
  box-shadow: -5px 0 30px rgba(0, 0, 0, 0.1);
  display: flex;
  flex-direction: column;
  animation: drawer-slide 0.3s ease-out;
}

@keyframes drawer-slide {
  from { transform: translateX(100%); }
  to { transform: translateX(0); }
}

.cart-drawer-header {
  padding: 20px;
  border-bottom: 1px solid var(--card-border);
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.cart-drawer-header h3 {
  font-family: var(--font-heading);
  font-size: 1.25rem;
  color: var(--text-primary);
}

.close-drawer-btn {
  background: none;
  border: none;
  color: var(--text-secondary);
  font-size: 1.8rem;
  cursor: pointer;
}

.cart-drawer-body {
  flex: 1;
  overflow-y: auto;
  padding: 20px;
}

.cart-empty {
  height: 100%;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  color: var(--text-secondary);
  gap: 15px;
}

.cart-empty p {
  font-size: 1rem;
}

.cart-items-list {
  display: flex;
  flex-direction: column;
  gap: 20px;
}

.cart-item {
  display: flex;
  gap: 15px;
  align-items: center;
  border-bottom: 1px solid var(--card-border);
  padding-bottom: 15px;
}

.cart-item-img {
  width: 70px;
  height: 70px;
  background-color: var(--bg-primary);
  border-radius: 8px;
  overflow: hidden;
  display: flex;
  align-items: center;
  justify-content: center;
  border: 1px solid var(--card-border);
}

.cart-item-img img {
  width: 100%;
  height: 100%;
  object-fit: cover;
}

.cart-item-details {
  flex: 1;
}

.cart-item-details h4 {
  font-size: 0.95rem;
  font-weight: 600;
  color: var(--text-primary);
  margin-bottom: 4px;
}

.cart-item-meta {
  font-size: 0.8rem;
  color: var(--text-secondary);
  margin-bottom: 6px;
}

.cart-item-price {
  font-size: 0.95rem;
  font-weight: 700;
  color: var(--accent-primary);
  margin-bottom: 8px;
}

.cart-item-qty-row {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.qty-selector {
  display: flex;
  align-items: center;
  background: var(--bg-primary);
  border: 1px solid var(--card-border);
  border-radius: 6px;
  overflow: hidden;
}

.qty-selector button {
  background: none;
  border: none;
  width: 28px;
  height: 28px;
  cursor: pointer;
  font-weight: 600;
  color: var(--text-primary);
  transition: background-color 0.2s;
}

.qty-selector button:hover {
  background-color: rgba(0, 0, 0, 0.05);
}

.qty-selector span {
  width: 30px;
  text-align: center;
  font-size: 0.85rem;
  font-weight: 600;
}

.remove-item-btn {
  background: none;
  border: none;
  color: var(--text-secondary);
  cursor: pointer;
  padding: 4px;
  transition: color 0.2s;
}

.remove-item-btn:hover {
  color: #10044E;
}

.cart-drawer-footer {
  padding: 20px;
  border-top: 1px solid var(--card-border);
  background: var(--bg-primary);
}

.cart-total-row {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 15px;
}

.cart-total-row span {
  font-size: 1rem;
  font-weight: 600;
  color: var(--text-primary);
}

.cart-total-row .cart-total-price {
  font-size: 1.25rem;
  font-weight: 700;
  color: var(--accent-primary);
}

.checkout-whatsapp-btn {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 10px;
  background: linear-gradient(135deg, var(--accent-primary), #1a0870);
  color: white;
  text-decoration: none;
  font-weight: 700;
  font-size: 0.95rem;
  padding: 13px;
  border-radius: 99px;
  text-align: center;
  transition: all 0.25s;
  box-shadow: 0 4px 16px rgba(16, 4, 78, 0.25);
}

.checkout-whatsapp-btn:hover {
  background: linear-gradient(135deg, var(--accent-primary-hover), #0d0340);
  box-shadow: 0 6px 20px rgba(16, 4, 78, 0.35);
  transform: translateY(-1px);
}

/* Drawer Transitions */
.drawer-fade-enter-active,
.drawer-fade-leave-active {
  transition: opacity 0.3s ease;
}
.drawer-fade-enter-active .cart-drawer,
.drawer-fade-leave-active .cart-drawer {
  transition: transform 0.3s ease;
}
.drawer-fade-enter-from,
.drawer-fade-leave-to {
  opacity: 0;
}
.drawer-fade-enter-from .cart-drawer,
.drawer-fade-leave-to .cart-drawer {
  transform: translateX(100%);
}

.card-footer-row {
  display: flex;
  align-items: center;
  justify-content: space-between;
  margin-top: 10px;
  gap: 10px;
}

.card-add-btn {
  display: flex;
  align-items: center;
  gap: 6px;
  background: var(--accent-indigo);
  color: #ffffff;
  border: none;
  font-weight: 700;
  font-size: 0.8rem;
  padding: 8px 16px;
  border-radius: 99px;
  cursor: pointer;
  transition: all 0.2s;
  box-shadow: 0 2px 8px rgba(16, 4, 78, 0.1);
}

.card-add-btn:hover {
  background: #1e0b7a;
  transform: translateY(-1px);
  box-shadow: 0 4px 12px rgba(16, 4, 78, 0.2);
}

.card-add-icon-btn {
  display: flex;
  align-items: center;
  justify-content: center;
  background: var(--accent-indigo);
  color: #ffffff;
  border: none;
  height: 28px;
  padding: 0 10px;
  border-radius: 99px;
  cursor: pointer;
  transition: all 0.2s;
  box-shadow: 0 2px 8px rgba(16, 4, 78, 0.1);
}

.card-add-icon-btn:hover {
  background: #1e0b7a;
  transform: scale(1.06);
  box-shadow: 0 4px 12px rgba(16, 4, 78, 0.2);
}

.btn-icons {
  display: inline-flex;
  align-items: center;
  gap: 1px;
  margin-right: 2px;
}

.plus-sign {
  font-size: 0.85rem;
  font-weight: 700;
  line-height: 1;
}
</style>
