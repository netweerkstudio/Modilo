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

const categories = ['all', 'hoodie', 't-shirt', 'bags', 'tote-bags'];
const categoryLabels = {
  all: 'Tout',
  hoodie: 'Sweats',
  't-shirt': 'T-Shirts',
  bags: 'Sacs',
  'tote-bags': 'Tote Bags'
};
const selectedCategory = ref('all');
const activeQuickView = ref(null);

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
  document.body.style.overflow = 'hidden';
}

function closeQuickView() {
  activeQuickView.value = null;
  document.body.style.overflow = 'auto';
}

// Generate custom WhatsApp message link
function getWhatsAppLink(product) {
  const message = `Bonjour Modilo, je suis intéressé(e) par ${product.title} ($${product.price}). Pourriez-vous m’en dire plus ?`;
  return `https://wa.me/+212630257293?text=${encodeURIComponent(message)}`;
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
            <p class="card-price">${{ product.price }}</p>
          </div>
        </div>
      </div>
    </div>

    <!-- Main Shop / Categories Section -->
    <div v-if="!props.showOnlyFeatured" class="container">
      <div class="section-header">
        <span class="subtitle">Notre Collection</span>
        <h2 class="section-title">Parcourir par Catégorie</h2>
        <div class="accent-line"></div>
      </div>

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
            <p class="card-price">${{ product.price }}</p>
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
              <p class="modal-price">${{ activeQuickView.price }}</p>
              
              <div class="divider"></div>
              
              <p class="modal-desc">{{ activeQuickView.description }}</p>
              
              <!-- Variations -->
              <div class="modal-variations">
                <div v-if="activeQuickView.variations.sizes" class="var-group">
                  <span class="var-label">Tailles disponibles :</span>
                  <div class="var-options">
                    <span v-for="size in activeQuickView.variations.sizes" :key="size" class="option-tag">{{ size }}</span>
                  </div>
                </div>
                
                <div v-if="activeQuickView.variations.colors" class="var-group">
                  <span class="var-label">Couleurs disponibles :</span>
                  <div class="var-options">
                    <span v-for="color in activeQuickView.variations.colors" :key="color" class="option-tag">{{ color }}</span>
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
              
              <a :href="getWhatsAppLink(activeQuickView)" target="_blank" class="buy-whatsapp-btn">
                <span>Acheter via WhatsApp</span>
                <svg viewBox="0 0 24 24" width="20" height="20">
                  <path fill="currentColor" d="M12.012 2c-5.506 0-9.989 4.478-9.99 9.984a9.96 9.96 0 0 0 1.333 4.982L2 22l5.233-1.371a9.936 9.936 0 0 0 4.779 1.21h.005c5.505 0 9.988-4.479 9.989-9.986 0-2.67-1.037-5.178-2.922-7.062A9.925 9.925 0 0 0 12.012 2zm5.835 14.165c-.32.9-.1.9-.1.9s-.427 1.099-1.637 1.328c-1.21.23-2.56.096-4.576-.718-2.016-.814-3.526-2.585-4.464-3.878-.938-1.293-1.579-2.738-1.579-4.22 0-1.48.718-2.203 1.026-2.502.308-.299.82-.455 1.077-.455.257 0 .492-.01.705.01.214.02.487-.08.761.564.274.645.94 2.274 1.017 2.43.077.157.12.338.017.532-.102.193-.154.314-.308.492-.154.177-.325.298-.462.46-.153.18-.316.378-.137.684.18.307.8 1.314 1.718 2.128.918.814 1.692 1.065 1.992 1.185.3.12.478.105.658-.1.18-.206.77-.895.974-1.2.206-.307.41-.258.693-.153.282.105 1.795.847 2.103.992.308.145.513.218.59.35.077.133.077.77-.243 1.67z"/>
                </svg>
              </a>
            </div>
          </div>
        </div>
      </div>
    </Transition>
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
  background: linear-gradient(90deg, var(--accent-red), var(--accent-cyan));
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
  background: var(--bg-primary);
  color: var(--text-primary);
}

.tab-btn.active {
  background: var(--text-primary);
  border-color: var(--text-primary);
  color: var(--bg-secondary);
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
  border-radius: 12px;
  overflow: hidden;
  cursor: pointer;
  transition: border-color 0.2s ease;
}

.card:hover {
  border-color: var(--text-primary);
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
  background: var(--accent-red);
  color: white;
  padding: 4px 10px;
  font-size: 0.7rem;
  font-weight: 700;
  border-radius: 4px;
  text-transform: uppercase;
  letter-spacing: 0.5px;
}

.card-overlay {
  position: absolute;
  inset: 0;
  background: rgba(255, 255, 255, 0.95);
  opacity: 0;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: opacity 0.2s ease;
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
  color: var(--text-primary);
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
  background: var(--text-primary);
  border: none;
  color: var(--bg-secondary);
}

.btn-secondary:hover {
  background: var(--text-secondary);
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
  color: var(--accent-red);
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
  background-color: var(--accent-green);
  color: white;
  text-decoration: none;
  font-weight: 700;
  font-size: 0.95rem;
  padding: 12px 24px;
  border-radius: 6px;
  text-align: center;
  transition: background-color 0.2s;
  margin-top: auto;
}

.buy-whatsapp-btn:hover {
  background-color: #128c3e;
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
  border-radius: 8px;
  overflow: hidden;
  cursor: pointer;
  transition: border-color 0.2s ease;
}

.small-card:hover {
  border-color: var(--text-primary);
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
  color: var(--text-primary);
}

.view-label {
  font-size: 0.75rem;
  font-weight: 600;
  color: var(--text-primary);
  text-transform: uppercase;
  letter-spacing: 0.5px;
}
</style>
