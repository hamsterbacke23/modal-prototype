<script setup lang="ts">
import { computed, watch, nextTick, ref } from 'vue';
import { KdsModal } from '@knime/kds-components';

export interface KdsEnhancedModalProps {
  active: boolean;
  title?: string;
  icon?: string;
  size?: 400 | 512 | 720 | 976 | 'fullscreen';
  variant?: 'default' | 'plain';
  closedby?: 'any' | 'closerequest' | 'none';
}

const props = withDefaults(defineProps<KdsEnhancedModalProps>(), {
  active: false,
  title: '',
  icon: undefined,
  size: 512,
  variant: 'default',
  closedby: 'any',
});

const emit = defineEmits<{
  close: [event: Event];
}>();

const modalRef = ref<InstanceType<typeof KdsModal>>();
const isClosing = ref(false);

// Watch for active changes and apply animations via JavaScript
watch(() => props.active, async (isActive, wasActive) => {
  if (isActive === wasActive) return;
  
  // Only handle opening animation here - closing is handled by handleClose
  if (!isActive) return;
  
  isClosing.value = false;
  
  await nextTick();
  await nextTick();
  
  // Find the dialog element
  let dialog: HTMLDialogElement | null = null;
  
  if (modalRef.value) {
    const el = modalRef.value.$el as HTMLElement;
    dialog = el?.querySelector('dialog') as HTMLDialogElement;
  }
  
  if (!dialog) {
    const dialogs = document.querySelectorAll('dialog');
    for (let i = dialogs.length - 1; i >= 0; i--) {
      const d = dialogs[i] as HTMLDialogElement;
      if (d.open) {
        dialog = d;
        break;
      }
    }
  }
  
  if (!dialog) {
    console.error('Could not find dialog element for size:', props.size);
    return;
  }
  
  console.log(`Opening size ${props.size} with duration: ${animationDuration.value}ms, scale: ${scaleFrom.value}`);
  
  const duration = animationDuration.value;
  const scale = scaleFrom.value;
  
  // Opening animation - dialog and backdrop
  dialog.style.transition = 'none';
  dialog.style.opacity = '0';
  dialog.style.transform = `scale(${scale}) translateY(10px)`;
  
  // Remove any existing backdrop animation class
  dialog.classList.remove('backdrop-entering', 'backdrop-exiting');
  
  void dialog.offsetHeight;
  
  requestAnimationFrame(() => {
    const scaleDuration = Math.round(duration * 0.6);
    dialog.style.transition = `opacity ${duration}ms cubic-bezier(0.0, 0.0, 0.2, 1), transform ${scaleDuration}ms cubic-bezier(0.0, 0.0, 0.2, 1)`;
    dialog.style.opacity = '1';
    dialog.style.transform = 'scale(1) translateY(0)';
    
    // Add class to animate backdrop with delay
    dialog.classList.add('backdrop-entering');
  });
}, { flush: 'post' });

const handleClose = async (event: Event) => {
  // Prevent default close and play exit animation first
  if (!isClosing.value) {
    event.preventDefault();
    isClosing.value = true;
    
    // Find the dialog element
    let dialog: HTMLDialogElement | null = null;
    
    if (modalRef.value) {
      const el = modalRef.value.$el as HTMLElement;
      dialog = el?.querySelector('dialog') as HTMLDialogElement;
    }
    
    if (!dialog) {
      const dialogs = document.querySelectorAll('dialog');
      for (let i = dialogs.length - 1; i >= 0; i--) {
        const d = dialogs[i] as HTMLDialogElement;
        if (d.open) {
          dialog = d;
          break;
        }
      }
    }
    
    console.log('Closing with fast fade animation, dialog found:', !!dialog);
    
    if (dialog) {
      dialog.style.transition = 'opacity 100ms ease-out';
      dialog.style.opacity = '0';
      
      // Remove entering class and add exiting for backdrop
      dialog.classList.remove('backdrop-entering');
      dialog.classList.add('backdrop-exiting');
    }
    
    // Emit close after animation completes
    setTimeout(() => {
      emit('close', event);
      isClosing.value = false;
    }, 100);
  }
};

