<template>
  <span>
    <transition
      :name="transition"
      @after-enter="handleAfterEnter"
      @after-leave="handleAfterLeave">
      <div
        class="el-popover el-popper"
        :class="[popperClass, content && 'el-popover--plain']"
        ref="popper"
        v-show="!disabled && showPopper"
        :style="{ width: width + 'px' }"
        role="tooltip"
        :id="tooltipId"
        :aria-hidden="(disabled || !showPopper) ? 'true' : 'false'"
      >
        <div class="el-popover__title" v-if="title" v-text="title"></div>
        <slot>{{ content }}</slot>
      </div>
    </transition>
    <slot name="reference"></slot>
  </span>
</template>
<script>
import Popper from 'element-ui/src/utils/vue-popper';
import { on, off } from 'element-ui/src/utils/dom';
import { addClass, removeClass } from 'element-ui/src/utils/dom';
import { generateId } from 'element-ui/src/utils/util';

export default {
  name: 'ElPopover',

  mixins: [Popper],

  props: {
    trigger: {
      type: String,
      default: 'click',
      validator: value => ['click', 'focus', 'hover', 'manual'].indexOf(value) > -1
    },
    title: String,
    disabled: Boolean,
    content: String,
    reference: {},
    popperClass: String,
    width: {},
    visibleArrow: {
      default: true
    },
    arrowOffset: {
      type: Number,
      default: 0
    },
    transition: {
      type: String,
      default: 'fade-in-linear'
    },
    tabindex: {
      type: Number,
      default: 0
    }
  },

  computed: {
    tooltipId() {
      return `el-popover-${generateId()}`;
    }
  },
  watch: {
    showPopper(val) {
      if (this.disabled) {
        return;
      }
      val ? this.$emit('show') : this.$emit('hide');
    },
    trigger() {
      this.removeEvents();
      this.createEvents();
    }
  },

  mounted() {
    this.createEvents();
  },

  beforeUpdate() {
    this.removeEvents();
  },

  updated() {
    this.createEvents();
  },

  beforeDestroy() {
    this.removeEvents();
  },

  methods: {
    createEvents() {
      const popper = this.getPopper();
      const reference = this.getReference();

      // 可访问性
      if (reference) {
        addClass(reference, 'el-popover__reference');
        reference.setAttribute('aria-describedby', this.tooltipId);
        reference.setAttribute('tabindex', this.tabindex); // tab序列
        popper.setAttribute('tabindex', 0);

        if (this.trigger !== 'click') {
          on(reference, 'focusin', () => {
            this.handleFocus();
            const instance = reference.__vue__;
            if (instance && typeof instance.focus === 'function') {
              instance.focus();
            }
          });
          on(popper, 'focusin', this.handleFocus);
          on(reference, 'focusout', this.handleBlur);
          on(popper, 'focusout', this.handleBlur);
        }
        on(reference, 'keydown', this.handleKeydown);
        on(reference, 'click', this.handleClick);
      }
      if (this.trigger === 'click') {
        on(reference, 'click', this.doToggle);
        on(document, 'click', this.handleDocumentClick);
      } else if (this.trigger === 'hover') {
        on(reference, 'mouseenter', this.handleMouseEnter);
        on(popper, 'mouseenter', this.handleMouseEnter);
        on(reference, 'mouseleave', this.handleMouseLeave);
        on(popper, 'mouseleave', this.handleMouseLeave);
      } else if (this.trigger === 'focus') {
        if (this.tabindex < 0) {
          console.warn('[Element Warn][Popover]a negative tabindex means that the element cannot be focused by tab key');
        }
        if (reference.querySelector('input, textarea')) {
          on(reference, 'focusin', this.doShow);
          on(reference, 'focusout', this.doClose);
        } else {
          on(reference, 'mousedown', this.doShow);
          on(reference, 'mouseup', this.doClose);
        }
      }
    },
    removeEvents() {
      const reference = this.getReference();

      off(reference, 'click', this.doToggle);
      off(reference, 'mouseup', this.doClose);
      off(reference, 'mousedown', this.doShow);
      off(reference, 'focusin', this.doShow);
      off(reference, 'focusout', this.doClose);
      off(reference, 'mousedown', this.doShow);
      off(reference, 'mouseup', this.doClose);
      off(reference, 'mouseleave', this.handleMouseLeave);
      off(reference, 'mouseenter', this.handleMouseEnter);
      off(document, 'click', this.handleDocumentClick);
    },

    handleFocus() {
      addClass(this.getReference(), 'focusing');
      if (this.trigger === 'click' || this.trigger === 'focus') this.showPopper = true;
    },
    handleClick() {
      removeClass(this.getReference(), 'focusing');
    },
    handleBlur() {
      removeClass(this.getReference(), 'focusing');
      if (this.trigger === 'click' || this.trigger === 'focus') this.showPopper = false;
    },
    handleMouseEnter() {
      this.doShow();
    },
    handleKeydown(ev) {
      if (ev.keyCode === 27 && this.trigger !== 'manual') { // esc
        this.doClose();
      }
    },
    handleMouseLeave() {
      this.doClose();
    },
    handleDocumentClick(e) {
      const reference = this.getReference();
      const popper = this.getPopper();

      if (!this.$el ||
        !reference ||
        this.$el.contains(e.target) ||
        reference.contains(e.target) ||
        !popper ||
        popper.contains(e.target)) return;
      this.showPopper = false;
    },
    handleAfterEnter() {
      this.$emit('after-enter');
    },
    handleAfterLeave() {
      this.$emit('after-leave');
      this.doDestroy();
    },
  },
};
</script>
