<script lang="ts">
  import { spring, tweened } from 'svelte/motion';
  import { elasticOut, backInOut, bounceOut } from 'svelte/easing';
  import { cls } from '$lib/utils/styles';
  import { modulo } from '$lib/utils/number';
  import { getComponentTheme } from './theme';

  export let value = 0;
  export let single = false;
  export let format: (value: number) => string | number = (value) => value;
  export let axis: 'x' | 'y' = 'y';

  export let classes: {
    root?: string;
    value?: string;
  } = {};
  const theme = getComponentTheme('ScrollingValue');

  const displayValue = spring();
  // 	const displayValue = tweened(value, { duration: 1000, easing: bounceOut });
  $: $displayValue = value;
  $: offset = modulo($displayValue, 1);

  $: nextDisplayValue = Math.floor(single && $displayValue >= 9 ? 0 : $displayValue + 1);
  $: currentDisplayValue = Math.floor($displayValue);
</script>

<div
  class={cls(
    'ScrollingValue',
    'inline-grid overflow-hidden',
    theme.root,
    classes.root,
    $$props.class
  )}
>
  <div
    class={cls('col-span-full row-span-full', theme.value, classes.value)}
    style:transform={axis === 'x'
      ? `translateX(${100 + 100 * -offset}%)`
      : `translateY(${-100 + 100 * offset}%)`}
  >
    <slot value={nextDisplayValue}>
      {format(nextDisplayValue)}
    </slot>
  </div>
  <div
    class={cls('col-span-full row-span-full', theme.value, classes.value)}
    style:transform={axis === 'x'
      ? `translateX(${100 * -offset}%)`
      : `translateY(${100 * offset}%)`}
  >
    <slot value={currentDisplayValue}>
      {format(currentDisplayValue)}
    </slot>
  </div>
</div>
