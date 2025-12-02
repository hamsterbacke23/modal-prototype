<script setup lang="ts">
import { computed, nextTick, useTemplateRef, watch, ref } from 'vue';
import { KdsButton, KdsIcon } from '@knime/kds-components';

export interface AnimatedModalProps {
  active: boolean;
  title?: string;
  icon?: string;
  size?: 400 | 512 | 720 | 976 | 'fullscreen';
  variant?: 'default' | 'plain';
  closedby?: 'any' | 'closerequest' | 'none';
}

const props = withDefaults(defineProps<AnimatedModalProps>(), {
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

const dialog = useTemplateRef('dialogElement');
const isClosing = ref(false);
const shouldShowContent = ref(false);

// Calculate animation duration based on size (100ms - 350ms) - shorter overall
const animationDuration = computed(() => {
  if (props.size === 'fullscreen') return 350;
  if (typeof props.size === 'number') {
    const minSize = 400;
    const maxSize = 976;
    const minDuration = 100;
    const maxDuration = 300;
    
    const normalized = (props.size - minSize) / (maxSize - minSize);
    return Math.round(minDuration + normalized * (maxDuration - minDuration));
  }
  return 250;
});

// Scale from value based on size - more dramatic for expansion effect
const scaleFrom = computed(() => {
  if (props.size === 400) return 0.85;
  if (props.size === 'fullscreen') return 0.92;
  return 0.88;
});

// Custom width for non-fullscreen sizes
const cssModalWidth = computed(() => {
  if (props.size === 'fullscreen') return 'var(--modal-full-size)';
  if (typeof props.size === 'number') {
    return `${props.size}px`;
  }
  return '512px';
});

const cssModalHeight = computed(() => {
  if (props.size === 'fullscreen') return 'var(--modal-full-size)';
  return 'fit-content';
});

const onClose = (e: Event) => {
  // Don't emit immediately - let the watch handle the animation
  if (!isClosing.value && props.active) {
    e.preventDefault();
    // Trigger closing by emitting (parent will set active=false)
    emit('close', e);
  }
};

// Watch for active changes and apply animations
watch(
  () => props.active,
  async (active) => {
    if (active) {
      isClosing.value = false;
      shouldShowContent.value = true;
      await nextTick();
      dialog.value?.showModal();
      
      await nextTick();
      
      // Opening animation
      const dialogEl = dialog.value as HTMLDialogElement;
      if (dialogEl) {
        const duration = animationDuration.value;
        const scale = scaleFrom.value;
        
        // Set initial state - clipped height with slight scale
        dialogEl.style.transition = 'none';
        dialogEl.style.opacity = '0.7';
        dialogEl.style.transform = `scale(${scale}) translateY(-10px)`;
        dialogEl.style.clipPath = 'inset(0 0 70% 0)';
        dialogEl.classList.add('backdrop-entering');
        dialogEl.classList.remove('backdrop-exiting');
        
        void dialogEl.offsetHeight;
        
        // Animate to final state - reveal height + scale
        requestAnimationFrame(() => {
          const opacityDuration = Math.round(duration * 0.2);
          dialogEl.style.transition = `opacity ${opacityDuration}ms cubic-bezier(0.0, 0.0, 0.2, 1), transform ${duration}ms cubic-bezier(0.0, 0.0, 0.2, 1), clip-path ${duration}ms cubic-bezier(0.0, 0.0, 0.2, 1)`;
          dialogEl.style.opacity = '1';
          dialogEl.style.transform = 'scale(1) translateY(0)';
          dialogEl.style.clipPath = 'inset(0 0 0% 0)';
        });
      }
    } else if (!isClosing.value) {
      // Closing animation
      const dialogElClose = dialog.value as HTMLDialogElement;
      if (dialogElClose) {
        isClosing.value = true;
        
        // Exit duration: 67% of entrance
        const exitDuration = Math.round(animationDuration.value * 0.67);
        const scale = scaleFrom.value;
        
        dialogElClose.classList.remove('backdrop-entering');
        dialogElClose.classList.add('backdrop-exiting');
        
        // Collapse height with scale - opacity only at very end
        const opacityDuration = Math.round(exitDuration * 0.25);
        const opacityDelay = Math.round(exitDuration * 0.65);
        dialogElClose.style.transition = `opacity ${opacityDuration}ms ease-out ${opacityDelay}ms, transform ${exitDuration}ms cubic-bezier(0.4, 0.0, 1, 1), clip-path ${exitDuration}ms cubic-bezier(0.4, 0.0, 1, 1)`;
        dialogElClose.style.opacity = '0';
        dialogElClose.style.transform = `scale(${scale}) translateY(-10px)`;
        dialogElClose.style.clipPath = 'inset(0 0 80% 0)';
        
        setTimeout(() => {
          dialog.value?.close();
          shouldShowContent.value = false;
          isClosing.value = false;
        }, exitDuration);
      } else {
        dialog.value?.close();
        shouldShowContent.value = false;
      }
    }
  },
  { immediate: true }
);
</script>

<template>
  <dialog
    ref="dialogElement"
    :class="['animated-modal', `size-${size}`]"
    :closedby="closedby"
    @cancel.prevent="onClose"
  >
    <template v-if="active || isClosing">
      <header class="modal-header">
        <KdsIcon v-if="props.icon" :name="props.icon" size="medium" />
        <div class="modal-header-title">{{ title }}</div>
        <KdsButton
          leading-icon="x-close"
          variant="transparent"
          size="medium"
          title="Close"
          @click="onClose"
        />
      </header>

      <div :class="['modal-body', `modal-body-variant-${variant}`]">
        <slot />
      </div>

      <footer class="modal-footer">
        <slot name="footer" />
      </footer>
    </template>
  </dialog>
</template>

<style>
body:has(dialog.animated-modal[open]) {
  overflow: hidden;
}
</style>

<style scoped>
.animated-modal {
  --modal-full-size: calc(100vh - 60px);
  --modal-padding-left: var(--kds-spacing-container-1-5x);
  --modal-padding-right: var(--kds-spacing-container-1-5x);
  --modal-padding-top: var(--kds-spacing-container-0-5x);
  --modal-padding-bottom: var(--kds-spacing-container-1x);
  --modal-gap: var(--kds-spacing-container-1x);

  display: grid;
  grid-template-rows: auto 1fr auto;
  width: min(var(--modal-full-size), v-bind("cssModalWidth"));
  height: v-bind("cssModalHeight");
  max-height: var(--modal-full-size);
  padding: 0;
  overflow: hidden;
  font: var(--kds-font-base-body-medium);
  color: var(--kds-color-text-and-icon-neutral);
  background-color: var(--kds-color-surface-default);
  border: none;
  border-radius: var(--kds-border-radius-container-0-37x);
  box-shadow: var(--kds-elevation-level-3);

  &:not([open]) {
    display: none;
  }

  &:focus-visible,
  &:focus {
    outline: none;
  }

  &::backdrop {
    background: var(--kds-color-blanket-default);
  }

  /* Backdrop animations - fast and consistent regardless of modal size */
  &.backdrop-entering::backdrop {
    animation: backdropFadeIn 150ms ease-out;
  }

  &.backdrop-exiting::backdrop {
    animation: backdropFadeOut calc(v-bind("animationDuration + 'ms'") * 0.67) ease-out;
    animation-fill-mode: forwards;
  }

  & .modal-header {
    display: flex;
    gap: var(--kds-spacing-container-0-5x);
    align-items: center;
    padding: var(--kds-spacing-container-0-5x) var(--kds-spacing-container-0-5x)
      var(--kds-spacing-container-0-5x) var(--kds-spacing-container-1-5x);
    font: var(--kds-font-base-title-medium-strong);
    color: var(--kds-color-text-and-icon-neutral);

    & .modal-header-title {
      flex: 1 1 auto;
    }
  }

  & .modal-body {
    display: flex;
    flex-direction: column;
    overflow: hidden;
    font: var(--kds-font-base-body-medium);
    color: var(--kds-color-text-and-icon-neutral);

    &.modal-body-variant-default {
      gap: var(--modal-gap);
      padding: var(--modal-padding-top) var(--modal-padding-right)
        var(--modal-padding-bottom) var(--modal-padding-left);
      overflow-y: auto;
      overscroll-behavior: contain;
    }
  }

  & .modal-footer {
    display: flex;
    gap: var(--kds-spacing-container-0-5x);
    justify-content: right;
    padding: var(--kds-spacing-container-1x) var(--kds-spacing-container-1-5x);
  }
}

/* Fullscreen sizing */
.animated-modal.size-fullscreen {
  width: calc(100vw - 60px);
  height: calc(100vh - 60px);
  max-width: calc(100vw - 60px);
  max-height: calc(100vh - 60px);
}

@keyframes backdropFadeIn {
  from {
    opacity: 0;
  }
  to {
    opacity: 1;
  }
}

@keyframes backdropFadeOut {
  from {
    opacity: 1;
  }
  to {
    opacity: 0;
  }
}

@media (prefers-reduced-motion: reduce) {
  .animated-modal {
    transition: opacity 150ms ease-out !important;
  }
  
  .animated-modal.backdrop-entering::backdrop,
  .animated-modal.backdrop-exiting::backdrop {
    animation: none !important;
  }
}
</style>
