<template>
  <div class="ui-dropdown" :class="classes" v-on="$listeners">
    <div
      ref="trigger"
      class="ui-dropdown__trigger"
      :class="triggerClass"
      v-on="trigger_listeners"
    >
      <slot v-bind="{ shown: local_shown }" />
    </div>

    <client-only>
      <component
        :is="mountSelf ? fade_transition : 'mounting-portal'"
        :transition="fade_transition"
        :mount-to="mountTo"
        append
      >
        <div
          v-if="local_shown && !disabled"
          ref="portal"
          class="ui-dropdown-portal"
          :class="portal_classes"
          :style="portal_styles"
          v-on="portal_listeners"
        >
          <div class="ui-dropdown-portal__layout">
            <slot name="portal" v-bind="{ hide }" />

            <div v-if="arrow" class="ui-dropdown-portal__arrow" />
          </div>
        </div>
      </component>
    </client-only>
  </div>
</template>

<script>
import { MountingPortal } from 'portal-vue';
import { ResizeObserver } from '@juggle/resize-observer';
import ClientOnly from 'vue-client-only';

const positions = ['top', 'right', 'bottom', 'left'];
const alignments = ['start', 'center', 'end', 'stretch'];

const placements = positions.reduce((acc, current) => [...acc, ...alignments.map((align) => `${current}-${align}`)], []);

