<script setup>
import { ref } from 'vue';
import { KdsButton } from '@knime/kds-components';
import AnimatedModal from './components/AnimatedModal.vue';

const activeModal = ref(null)
const resizeDemoActive = ref(false)
const resizeDemoSize = ref(400)
const resizeDemoContent = ref('small')
const isTransitioning = ref(false)

const modalSizes = [
  { size: 400, label: 'Small (400px)', description: 'Quick confirmation dialogs', duration: '100ms' },
  { size: 512, label: 'Medium (512px)', description: 'Standard forms and settings', duration: '158ms' },
  { size: 720, label: 'Large (720px)', description: 'Complex data entry', duration: '222ms' },
  { size: 976, label: 'XLarge (976px)', description: 'Wide tables and reviews', duration: '300ms' },
  { size: 'fullscreen', label: 'Fullscreen', description: 'Immersive workflows', duration: '350ms' },
]

const resizeSizes = [400, 512, 720, 976]

const openModal = (size) => {
  activeModal.value = size
}

const closeModal = () => {
  activeModal.value = null
}

const openResizeDemo = () => {
  resizeDemoActive.value = true
  resizeDemoSize.value = 400
  resizeDemoContent.value = 'small'
}

const closeResizeDemo = () => {
  resizeDemoActive.value = false
}

const cycleSize = async () => {
  if (isTransitioning.value) return
  
  const currentIndex = resizeSizes.indexOf(resizeDemoSize.value)
  const nextIndex = (currentIndex + 1) % resizeSizes.length
  const nextSize = resizeSizes[nextIndex]
  
  isTransitioning.value = true
  
  // Fade out content
  resizeDemoContent.value = 'transitioning'
  
  // Wait for fade out (150ms)
  await new Promise(resolve => setTimeout(resolve, 150))
  
  // Change size - let the CSS transition handle it (100ms)
  resizeDemoSize.value = nextSize
  
  // Wait for size transition to complete
  await new Promise(resolve => setTimeout(resolve, 100))
  
  // Fade in new content
  const sizeLabels = { 400: 'small', 512: 'medium', 720: 'large', 976: 'xlarge' }
  resizeDemoContent.value = sizeLabels[nextSize]
  
  isTransitioning.value = false
}
</script>

