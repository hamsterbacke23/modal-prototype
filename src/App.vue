<script setup>
import { ref } from 'vue';
import { KdsButton } from '@knime/kds-components';
import AnimatedModal from './components/AnimatedModal.vue';

const activeModal = ref(null)

const modalSizes = [
  { size: 400, label: 'Small (400px)', description: 'Quick confirmation dialogs', duration: '~150ms' },
  { size: 512, label: 'Medium (512px)', description: 'Standard forms and settings', duration: '~200ms' },
  { size: 720, label: 'Large (720px)', description: 'Complex data entry', duration: '~300ms' },
  { size: 976, label: 'XLarge (976px)', description: 'Wide tables and reviews', duration: '~400ms' },
  { size: 'fullscreen', label: 'Fullscreen', description: 'Immersive workflows', duration: '~500ms' },
]

const openModal = (size) => {
  activeModal.value = size
}

const closeModal = () => {
  activeModal.value = null
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
          <li><strong>Duration Scaling:</strong> Smaller modals = snappier (150ms), larger = smoother (500ms)</li>
          <li><strong>Scale + Fade:</strong> Entrance combines opacity and scale for depth perception</li>
          <li><strong>Asymmetric Timing:</strong> Exits are 33% faster than entrances</li>
          <li><strong>Reduced Motion:</strong> Respects user accessibility preferences</li>
        </ul>
      </div>

      <div class="modal-grid">
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
        <p><strong>Animation Details:</strong></p>
        <ul>
          <li>Size: {{ modal.size === 'fullscreen' ? 'calc(100vw - 60px)' : `${modal.size}px` }}</li>
          <li>Duration: {{ modal.duration }}</li>
          <li>Easing: Emphasized Decelerate (entrance)</li>
          <li>Choreography: Height reveal + Scale ({{ modal.size === 400 ? '0.85' : modal.size === 'fullscreen' ? '0.92' : '0.88' }} → 1.0) + Fade</li>
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
</style>
