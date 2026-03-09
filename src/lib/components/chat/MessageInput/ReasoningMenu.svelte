<script lang="ts">
	import { getContext } from 'svelte';
	import type { Writable } from 'svelte/store';
	import type { i18n as i18nType } from 'i18next';
	import { DropdownMenu } from 'bits-ui';

	import type { Model } from '$lib/stores';
	import { showControls } from '$lib/stores';
	import { flyAndScale } from '$lib/utils/transitions';

	import Dropdown from '$lib/components/common/Dropdown.svelte';
	import Tooltip from '$lib/components/common/Tooltip.svelte';
	import LightBulb from '$lib/components/icons/LightBulb.svelte';

	const i18n = getContext<Writable<i18nType>>('i18n');

	export let params: Record<string, any> = {};
	export let selectedModel: Model | null = null;
	export let onClose: Function = () => {};

	let show = false;

	const STANDARD_REASONING_EFFORTS = ['low', 'medium', 'high'];

	const presetButtonClass = (active: boolean) =>
		active
			? 'bg-gray-900 text-white dark:bg-white dark:text-black'
			: 'bg-gray-100 text-gray-700 hover:bg-gray-200 dark:bg-gray-800 dark:text-gray-300 dark:hover:bg-gray-700';

	const titleCase = (value: string) => value.charAt(0).toUpperCase() + value.slice(1);

	const setThink = (value: boolean | null) => {
		params = { ...params, think: value };
	};

	const setReasoningEffort = (value: string | null) => {
		params = { ...params, reasoning_effort: value };
	};

	const openFullControls = async () => {
		show = false;
		showControls.set(true);
	};

	$: isOllamaModel = selectedModel?.owned_by === 'ollama';
	$: isOpenAIModel = selectedModel?.owned_by === 'openai';

	$: normalizedReasoningEffort =
		typeof params?.reasoning_effort === 'string' ? params.reasoning_effort.toLowerCase() : null;
	$: thinkIsCustom = typeof params?.think === 'string';
	$: reasoningEffortIsCustom =
		(params?.reasoning_effort ?? null) !== null &&
		!STANDARD_REASONING_EFFORTS.includes(normalizedReasoningEffort ?? '');

	$: isActive =
		(isOllamaModel && (params?.think ?? null) !== null) ||
		(isOpenAIModel && (params?.reasoning_effort ?? null) !== null);

	$: buttonLabel = (() => {
		if (isOllamaModel) {
			if (params?.think === true) {
				return $i18n.t('Think');
			}
			if (params?.think === false) {
				return $i18n.t('Off');
			}
			if (thinkIsCustom) {
				return $i18n.t('Custom');
			}
		}

		if (isOpenAIModel) {
			if (normalizedReasoningEffort && STANDARD_REASONING_EFFORTS.includes(normalizedReasoningEffort)) {
				return titleCase(normalizedReasoningEffort);
			}
			if (reasoningEffortIsCustom) {
				return $i18n.t('Custom');
			}
		}

		return '';
	})();

	$: bulbToneClass = (() => {
		if (isOllamaModel && params?.think === true) {
			return 'text-emerald-600 dark:text-emerald-400';
		}

		if (isOllamaModel && params?.think === false) {
			return 'text-red-600 dark:text-red-400';
		}

		return '';
	})();
</script>