// Map custom sizes to KdsModal width prop
const modalWidth = computed(() => {
  if (props.size === 'fullscreen') return 'full';
  if (props.size <= 400) return 'small';
  if (props.size <= 512) return 'medium';
  if (props.size <= 720) return 'large';
  return 'xlarge';
});

// Calculate animation duration based on size (150ms - 500ms)
const animationDuration = computed(() => {
  if (props.size === 'fullscreen') return 500;
  if (typeof props.size === 'number') {
    // Scale duration: 400px = 150ms, 976px = 400ms
    const minSize = 400;
    const maxSize = 976;
    const minDuration = 150;
    const maxDuration = 400;
    
    const normalized = (props.size - minSize) / (maxSize - minSize);
    return Math.round(minDuration + normalized * (maxDuration - minDuration));
  }
  return 300;
});

// Size class for custom styling
const sizeClass = computed(() => {
  return `size-${props.size}`;
});

// Transition name based on size
const transitionName = computed(() => {
  if (props.size === 400) return 'modal-small';
  if (props.size === 'fullscreen') return 'modal-fullscreen';
  return 'modal-standard';
});

// Exit duration (33% faster than entrance)
const exitDuration = computed(() => {
  return Math.round(animationDuration.value * 0.67);
});

// Custom width for non-fullscreen sizes
const customWidth = computed(() => {
  if (props.size === 'fullscreen') return undefined;
  if (typeof props.size === 'number') {
    return `${props.size}px`;
  }
  return undefined;
});

// Scale from value based on size
const scaleFrom = computed(() => {
  if (props.size === 400) return '0.92';
  if (props.size === 'fullscreen') return '0.97';
  return '0.95';
});
</script>

<template>
  <KdsModal
    ref="modalRef"
    :active="active"
    :title="title"
    :icon="icon"
    :width="modalWidth"
    :variant="variant"
    :closedby="closedby"
    :class="['enhanced-modal', sizeClass]"
    :style="{
      '--modal-width': customWidth,
      '--animation-duration': `${animationDuration}ms`,
      '--exit-duration': `${exitDuration}ms`,
      '--scale-from': scaleFrom,
    }"
    @close="handleClose"
  >
    <slot />
    <template #actions>
      <slot name="actions" />
    </template>
  </KdsModal>
</template>

<style>
/* Remove scoped to ensure styles apply */
.enhanced-modal.size-400 dialog,
.enhanced-modal.size-512 dialog,
.enhanced-modal.size-720 dialog,
<style>
/* Remove scoped to ensure styles apply */
.enhanced-modal.size-400 dialog,
.enhanced-modal.size-512 dialog,
.enhanced-modal.size-720 dialog,
.enhanced-modal.size-976 dialog {
  max-width: var(--modal-width) !important;
  width: var(--modal-width) !important;
}

.enhanced-modal.size-fullscreen dialog {
  width: calc(100vw - 60px) !important;
  height: calc(100vh - 60px) !important;
  max-width: calc(100vw - 60px) !important;
  max-height: calc(100vh - 60px) !important;
}

/* Backdrop animations - override KDS styles with higher specificity */
.enhanced-modal dialog.backdrop-entering::backdrop,
dialog[data-v-2b38736e].backdrop-entering::backdrop {
  background: var(--kds-color-blanket-default) !important;
  animation: backdropFadeIn var(--animation-duration) ease-out !important;
  animation-delay: calc(var(--animation-duration) * 0.3) !important;
  animation-fill-mode: backwards !important;
}

.enhanced-modal dialog.backdrop-exiting::backdrop,
dialog[data-v-2b38736e].backdrop-exiting::backdrop {
  background: var(--kds-color-blanket-default) !important;
  animation: backdropFadeOut 100ms ease-out !important;
  animation-fill-mode: forwards !important;
}

@keyframes backdropFadeIn {
  from {
    opacity: 0 !important;
  }
  to {
    opacity: 1 !important;
  }
}

@keyframes backdropFadeOut {
  from {
    opacity: 1 !important;
  }
  to {
    opacity: 0 !important;
  }
}

@media (prefers-reduced-motion: reduce) {
  .enhanced-modal dialog {
    transition: opacity var(--animation-duration) ease-out !important;
  }
  
  dialog.backdrop-entering::backdrop,
  dialog.backdrop-exiting::backdrop {
    animation: none !important;
  }
}
</style>