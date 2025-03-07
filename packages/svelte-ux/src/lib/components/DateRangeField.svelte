<script lang="ts">
  import { createEventDispatcher, type ComponentProps } from 'svelte';
  import { mdiCheck, mdiChevronLeft, mdiChevronRight, mdiClose } from '@mdi/js';

  import Button from './Button.svelte';
  import DateRange from './DateRange.svelte';
  import DateRangeDisplay from './DateRangeDisplay.svelte';
  import Dialog from './Dialog.svelte';
  import Field from './Field.svelte';

  import { PeriodType, getDateFuncsByPeriodType, getPeriodTypeName } from '../utils/date';
  import { getDateRangePresets, type DateRange as DateRangeType } from '../utils/dateRange';
  import { cls } from '../utils/styles';

  const dispatch = createEventDispatcher();

  const _defaultValue: DateRangeType = {
    from: null,
    to: null,
    periodType: null,
  };

  export let value: DateRangeType = _defaultValue;
  export let stepper: boolean = false;
  export let center: boolean = false;
  export let periodTypes: PeriodType[] = [
    PeriodType.Day,
    PeriodType.WeekSun,
    PeriodType.BiWeek1Sun,
    // PeriodType.BiWeek2Sun,
    PeriodType.Month,
    PeriodType.Quarter,
    PeriodType.CalendarYear,
    PeriodType.FiscalYearOctober,
  ];
  export let getPeriodTypePresets = getDateRangePresets;

  export let classes: {
    field?: ComponentProps<Field>['classes'];
    dialog?: ComponentProps<Dialog>['classes'];
  } = {};

  // Field props
  export let label: string | null = null;
  // export let value = '';
  export let error = '';
  export let hint = '';
  export let disabled = false;
  export let clearable = false;
  export let base = false;
  export let rounded = false;
  export let dense = false;
  export let icon: string | null = null;

  let open: boolean = false;
  let format: string = undefined;

  let currentValue = value;
</script>

<Field
  label={label ?? (value.periodType ? getPeriodTypeName(value.periodType) : '')}
  {icon}
  {error}
  {hint}
  {disabled}
  {base}
  {rounded}
  {dense}
  {center}
  classes={classes.field}
  let:id
  {...$$restProps}
>
  <span slot="prepend" class="flex items-center">
    <slot name="prepend" />

    {#if stepper}
      <Button
        icon={mdiChevronLeft}
        class="p-2"
        on:click={() => {
          if (value && value.from && value.to && value.periodType) {
            const { difference, start, end, add } = getDateFuncsByPeriodType(value.periodType);
            const offset = difference(value.from, value.to) - 1;
            value = {
              from: start(add(value.from, offset)),
              to: end(add(value.to, offset)),
              periodType: value.periodType,
            };
            dispatch('change', value);
          }
        }}
      />
    {/if}
  </span>

  <button
    type="button"
    class={cls(
      'text-sm whitespace-nowrap w-full focus:outline-none',
      center ? 'text-center' : 'text-left'
    )}
    on:click={() => (open = true)}
    {id}
  >
    <DateRangeDisplay {value} {format} />
  </button>

  <div slot="append" class="flex items-center">
    {#if clearable && (value?.periodType || value?.from || value?.to)}
      <Button
        icon={mdiClose}
        class="text-black/50 p-1"
        on:click={() => {
          value = _defaultValue;
          dispatch('clear');
          dispatch('change', value);
        }}
      />
    {/if}

    <slot name="append" />

    {#if stepper}
      <Button
        icon={mdiChevronRight}
        class="p-2"
        on:click={() => {
          if (value && value.from && value.to && value.periodType) {
            const { difference, start, end, add } = getDateFuncsByPeriodType(value.periodType);
            const offset = difference(value.to, value.from) + 1;
            value = {
              from: start(add(value.from, offset)),
              to: end(add(value.to, offset)),
              periodType: value.periodType,
            };
            dispatch('change', value);
          }
        }}
      />
    {/if}
  </div>
</Field>

<Dialog
  classes={{
    ...classes.dialog,
    dialog: cls('max-h-[90vh] grid grid-rows-[auto,1fr,auto]', classes.dialog?.dialog),
  }}
  bind:open
>
  <div class="flex flex-col justify-center bg-accent-500 text-white px-6 h-24">
    <div class="text-sm text-white/50">
      {currentValue.periodType ? getPeriodTypeName(currentValue.periodType) : ''}&nbsp;
    </div>
    <div class="text-xl sm:text-2xl text-white">
      <DateRangeDisplay value={currentValue} />
    </div>
  </div>

  <div class="p-2 border-b overflow-auto">
    <DateRange bind:selected={currentValue} {periodTypes} {getPeriodTypePresets} class="h-full" />
  </div>

  <div slot="actions" class="flex items-center gap-2">
    <Button
      icon={mdiCheck}
      on:click={() => {
        open = false;
        value = currentValue;
        dispatch('change', value);
      }}
      color="blue"
      variant="fill"
    >
      OK
    </Button>

    <Button
      on:click={() => {
        open = false;
        currentValue = value;
      }}
    >
      Cancel
    </Button>
  </div>
</Dialog>
