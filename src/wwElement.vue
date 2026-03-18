<template>
  <div class="pv-checkbox">
    <div class="pv-checkbox__inner">
      <PvCheckbox
        :modelValue="internalValue"
        :binary="true"
        :disabled="props.content?.disabled"
        :readonly="props.content?.readonly"
        unstyled
        :pt="passthrough"
        @update:modelValue="handleChange"
      />
      <input
        type="checkbox"
        :name="props.content?.fieldName"
        :checked="internalValue"
        :required="props.content?.required"
        tabindex="-1"
        class="pv-checkbox__fake-input"
      />
    </div>
  </div>
</template>

<script>
import { computed, inject, watch } from 'vue';
import { installPrimeVue } from '../shared/install-primevue.js';
import Checkbox from 'primevue/checkbox';

export default {
  components: {
    PvCheckbox: Checkbox,
  },
  props: {
    uid: { type: String, required: true },
    content: { type: Object, required: true },
    wwElementState: { type: Object, default: () => ({}) },
    /* wwEditor:start */
    wwEditorState: { type: Object, required: true },
    /* wwEditor:end */
  },
  emits: ['trigger-event'],
  setup(props, { emit }) {
    installPrimeVue();

    // ── Internal variable ──
    const { value: internalValue, setValue: setInternalValue } =
      wwLib.wwVariable.useComponentVariable({
        uid: props.uid,
        name: 'value',
        type: 'boolean',
        defaultValue: computed(() =>
          props.content?.initialValue === undefined ? false : !!props.content?.initialValue
        ),
      });

    // ── Sync initialValue ──
    watch(
      () => props.content?.initialValue,
      (newVal) => {
        const v = !!newVal;
        if (v !== internalValue.value) {
          setInternalValue(v);
          emit('trigger-event', { name: 'initValueChange', event: { value: v } });
        }
      },
      { immediate: true }
    );

    // ── Change handler ──
    const handleChange = (newVal) => {
      const v = !!newVal;
      if (v !== internalValue.value) {
        setInternalValue(v);
        emit('trigger-event', { name: 'change', event: { value: v } });
      }
    };

    // ── Form integration ──
    const fieldName = computed(() => props.content?.fieldName || props.wwElementState?.name);
    const validation = computed(() => props.content?.validation);
    const customValidation = computed(() => props.content?.customValidation);
    const initValue = computed(() => props.content?.initialValue ?? false);

    const useForm = inject('_wwForm:useForm', () => {});
    useForm(
      internalValue,
      { fieldName, validation, customValidation, initialValue: initValue },
      { elementState: props.wwElementState, emit, sidepanelFormPath: 'form', setValue: setInternalValue }
    );

    /* wwEditor:start */
    const selectForm = inject('_wwForm:selectForm', () => {});
    /* wwEditor:end */

    // ── PassThrough ──
    const passthrough = {
      root: { class: 'pv-checkbox__root' },
      input: { class: 'pv-checkbox__input' },
      box: { class: 'pv-checkbox__box' },
      icon: { class: 'pv-checkbox__icon' },
    };

    return {
      props,
      internalValue,
      handleChange,
      passthrough,
      /* wwEditor:start */
      selectForm,
      /* wwEditor:end */
    };
  },
};
</script>

<style scoped lang="scss">
@import '../shared/styles/base';
@import '../shared/styles/tokens';

.pv-checkbox {
  &__inner {
    position: relative;
    display: inline-flex;
    align-items: center;
  }

  &__root {
    display: inline-flex;
    align-items: center;
    cursor: pointer;

    &[data-p-disabled="true"] {
      opacity: 0.5;
      cursor: not-allowed;
    }
  }

  // PrimeVue internal elements — need :deep() for scoped style penetration
  :deep(.pv-checkbox__input) {
    position: absolute;
    opacity: 0;
    width: 0;
    height: 0;
  }

  :deep(.pv-checkbox__box) {
    display: flex;
    align-items: center;
    justify-content: center;
    width: 20px;
    height: 20px;
    border: 2px solid var(--pv-border, #e2e8f0);
    border-radius: var(--pv-radius-sm, 4px);
    background: var(--pv-surface, #f8fafc);
    transition: all var(--pv-transition, 150ms);
  }

  // Checked state
  :deep(.pv-checkbox__root[data-p-checked="true"] .pv-checkbox__box) {
    background-color: var(--pv-primary, #3B82F6);
    border-color: var(--pv-primary, #3B82F6);
  }

  :deep(.pv-checkbox__icon) {
    color: var(--pv-primary-contrast, #ffffff);

    svg {
      width: 12px;
      height: 12px;
    }
  }

  // Focus
  :deep(.pv-checkbox__root:focus-visible .pv-checkbox__box) {
    box-shadow: 0 0 0 2px var(--pv-focus-ring, rgba(59, 130, 246, 0.3));
  }

  // Hover
  :deep(.pv-checkbox__root:hover:not([data-p-disabled="true"]) .pv-checkbox__box) {
    border-color: var(--pv-primary, #3B82F6);
  }

  &__fake-input {
    background: rgba(0, 0, 0, 0);
    border: 0;
    bottom: -1px;
    font-size: 0;
    height: 1px;
    left: 0;
    outline: none;
    padding: 0;
    position: absolute;
    right: 0;
    width: 100%;
  }
}
</style>