<div class="ml-1 flex gap-1.5">
	<Dropdown
		bind:show
		on:change={(e) => {
			if (e.detail === false) {
				onClose();
			}
		}}
		align="end"
	>
		<Tooltip content={$i18n.t('Reasoning')} placement="top">
			<button
				type="button"
				id="reasoning-menu-button"
				class="flex items-center gap-1.5 translate-y-[1px] hover:bg-gray-50 dark:hover:bg-gray-850 text-sm transition rounded-lg cursor-pointer {buttonLabel !== '' || isActive
					? 'px-2.5 py-1 text-amber-700 dark:text-amber-300'
					: 'p-2 opacity-70 text-gray-700 dark:text-white'}"
				aria-label={$i18n.t('Reasoning')}
			>
				<LightBulb className={`size-4 ${bulbToneClass}`.trim()} strokeWidth="1.75" />

					{#if buttonLabel !== ''}
						<span class="truncate text-[13px] max-w-[100px] text-gray-700 dark:text-white">
							{buttonLabel}
						</span>
					{/if}
				</button>
		</Tooltip>

		<div slot="content">
			<DropdownMenu.Content
				class="w-full max-w-72 rounded-2xl px-1 py-1 border border-gray-100 dark:border-gray-800 z-50 bg-white dark:bg-gray-850 dark:text-white shadow-lg transition"
				sideOffset={4}
				side="bottom"
				align="end"
				transition={flyAndScale}
			>
				<div class="px-3 py-2.5 space-y-3">
					<div class="flex items-start justify-between gap-3">
						<div class="min-w-0">
							<div
								class="text-[10px] font-medium text-gray-400 dark:text-gray-500 uppercase tracking-wider"
							>
								{$i18n.t('Quick Reasoning')}
							</div>
							<div class="text-sm font-medium truncate">{selectedModel?.name}</div>
						</div>

						<button
							type="button"
							class="text-xs text-gray-500 hover:text-gray-700 dark:text-gray-400 dark:hover:text-gray-200 transition"
							on:click={openFullControls}
						>
							{$i18n.t('Open full controls')}
						</button>
					</div>

					{#if isOllamaModel}
						<div class="space-y-2">
							<div class="flex items-center justify-between gap-2">
								<div class="text-sm font-medium">think</div>
								<div class="text-[11px] text-gray-500 dark:text-gray-400">
									{$i18n.t('Ollama')}
								</div>
							</div>

							<div class="grid grid-cols-3 gap-1">
								<button
									type="button"
									class="rounded-xl px-2.5 py-2 text-xs font-medium transition {presetButtonClass((params?.think ?? null) === null)}"
									on:click={() => setThink(null)}
								>
									{$i18n.t('Default')}
								</button>
								<button
									type="button"
									class="rounded-xl px-2.5 py-2 text-xs font-medium transition {presetButtonClass(params?.think === true)}"
									on:click={() => setThink(true)}
								>
									{$i18n.t('On')}
								</button>
								<button
									type="button"
									class="rounded-xl px-2.5 py-2 text-xs font-medium transition {presetButtonClass(params?.think === false)}"
									on:click={() => setThink(false)}
								>
									{$i18n.t('Off')}
								</button>
							</div>

							{#if thinkIsCustom}
								<div class="rounded-xl border border-dashed border-amber-200 dark:border-amber-900/60 px-3 py-2">
									<div class="text-[11px] font-medium text-amber-700 dark:text-amber-300">
										{$i18n.t('Custom')}
									</div>
									<div class="text-xs font-mono break-all">{params.think}</div>
									<div class="mt-1 text-[11px] text-gray-500 dark:text-gray-400">
										{$i18n.t('Selecting a preset will overwrite the custom value.')}
									</div>
								</div>
							{/if}
						</div>
					{/if}

					{#if isOpenAIModel}
						<div class="space-y-2">
							<div class="flex items-center justify-between gap-2">
								<div class="text-sm font-medium">{$i18n.t('Reasoning Effort')}</div>
								<div class="text-[11px] text-gray-500 dark:text-gray-400">
									{$i18n.t('OpenAI')}
								</div>
							</div>

							<div class="grid grid-cols-4 gap-1">
								<button
									type="button"
									class="rounded-xl px-2 py-2 text-xs font-medium transition {presetButtonClass((params?.reasoning_effort ?? null) === null)}"
									on:click={() => setReasoningEffort(null)}
								>
									{$i18n.t('Default')}
								</button>
								<button
									type="button"
									class="rounded-xl px-2 py-2 text-xs font-medium transition {presetButtonClass(normalizedReasoningEffort === 'low')}"
									on:click={() => setReasoningEffort('low')}
								>
									{$i18n.t('Low')}
								</button>
								<button
									type="button"
									class="rounded-xl px-2 py-2 text-xs font-medium transition {presetButtonClass(normalizedReasoningEffort === 'medium')}"
									on:click={() => setReasoningEffort('medium')}
								>
									{$i18n.t('Medium')}
								</button>
								<button
									type="button"
									class="rounded-xl px-2 py-2 text-xs font-medium transition {presetButtonClass(normalizedReasoningEffort === 'high')}"
									on:click={() => setReasoningEffort('high')}
								>
									{$i18n.t('High')}
								</button>
							</div>

							<div class="text-[11px] text-gray-500 dark:text-gray-400">
								{$i18n.t('Provider support varies by model.')}
							</div>

							{#if reasoningEffortIsCustom}
								<div class="rounded-xl border border-dashed border-amber-200 dark:border-amber-900/60 px-3 py-2">
									<div class="text-[11px] font-medium text-amber-700 dark:text-amber-300">
										{$i18n.t('Custom')}
									</div>
									<div class="text-xs font-mono break-all">{params.reasoning_effort}</div>
									<div class="mt-1 text-[11px] text-gray-500 dark:text-gray-400">
										{$i18n.t('Selecting a preset will overwrite the custom value.')}
									</div>
								</div>
							{/if}
						</div>
					{/if}
				</div>
			</DropdownMenu.Content>
		</div>
	</Dropdown>
</div>