<template>
  <div class="demo-container">
    <header class="demo-header">
      <h1>Modal Animation Prototype</h1>
      <p class="subtitle">Physics-based transitions with size-aware choreography</p>
    </header>

    <section class="demo-content">
      <div class="info-panel">
        <h2>Design Principles</h2>
        <ul>
          <li><strong>Duration Scaling:</strong> Smaller modals = snappier (100ms), larger = smoother (350ms)</li>
          <li><strong>Scale + Fade:</strong> Entrance combines opacity and scale for depth perception</li>
          <li><strong>Asymmetric Timing:</strong> Exits are 33% faster than entrances (67% of entrance duration)</li>
          <li><strong>Reduced Motion:</strong> Respects user accessibility preferences</li>
        </ul>
      </div>

      <div class="modal-grid">
        <div
          class="modal-card resize-demo-card"
          @click="openResizeDemo"
        >
          <div class="modal-card-header">
            <h3>Size Transition Demo</h3>
            <span class="duration-badge special">🔄</span>
          </div>
          <p class="modal-card-description">Smooth size transitions with content fade</p>
          <KdsButton
            label="Open Demo"
            variant="primary"
            class="trigger-button"
            @click="openResizeDemo"
          />
        </div>
        
        <div
          v-for="modal in modalSizes"
          :key="modal.size"
          class="modal-card"
          @click="openModal(modal.size)"
        >
          <div class="modal-card-header">
            <h3>{{ modal.label }}</h3>
            <span class="duration-badge">{{ modal.duration }}</span>
          </div>
          <p class="modal-card-description">{{ modal.description }}</p>
          <KdsButton
            label="Open Modal"
            variant="primary"
            class="trigger-button"
            @click="openModal(modal.size)"
          />
        </div>
      </div>
    </section>

    <!-- Modal instances -->
    <AnimatedModal
      v-for="modal in modalSizes"
      :key="`modal-${modal.size}`"
      :active="activeModal === modal.size"
      :size="modal.size"
      :title="`${modal.label} Modal`"
      @close="closeModal"
    >
      <div class="modal-demo-content">
        <p><strong>{{ modal.description }}</strong></p>
        <p><strong>Animation Details:</strong></p>
        <ul>
          <li>Modal Size: {{ modal.size === 'fullscreen' ? 'calc(100vw - 60px)' : `${modal.size}px` }}</li>
          <li>Entrance Duration: {{ modal.duration.replace('~', '') }}</li>
          <li>Exit Duration: {{ Math.round(parseInt(modal.duration.replace(/[^\d]/g, '')) * 0.67) }}ms (33% faster)</li>
          <li>Scale Transform: {{ modal.size === 400 ? '0.85' : modal.size === 'fullscreen' ? '0.92' : '0.88' }} → 1.0</li>
          <li>Easing: Emphasized Decelerate (entrance), Emphasized Accelerate (exit)</li>
        </ul>
        <p>
          This modal demonstrates the physics-based animation system where duration
          and intensity scale proportionally with modal size to maintain consistent
          perceived velocity and cognitive continuity.
        </p>
      </div>
      <template #footer>
        <KdsButton variant="transparent" label="Cancel" @click="closeModal" />
        <KdsButton variant="filled" label="Submit" @click="closeModal" />
      </template>
    </AnimatedModal>
    
    <!-- Resize Demo Modal -->
    <AnimatedModal
      :active="resizeDemoActive"
      :size="resizeDemoSize"
      :title="`Size Transition Demo (${resizeDemoSize}px)`"
      @close="closeResizeDemo"
    >
      <div class="resize-demo-content">
        <div class="content-section" :class="{ 'active': resizeDemoContent === 'small', 'transitioning': resizeDemoContent === 'transitioning' }">
          <h3>Small Modal (400px)</h3>
          <p>This is the smallest size. Perfect for quick confirmations and simple decisions.</p>
          <p>Content is minimal and focused on a single action or question.</p>
        </div>
        
        <div class="content-section" :class="{ 'active': resizeDemoContent === 'medium', 'transitioning': resizeDemoContent === 'transitioning' }">
          <h3>Medium Modal (512px)</h3>
          <p>Standard size for most use cases. Provides enough space for forms and settings.</p>
          <p>Can accommodate multiple form fields and moderate amounts of text content.</p>
          <p>This is the default size that works well for most interactions.</p>
        </div>
        
        <div class="content-section" :class="{ 'active': resizeDemoContent === 'large', 'transitioning': resizeDemoContent === 'transitioning' }">
          <h3>Large Modal (720px)</h3>
          <p>Expanded size for complex data entry and detailed information displays.</p>
          <p>Suitable for:</p>
          <ul>
            <li>Multi-column layouts</li>
            <li>Complex forms with many fields</li>
            <li>Tables with several columns</li>
            <li>Rich text editors</li>
          </ul>
          <p>Provides ample space while maintaining focus on the task.</p>
        </div>
        
        <div class="content-section" :class="{ 'active': resizeDemoContent === 'xlarge', 'transitioning': resizeDemoContent === 'transitioning' }">
          <h3>XLarge Modal (976px)</h3>
          <p>Maximum standard size for the most complex interfaces.</p>
          <p>Ideal for:</p>
          <ul>
            <li>Wide data tables</li>
            <li>Complex review interfaces</li>
            <li>Multi-panel configurations</li>
            <li>Advanced settings with preview panes</li>
          </ul>
          <p>This size approaches the limit before fullscreen becomes more appropriate.</p>
          <p>Notice how the content smoothly transitions without any jarring shifts or text reflow.</p>
        </div>
      </div>
      
      <template #footer>
        <KdsButton variant="transparent" label="Close" @click="closeResizeDemo" />
        <KdsButton 
          variant="filled" 
          label="Cycle Size" 
          @click="cycleSize"
          :disabled="isTransitioning"
        />
      </template>
    </AnimatedModal>
  </div>
</template>

<style scoped>
.demo-container {
  min-height: 100vh;
  padding: var(--kds-spacing-container-2x);
  background: var(--knime-porcelain);
}

.demo-header {
  text-align: center;
  margin-bottom: var(--kds-spacing-container-4x);
  padding: var(--kds-spacing-container-3x) var(--kds-spacing-container-2x);
  background: var(--knime-white);
  border-radius: var(--kds-border-radius-container-1x);
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.08);
  border: 1px solid var(--knime-silver);
}

.demo-header h1 {
  font-size: 2.5rem;
  font-weight: 700;
  line-height: 1.2;
  color: var(--knime-masala);
  margin: 0 0 var(--kds-spacing-container-0-75x) 0;
  letter-spacing: -0.02em;
}

.subtitle {
  font-size: 1.125rem;
  color: var(--knime-dove-gray);
  margin: 0;
  font-weight: 500;
}

.demo-content {
  max-width: 1200px;
  margin: 0 auto;
}

.info-panel {
  background: var(--knime-white);
  border-radius: var(--kds-border-radius-container-1x);
  padding: var(--kds-spacing-container-2x);
  margin-bottom: var(--kds-spacing-container-3x);
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.08);
  border-left: 4px solid var(--knime-cornflower);
}

.info-panel h2 {
  margin-top: 0;
  margin-bottom: var(--kds-spacing-container-1-5x);
  font-size: 1.25rem;
  font-weight: 600;
  color: var(--knime-masala);
  display: flex;
  align-items: center;
  gap: var(--kds-spacing-container-0-5x);
}

