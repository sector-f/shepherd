<script>
  import { onMount } from 'svelte';
  import ShepherdContent from './shepherd-content.svelte';
  import { isUndefined } from '../utils/type-check.js';

  const KEY_TAB = 9;
  const KEY_ESC = 27;
  const LEFT_ARROW = 37;
  const RIGHT_ARROW = 39;

  export let classes, classPrefix, element, descriptionId, firstFocusableElement,
    focusableElements, labelId, lastFocusableElement, step;
  const dataStepId = { [`data-${classPrefix}shepherd-step-id`]: step.id };
  const hasCancelIcon = step.options && step.options.cancelIcon && step.options.cancelIcon.enabled;
  const hasTitle = step.options && step.options.title;

  export const getElement = () => element;

  onMount(() => {
    // Get all elements that are focusable
    focusableElements = element.querySelectorAll('a[href], area[href], input:not([disabled]), select:not([disabled]), textarea:not([disabled]), button:not([disabled]), [tabindex="0"]');
    firstFocusableElement = focusableElements[0];
    lastFocusableElement = focusableElements[focusableElements.length - 1];
  });

  /**
   * Setup keydown events to allow closing the modal with ESC
   *
   * Borrowed from this great post! https://bitsofco.de/accessible-modal-dialog/
   *
   * @private
   */
  const handleKeyDown = (e) => {
    const { tour } = step;
    switch (e.keyCode) {
      case KEY_TAB:
        if (focusableElements.length === 0) {
          e.preventDefault();
          break;
        }
        // Backward tab
        if (e.shiftKey) {
          if (document.activeElement === firstFocusableElement) {
            e.preventDefault();
            lastFocusableElement.focus();
          }
        } else {
          if (document.activeElement === lastFocusableElement) {
            e.preventDefault();
            firstFocusableElement.focus();
          }
        }
        break;
      case KEY_ESC:
        if (tour.options.exitOnEsc) {
          step.cancel();
        }
        break;
      case LEFT_ARROW:
        if (tour.options.keyboardNavigation) {
          tour.back();
        }
        break;
      case RIGHT_ARROW:
        if (tour.options.keyboardNavigation) {
          tour.next();
        }
        break;
      default:
        break;
    }
  };
</script>

<style global>
  .shepherd-element {
    background: #fff;
    border-radius: 5px;
    box-shadow: 0 1px 4px rgba(0, 0, 0, 0.2);
    max-width: 400px;
    outline: none;
    z-index: 9999;
  }

  .shepherd-element, .shepherd-element *, .shepherd-element *:after, .shepherd-element *:before {
    box-sizing: border-box;
  }

  .shepherd-element .shepherd-arrow {
    border: 16px solid transparent;
    content: '';
    display: block;
    height: 16px;
    pointer-events: none;
    position: absolute;
    width: 16px;
    z-index: 10000;
  }

  /**
   * Arrow on bottom of tooltip centered horizontally
   */
  .shepherd-element.shepherd-element-attached-bottom.shepherd-element-attached-center .shepherd-arrow,
  .shepherd-element.shepherd-pinned-top .shepherd-arrow {
    bottom: 0;
    border-top-color: #fff;
    left: 50%;
    transform: translate(-50%, 100%);
  }

  /**
   * Arrow on top of tooltip centered horizontally
   */
  .shepherd-element.shepherd-element-attached-top.shepherd-element-attached-center .shepherd-arrow {
    border-bottom-color: #fff;
    left: 50%;
    top: 0;
    transform: translate(-50%, -100%);
  }

  /**
  * Arrow on top of tooltip centered horizontally, with title color
  */
  .shepherd-element.shepherd-element-attached-top.shepherd-element-attached-center.shepherd-has-title .shepherd-arrow {
    border-bottom-color: #e6e6e6;
  }

  /**
   * Arrow on left of tooltip, centered vertically
   */
  .shepherd-element.shepherd-element-attached-middle.shepherd-element-attached-left .shepherd-arrow,
  .shepherd-element.shepherd-pinned-right .shepherd-arrow {
    border-right-color: #fff;
    left: 0;
    top: 50%;
    transform: translate(-100%, -50%);
  }

  /**
   * Arrow on right of tooltip, centered vertically
   */
  .shepherd-element.shepherd-element-attached-middle.shepherd-element-attached-right .shepherd-arrow,
  .shepherd-element.shepherd-pinned-left .shepherd-arrow {
    border-left-color: #fff;
    right: 0;
    top: 50%;
    transform: translate(100%, -50%);
  }

  .shepherd-target-click-disabled.shepherd-enabled.shepherd-target,
  .shepherd-target-click-disabled.shepherd-enabled.shepherd-target * {
    pointer-events: none;
  }
</style>

<div
  aria-describedby={!isUndefined(step.options.text) ? descriptionId : null}
  aria-labelledby={step.options.title ? labelId : null}
  bind:this={element}
  class="{`${classes} shepherd-element ${hasCancelIcon ? 'shepherd-has-cancel-icon' : ''} ${hasTitle ? 'shepherd-has-title' : ''}`}"
  {...dataStepId}
  on:keydown={handleKeyDown}
  role="dialog"
  tabindex="0"
>
    {#if step.options.arrow && step.options.attachTo && step.options.attachTo.element}
      <div class="shepherd-arrow"></div>
    {/if}
  <ShepherdContent
    {descriptionId}
    {labelId}
    {step}
  />
</div>
