<script lang='ts'>
  import Logger from '../utils/logger';
  import { getComponentTheme } from './theme';
  import { cls } from '../utils/styles';

  const logger = new Logger('SelectListOptions');

  export let optionText: (option: any) => string;
  export let optionValue: (option: any) => any;
	export let selectIndex: (index: number) => any;
	export let selectOption: (x: any) => any;
	export let onKeyDown: (x: KeyboardEvent) => void;
	export let onKeyPress: (x: KeyboardEvent) => void;

	export let open: boolean;
  export let loading: boolean;
	export let filteredOptions: any[];

  export let value: any = undefined;
  export let selected: any = undefined;

  export let highlightIndex: number;
  export let searchText: string;

  export let classes: {
    root?: string;
    option?: string;
    selected?: string;
    group?: string;
    empty?: string;
  } = {};

  const theme = getComponentTheme('SelectField');

	export let menuOptionsEl: HTMLDivElement;
</script>

<div
	role='listbox'
	tabindex='-1'
	aria-expanded={open ? 'true' : 'false'}
	class={cls('_SelectListOptions options group p-1 focus:outline-none', theme.options, classes.root)}
	class:opacity-50={loading}
	bind:this={menuOptionsEl}
	on:click|stopPropagation={(e) => {
		logger.debug('options container clicked');
	
		if (e.target instanceof HTMLElement) {
			// Find slot parent of click target option, fallback to `e.target` if slot is not overridden
			// Use `.options > ` in case slot is nested (ex. GraphQLSelect with slot)
			const slotEl = e.target.closest('.options > [slot=option]') ?? e.target;
			// Find the index of the clicked on element (ignoring group headers)
			const optionIndex = slotEl
				? [...menuOptionsEl.children]
						.filter((el) => !el.classList.contains('group-header'))
						.indexOf(slotEl)
				: -1;
			logger.debug({ slotEl, optionIndex });
			// ignore clicks on group options
			if (optionIndex !== -1) {
				selectIndex(optionIndex);
			}
		}
	}}
	on:keydown={e => { 
		logger.debug('keydown: calling given onKeyDown...');
		onKeyDown(e);
	}}
	on:keypress={e => {
		logger.debug('keypress: calling given onKeyPress...');
		onKeyPress(e);
	}}
>
	{#each filteredOptions ?? [] as option, index (optionValue(option))}
		{@const previousOption = filteredOptions[index - 1]}
		{#if option.group && option.group !== previousOption?.group}
			<div
				class={cls(
					'group-header text-xs leading-8 tracking-widest text-black/50 px-2',
					theme.group,
					classes.group
				)}
			>
				{option.group}
			</div>
		{/if}

		<slot name="option" {option} {index}></slot>
	{:else}
		<slot name="empty" {loading} {searchText}></slot>
	{/each}
</div>