.info-panel h2::before {
  content: '✨';
  font-size: 1.5em;
}

.info-panel ul {
  list-style: none;
  padding: 0;
  margin: 0;
  display: flex;
  flex-direction: column;
  gap: var(--kds-spacing-container-1x);
}

.info-panel li {
  color: var(--knime-dove-gray);
}

.modal-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
  column-gap: var(--kds-spacing-container-3x);
  row-gap: var(--kds-spacing-container-3x);
  margin-bottom: var(--kds-spacing-container-3x);
}

.modal-card {
  background: var(--knime-white);
  border-radius: var(--kds-border-radius-container-1x);
  padding: var(--kds-spacing-container-2x);
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.08);
  transition: all 0.2s cubic-bezier(0.4, 0.0, 0.2, 1);
  display: flex;
  flex-direction: column;
  gap: var(--kds-spacing-container-1-5x);
  border: 2px solid transparent;
  position: relative;
  overflow: hidden;
  cursor: pointer;
}

.modal-card::before {
  content: '';
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  height: 4px;
  background: linear-gradient(90deg, 
    var(--knime-cornflower) 0%, 
    var(--knime-masala) 100%);
  transform: scaleX(0);
  transition: transform 0.3s cubic-bezier(0.4, 0.0, 0.2, 1);
}

.modal-card:hover {
  transform: translateY(-6px);
  box-shadow: 0 8px 24px rgba(0, 0, 0, 0.12);
  border-color: var(--knime-cornflower);
}

.modal-card:hover::before {
  transform: scaleX(1);
}

.modal-card:active {
  transform: translateY(-2px);
}

.modal-card-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  gap: var(--kds-spacing-container-0-75x);
}

.modal-card-header h3 {
  margin: 0;
  color: var(--knime-masala);
  font-size: 1.125rem;
  font-weight: 600;
}

.duration-badge {
  background: linear-gradient(135deg, var(--knime-cornflower) 0%, var(--knime-masala) 100%);
  color: var(--knime-white);
  padding: var(--kds-spacing-container-0-37x) var(--kds-spacing-container-1x);
  border-radius: var(--kds-border-radius-container-0-50x);
  font-size: 0.875rem;
  font-weight: 600;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
  white-space: nowrap;
}

.modal-card-description {
  color: var(--knime-dove-gray);
  margin: 0;
  font-size: 0.9375rem;
  line-height: 1.5;
}

.trigger-button {
  width: 100%;
}

.trigger-button:deep(button) {
  width: 100%;
}

.modal-demo-content {
  line-height: 1.6;
}

.modal-demo-content p {
  margin: 0 0 var(--kds-spacing-container-1x) 0;
  line-height: 1.6;
  color: var(--knime-masala);
}

.modal-demo-content strong {
  color: var(--knime-masala);
  font-weight: 600;
}

.modal-demo-content ul {
  list-style: none;
  padding-left: 0;
  margin: 0 0 var(--kds-spacing-container-1x) 0;
  display: flex;
  flex-direction: column;
  gap: var(--kds-spacing-container-0-5x);
}

.modal-demo-content li {
  color: var(--knime-dove-gray);
}

.resize-demo-card::before {
  background: linear-gradient(90deg, 
    var(--knime-yellow) 0%, 
    var(--knime-coral) 100%) !important;
}

.duration-badge.special {
  background: linear-gradient(135deg, var(--knime-yellow) 0%, var(--knime-coral) 100%);
}

.resize-demo-content {
  position: relative;
  min-height: 400px;
}

.resize-demo-content .content-section {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  visibility: hidden;
  opacity: 0;
  transition: opacity 150ms ease-in-out;
  pointer-events: none;
}

.resize-demo-content .content-section.active {
  position: absolute;
  visibility: visible;
  opacity: 1;
  pointer-events: auto;
}

.resize-demo-content .content-section.transitioning {
  opacity: 0;
}

.resize-demo-content .content-section h3 {
  margin-top: 0;
  margin-bottom: var(--kds-spacing-container-1x);
  color: var(--knime-masala);
  font-size: 1.125rem;
  font-weight: 600;
}

.resize-demo-content .content-section p {
  margin: 0 0 var(--kds-spacing-container-1x) 0;
  line-height: 1.6;
  color: var(--knime-masala);
}

.resize-demo-content .content-section ul {
  list-style: disc;
  padding-left: var(--kds-spacing-container-2x);
  margin: var(--kds-spacing-container-1x) 0;
}

.resize-demo-content .content-section li {
  color: var(--knime-dove-gray);
  margin-bottom: var(--kds-spacing-container-0-5x);
}

@keyframes fadeIn {
  from {
    opacity: 0;
  }
  to {
    opacity: 1;
  }
}
</style>