export default {
  components: {
    MountingPortal,
    ClientOnly,
  },
  inheritAttrs: false,
  props: {
    shown: {
      type: Boolean,
      default: false,
    },
    arrow: {
      type: Boolean,
      default: true,
    },
    placement: {
      type: String,
      default: 'bottom-center',
      validator(value) {
        const [placement, fallback] = value.split('|');

        return fallback
          ? placements.includes(placement) && placements.includes(fallback)
          : placements.includes(placement);
      },
    },
    trigger: {
      type: String,
      default: 'click',
      validator(value) {
        return ['hover', 'click', ''].includes(value);
      },
    },
    relativeEl: {
      type: [String, global.Element],
      default: '',
    },
    viewportEl: {
      type: [String, global.Element],
      default: '',
    },
    documentTargets: {
      type: Array,
      default: () => ['portal', 'trigger'],
    },
    offset: {
      type: Array,
      default: () => [0, 0, 0, 0],
    },
    viewportOffset: {
      type: Array,
      default: () => [0, 0, 0, 0],
    },
    portalClass: {
      type: [Array, String, Object],
      default: '',
    },
    triggerClass: {
      type: [Array, String, Object],
      default: '',
    },
    autoHide: {
      type: Boolean,
      default: true,
    },
    hoverPortal: {
      type: Boolean,
      default: false,
    },
    clickPortal: {
      type: Boolean,
      default: false,
    },
    disabled: {
      type: Boolean,
      default: false,
    },
    noPadding: {
      type: Boolean,
      default: false,
    },
    zIndex: {
      type: [Number, String],
      default: 50,
    },
    transition: {
      type: String,
      default: 'fade',
    },
    transitionMode: {
      type: String,
      default: 'in-out',
    },
    mountTo: {
      type: String,
      default: 'body',
    },
    mountSelf: {
      type: Boolean,
      default: false,
    },
  },
  data() {
    return {
      local_shown: this.shown,
      trigger_hover_timer: null,
      portal_hover_timer: null,
      resize_observers: {
        portal: null,
        trigger: null,
        viewport: null,
      },
      coords: {
        document: {
          width: 0,
          height: 0,
          scrollTop: 0,
          scrollRight: 0,
          scrollBottom: 0,
          scrollLeft: 0,
        },
        portal: {
          width: 0,
          height: 0,
        },
        trigger: {
          width: 0,
          height: 0,
          top: 0,
          right: 0,
          bottom: 0,
          left: 0,
        },
        viewport: {
          height: 0,
          width: 0,
          top: 0,
          right: 0,
          bottom: 0,
          left: 0,
        },
      },
    };
  },
  computed: {
    fade_transition() {
      return {
        functional: true,
        render: (h, { children }) => {
          const props = {
            name: this.transition,
            mode: this.transitionMode,
          };

          const on = {
            enter: this.onShow,
            leave: this.onHide,
          };

          return h('transition', { props, on }, children);
        },
      };
    },
    position() {
      const [placement] = this.placement.split('|');

      return placement.split('-')[0];
    },
    fallback_position() {
      const [placement, fallback] = this.placement.split('|');

      return (fallback || placement).split('-')[0];
    },
    alignment() {
      const [placement] = this.placement.split('|');

      return placement.split('-')[1];
    },
    fallback_alignment() {
      const [placement, fallback] = this.placement.split('|');

      return (fallback || placement).split('-')[1];
    },
    portal_position_coords_top() {
      const { trigger, portal } = this.coords;
      const { scrollTop, scrollLeft } = this.coords.document;

      return {
        top: trigger.top - portal.height + scrollTop,
        left: trigger.left + scrollLeft,
      };
    },
    portal_position_coords_right() {
      const { trigger } = this.coords;
      const { scrollTop, scrollLeft } = this.coords.document;

      return {
        top: trigger.top + scrollTop,
        left: trigger.right + scrollLeft,
      };
    },
    portal_position_coords_bottom() {
      const { trigger } = this.coords;
      const { scrollTop, scrollLeft } = this.coords.document;

      return {
        top: trigger.bottom + scrollTop,
        left: trigger.left + scrollLeft,
      };
    },
    portal_position_coords_left() {
      const { trigger, portal } = this.coords;
      const { scrollTop, scrollLeft } = this.coords.document;

      return {
        top: trigger.top + scrollTop,
        left: trigger.left - portal.width + scrollLeft,
      };
    },
    portal_position_styles_top() {
      if (this.mountSelf) {
        return {
          top: -this.coords.portal.height,
          left: 0,
        };
      }
      return this.portal_position_coords_top;
    },
    portal_position_styles_right() {
      if (this.mountSelf) {
        return {
          top: 0,
          left: this.coords.trigger.width,
        };
      }
      return this.portal_position_coords_right;
    },
    portal_position_styles_bottom() {
      if (this.mountSelf) {
        return {
          top: this.coords.trigger.height,
          left: 0,
        };
      }
      return this.portal_position_coords_bottom;
    },
    portal_position_styles_left() {
      if (this.mountSelf) {
        return {
          top: 0,
          left: -this.coords.portal.width,
        };
      }
      return this.portal_position_coords_left;
    },
    portal_alignment_styles_x_start() {
      return { left: 0, top: 0 };
    },
    portal_alignment_styles_x_center() {
      const { trigger, portal } = this.coords;

      return {
        left: 0,
        top: trigger.height / 2 - portal.height / 2,
      };
    },
    portal_alignment_styles_x_end() {
      const { trigger, portal } = this.coords;

      return {
        left: 0,
        top: trigger.height - portal.height,
      };
    },
    portal_alignment_styles_x_stretch() {
      return {
        left: 0,
        top: 0,
        height: this.coords.trigger.height,
      };
    },
    portal_alignment_styles_y_start() {
      return { left: 0, top: 0 };
    },
    portal_alignment_styles_y_center() {
      const { trigger, portal } = this.coords;

      return {
        left: trigger.width / 2 - portal.width / 2,
        top: 0,
      };
    },
    portal_alignment_styles_y_end() {
      const { trigger, portal } = this.coords;

      return {
        left: trigger.width - portal.width,
        top: 0,
      };
    },
    portal_alignment_styles_y_stretch() {
      return {
        left: 0,
        top: 0,
        width: this.coords.trigger.width,
      };
    },
    portal_in_viewport_top() {
      const { top } = this.portal_position_coords_top;
      const { scrollTop } = this.coords.document;

      return top >= scrollTop + this.coords.viewport.top + this.viewportOffset[0];
    },
    portal_in_viewport_right() {
      const { document, viewport, portal } = this.coords;
      const { left } = this.portal_position_coords_right;
      const { scrollRight } = document;
      const right = left + portal.width;
      const viewportRight = viewport.right ? document.width - viewport.right : 0;

      return right <= scrollRight - viewportRight - this.viewportOffset[1];
    },
    portal_in_viewport_bottom() {
      const { document, viewport, portal } = this.coords;
      const { top } = this.portal_position_coords_bottom;
      const { scrollBottom } = document;
      const bottom = top + portal.height;
      const viewportBottom = viewport.bottom ? document.height - viewport.bottom : 0;

      return bottom <= scrollBottom - viewportBottom - this.viewportOffset[2];
    },
    portal_in_viewport_left() {
      const { left } = this.portal_position_coords_left;
      const { scrollLeft } = this.coords.document;

      return left >= scrollLeft + this.coords.viewport.left + this.viewportOffset[3];
    },
    in_viewport() {
      return this[`portal_in_viewport_${this.position}`];
    },
    computed_position() {
      return this.in_viewport ? this.position : this.fallback_position;
    },
    computed_alignment() {
      return this.in_viewport ? this.alignment : this.fallback_alignment;
    },
    computed_offset() {
      return this.in_viewport
        ? [this.offset[0], this.offset[1]]
        : [this.offset[2] || 0, this.offset[3] || 0];
    },
    portal_offset_styles_top() {
      return {
        left: this.computed_offset[0],
        top: -this.computed_offset[1],
      };
    },
    portal_offset_styles_right() {
      return {
        left: this.computed_offset[0],
        top: this.computed_offset[1],
      };
    },
    portal_offset_styles_bottom() {
      return {
        left: this.computed_offset[0],
        top: this.computed_offset[1],
      };
    },
    portal_offset_styles_left() {
      return {
        left: -this.computed_offset[0],
        top: this.computed_offset[1],
      };
    },
    portal_styles() {
      const axises = {
        top: 'y',
        bottom: 'y',
        left: 'x',
        right: 'x',
      };

      const axis = axises[this.computed_position];
      const position = this[`portal_position_styles_${this.computed_position}`];
      const alignment = this[
        `portal_alignment_styles_${axis}_${this.computed_alignment}`
      ];
      const offset = this[`portal_offset_styles_${this.computed_position}`];

      return {
        top: `${position.top + alignment.top + offset.top}px`,
        left: `${position.left + alignment.left + offset.left}px`,
        width: alignment.width ? `${alignment.width}px` : undefined,
        height: alignment.height ? `${alignment.height}px` : undefined,
        zIndex: this.zIndex,
      };
    },
    portal_classes() {
      return [
        this.portalClass,
        `ui-dropdown-portal--${this.computed_position}`,
        `ui-dropdown-portal--${this.computed_alignment}`,
        {
          'ui-dropdown-portal--arrow': this.arrow,
          'ui-dropdown-portal--no-padding': this.noPadding,
        },
      ];
    },
    classes() {
      return {
        'ui-dropdown--shown': this.local_shown,
        'ui-dropdown--disabled': this.disabled,
        'ui-dropdown--self': this.mountSelf,
      };
    },
    trigger_listeners_hover() {
      return {
        mouseenter: this.onTriggerMouseenter,
        mouseleave: this.onTriggerMouseleave,
      };
    },
    trigger_listeners_click() {
      return {
        click: this.toggle,
      };
    },
    trigger_listeners() {
      return this[`trigger_listeners_${this.trigger}`];
    },
    portal_listeners_hover() {
      return {
        mouseenter: this.onPortalMouseenter,
        mouseleave: this.onPortalMouseleave,
      };
    },
    portal_listeners_click() {
      return {
        click: this.onPortalClick,
      };
    },
    portal_listeners() {
      return this[`portal_listeners_${this.trigger}`];
    },
  },
  watch: {
    local_shown(value) {
      this.$emit('update:shown', value);
    },
    shown(value) {
      this.local_shown = value;
    },
  },
  methods: {
    getViewportEl() {
      return typeof this.viewportEl === 'string' && this.viewportEl
        ? this.$el.closest(this.viewportEl)
        : this.viewportEl;
    },
    getRelativeEl() {
      return typeof this.relativeEl === 'string' && this.relativeEl
        ? this.$el.closest(this.relativeEl)
        : this.relativeEl;
    },
    toggle() {
      if (!this.disabled) {
        this.local_shown = !this.local_shown;
      }
    },
    show() {
      if (!this.disabled) {
        this.local_shown = true;
      }
    },
    hide() {
      if (!this.disabled) {
        this.local_shown = false;
      }
    },
    onTriggerMouseenter() {
      if (this.portal_hover_timer) {
        clearInterval(this.portal_hover_timer);
      } else {
        this.show();
      }
    },
    onTriggerMouseleave() {
      if (this.hoverPortal) {
        this.trigger_hover_timer = setTimeout(() => {
          this.portal_hover_timer = null;

          this.hide();
        }, 100);
      } else {
        this.hide();
      }
    },
    onPortalMouseenter() {
      clearInterval(this.trigger_hover_timer);
    },
    onPortalMouseleave() {
      this.portal_hover_timer = setTimeout(() => {
        this.portal_hover_timer = null;

        this.hide();
      }, 100);
    },
    onPortalClick() {
      if (this.clickPortal) {
        this.hide();
      }
    },
    onDocumentClick(event) {
      const selectors = {
        trigger: '.ui-dropdown__trigger',
        portal: '.ui-dropdown-portal',
      };

      const elements = this.documentTargets.map((key) => this.$refs[key]);

      const targets = this.documentTargets.map((key) => event.target.closest(selectors[key]));

      if (!targets.some((target) => elements.includes(target))) {
        this.hide();
      }
    },
    onDocumentChangeCoords() {
      const { clientWidth, clientHeight } = document.documentElement;
      const { pageYOffset, pageXOffset } = window;

      this.coords.document = {
        width: clientWidth,
        height: clientHeight,
        scrollTop: pageYOffset,
        scrollRight: pageXOffset + clientWidth,
        scrollLeft: pageXOffset,
        scrollBottom: pageYOffset + clientHeight,
      };
    },
    onPortalChangeCoords() {
      if (this.$refs.portal) {
        const { portal } = this.$refs;
        const { width, height } = portal.getBoundingClientRect();

        this.coords.portal = { width, height };
      }
    },
    onTriggerChangeCoords() {
      const relativeEl = this.getRelativeEl() || this.$refs.trigger;

      if (relativeEl) {
        const {
          top, right, bottom, left, width, height,
        } = relativeEl.getBoundingClientRect();

        this.coords.trigger = {
          width,
          height,
          top,
          right,
          bottom,
          left,
        };
      }
    },
    onViewportElChangeCoords() {
      const viewportParent = this.getViewportEl();
      const {
        top, right, bottom, left, width, height,
      } = viewportParent.getBoundingClientRect();

      this.coords.viewport = {
        width,
        height,
        top,
        right,
        bottom,
        left,
      };
    },
    observeDocumentClick() {
      if (this.autoHide) {
        document.addEventListener('click', this.onDocumentClick, true);
      }
    },
    unobserveDocumentClick() {
      if (this.autoHide) {
        document.removeEventListener('click', this.onDocumentClick, true);
      }
    },
    observeDocumentCoords() {
      window.addEventListener('resize', this.onDocumentChangeCoords);
      window.addEventListener('scroll', this.onDocumentChangeCoords);

      window.addEventListener('scroll', this.onTriggerChangeCoords, true);
      window.addEventListener('scroll', this.onPortalChangeCoords, true);

      this.onDocumentChangeCoords();
    },
    unobserveDocumentCoords() {
      window.removeEventListener('resize', this.onDocumentChangeCoords);
      window.removeEventListener('scroll', this.onDocumentChangeCoords);

      window.removeEventListener('scroll', this.onTriggerChangeCoords, true);
      window.removeEventListener('scroll', this.onPortalChangeCoords, true);
    },
    observePortalCoords() {
      this.resize_observers.portal = new ResizeObserver(
        this.onPortalChangeCoords,
      );

      this.resize_observers.portal.observe(this.$refs.portal);
      this.resize_observers.portal.observe(document.documentElement);
    },
    unobservePortalCoords() {
      this.resize_observers.portal.disconnect();
    },
    observeTriggerCoords() {
      this.resize_observers.trigger = new ResizeObserver(
        this.onTriggerChangeCoords,
      );

      this.resize_observers.trigger.observe(this.$refs.trigger);
      this.resize_observers.trigger.observe(document.documentElement);
    },
    unobserveTriggerCoords() {
      this.resize_observers.trigger.disconnect();
    },
    observeViewportElCoords() {
      const viewportParent = this.getViewportEl();

      if (this.viewportEl && viewportParent) {
        this.resize_observers.viewport = new ResizeObserver(
          this.onViewportElChangeCoords,
        );

        this.resize_observers.viewport.observe(viewportParent);
      }
    },
    unobserveViewportElCoords() {
      const viewportParent = this.getViewportEl();

      if (this.viewportEl && viewportParent) {
        this.resize_observers.viewport.disconnect();
      }
    },
    onShow() {
      this.observeDocumentClick();
      this.observeDocumentCoords();
      this.observePortalCoords();
      this.observeTriggerCoords();
      this.observeViewportElCoords();

      this.$emit('show');
    },
    onHide() {
      this.unobserveDocumentClick();
      this.unobserveDocumentCoords();
      this.unobservePortalCoords();
      this.unobserveTriggerCoords();
      this.unobserveViewportElCoords();

      this.$emit('hide');
    },
  },
};
</script>

<style lang="sass" scoped>
.fade-enter-active,
.fade-leave-active
  transition: opacity 0.2s

.fade-enter,
.fade-leave-to
  opacity: 0

.ui-dropdown
  &--self
    position: relative

  &__trigger
    cursor: pointer

.ui-dropdown-portal
  position: absolute

  $parent: &

  &--arrow
    &#{$parent}--top
      padding-bottom: 5px

    &#{$parent}--bottom
      padding-top: 5px

    &#{$parent}--left
      padding-right: 5px

    &#{$parent}--right
      padding-left: 5px

  &:not(&--no-padding)
    #{$parent}__layout
      padding: 24px

  &__layout
    box-shadow: 0px 1px 20px rgba(51, 51, 51, 0.2)
    background: #FFFFFF
    border-radius: 10px
    overflow: hidden

  &__arrow
    position: absolute
    border-color: #FFFFFF
    border-style: solid
    width: 0
    height: 0

  &--top
    #{$parent}__arrow
      bottom: 0
      border-width: 5px 10px 0 10px
      border-left-color: transparent
      border-right-color: transparent
      border-bottom-color: transparent

  &--bottom
    #{$parent}__arrow
      top: 0
      border-width: 0 10px 5px 10px
      border-left-color: transparent
      border-right-color: transparent
      border-top-color: transparent

  &--top,
  &--bottom
    &#{$parent}--stretch,
    &#{$parent}--center
      #{$parent}__arrow
        margin: auto
        left: 0
        right: 0

    &#{$parent}--start
      #{$parent}__arrow
        left: 15px

    &#{$parent}--end
      #{$parent}__arrow
        right: 15px

  &--right
    #{$parent}__arrow
      left: 0
      border-width: 10px 5px 10px 0
      border-left-color: transparent
      border-top-color: transparent
      border-bottom-color: transparent

  &--left
    #{$parent}__arrow
      right: 0
      border-width: 10px 0 10px 5px
      border-top-color: transparent
      border-right-color: transparent
      border-bottom-color: transparent

  &--right,
  &--left
    &#{$parent}--stretch
      #{$parent}__layout
        height: 100%

    &#{$parent}--stretch,
    &#{$parent}--center
      #{$parent}__arrow
        margin: auto
        top: 0
        bottom: 0

    &#{$parent}--start
      #{$parent}__arrow
        top: 15px

    &#{$parent}--end
      #{$parent}__arrow
        bottom: 15px
</style>
