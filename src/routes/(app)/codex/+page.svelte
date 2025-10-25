<script lang="ts">
        import { onMount, onDestroy, getContext } from 'svelte';
        import type { SvelteComponent } from 'svelte';
        import { browser } from '$app/environment';
        import { goto } from '$app/navigation';

        import { config, models, settings } from '$lib/stores';

        import Modal from '$lib/components/common/Modal.svelte';
        import Badge from '$lib/components/common/Badge.svelte';
        import Tooltip from '$lib/components/common/Tooltip.svelte';
        import Sparkles from '$lib/components/icons/Sparkles.svelte';
        import CommandLine from '$lib/components/icons/CommandLine.svelte';
        import Users from '$lib/components/icons/Users.svelte';
        import Wrench from '$lib/components/icons/Wrench.svelte';
        import ModelSelector from '$lib/components/chat/ModelSelector.svelte';
        import { updateUserSettings } from '$lib/apis/users';

        const i18n = getContext('i18n');

        const ONBOARDING_KEY = 'codex_agents_onboarding_dismissed';

        type Capability = {
                icon: typeof SvelteComponent;
                title: string;
                description: string;
        };

        type QuickStartStep = {
                title: string;
                description: string;
        };

        type Integration = {
                name: string;
                badge: 'Beta' | 'Planned' | 'Research';
                description: string;
        };

        let showWelcomeModal = false;
        let showCreateAgentModal = false;
        let featureEnabled = false;
        let configUnsubscribe: (() => void) | undefined;
        let agentModels: string[] = [''];
        let agentModelsInitialised = false;
        let codingModelNames: string[] = [];
        let topModelSummary = '';

        const capabilityHighlights: Capability[] = [
                {
                        icon: CommandLine,
                        title: 'Instant code generation',
                        description: 'Draft endpoints, scripts, and tests with context-aware prompts.'
                },
                {
                        icon: Users,
                        title: 'Collaborative reviews',
                        description: 'Loop in teammates for approvals, annotations, and live iterations.'
                },
                {
                        icon: Wrench,
                        title: 'Workflow automation',
                        description: 'Connect repositories, CI, and deployment hooks for end-to-end agents.'
                }
        ];

        const quickStartSteps: QuickStartStep[] = [
                {
                        title: 'Pick a coding model',
                        description: 'Choose an AI model tuned for development and pair programming.'
                },
                {
                        title: 'Connect your codebase',
                        description: 'Sync repositories or upload archives to ground agent responses.'
                },
                {
                        title: 'Define objectives',
                        description: 'Outline tasks, guardrails, and review steps before running agents.'
                }
        ];

        const integrationRoadmap: Integration[] = [
                {
                        name: 'GitHub App',
                        badge: 'Beta',
                        description: 'Secure repository context sync and pull request reviews.'
                },
                {
                        name: 'Self-hosted runners',
                        badge: 'Planned',
                        description: 'Execute build pipelines and tests from your infrastructure.'
                },
                {
                        name: 'Live pair programming',
                        badge: 'Research',
                        description: 'Collaborate inside shared terminals and editors with your agents.'
                }
        ];

        const dismissOnboarding = () => {
                if (browser) {
                        localStorage.setItem(ONBOARDING_KEY, 'true');
                }
                showWelcomeModal = false;
        };

        const openCreateAgentModal = () => {
                showCreateAgentModal = true;
        };

        const closeCreateAgentModal = () => {
                showCreateAgentModal = false;
        };

        const startBlankAgent = async () => {
                const filteredModels = agentModels.filter((id) => id);

                if (filteredModels.length) {
                        const nextSettings = { ...$settings, models: filteredModels };
                        settings.set(nextSettings);

                        if (browser && localStorage.token) {
                                try {
                                        await updateUserSettings(localStorage.token, { ui: nextSettings });
                                } catch (error) {
                                        console.error('Failed to persist Codex agent model preferences', error);
                                }
                        }
                }

                closeCreateAgentModal();
                goto('/workspace');
        };

        onMount(() => {
                configUnsubscribe = config.subscribe((value) => {
                        featureEnabled = value?.features?.enable_codex_agents ?? false;

                        if (value && !featureEnabled) {
                                goto('/');
                                return;
                        }

                        if (browser && featureEnabled && !localStorage.getItem(ONBOARDING_KEY)) {
                                showWelcomeModal = true;
                        }
                });
        });

        onDestroy(() => {
                configUnsubscribe?.();
        });

        const CODING_KEYWORDS = ['code', 'coder', 'codellama', 'codex', 'deepseek', 'qwen'];

        $: if (!agentModelsInitialised && $settings?.models?.length) {
                agentModels = [...$settings.models];
                agentModelsInitialised = true;
        }

        $: if (!agentModels.length) {
                agentModels = [''];
        }

        $: if (agentModels.some((id) => id && !$models.find((model) => model.id === id))) {
                agentModels = agentModels.map((id) =>
                        id && !$models.find((model) => model.id === id) ? '' : id
                );
        }

        $: codingModelNames = $models
                .filter((model) => {
                        const id = model.id.toLowerCase();
                        const name = model.name.toLowerCase();
                        return CODING_KEYWORDS.some((keyword) => id.includes(keyword) || name.includes(keyword));
                })
                .slice(0, 5)
                .map((model) => model.name);

        $: if (!agentModelsInitialised && agentModels.some((id) => id)) {
                agentModelsInitialised = true;
        }

        $: topModelSummary = codingModelNames.length
                ? codingModelNames.join(', ')
                : $models.slice(0, 3).map((model) => model.name).join(', ');
</script>

<svelte:head>
        <title>{$i18n.t('Codex Agents')}</title>
</svelte:head>

{#if !featureEnabled}
        <div class="flex h-full min-h-[60vh] items-center justify-center">
                <div class="rounded-xl border border-gray-200 bg-white p-6 text-center text-sm text-gray-500 shadow-sm dark:border-gray-800 dark:bg-gray-900 dark:text-gray-300">
                        {$i18n.t('Codex Agents are disabled in this environment.')}
                </div>
        </div>
{:else}
        <div class="mx-auto flex w-full max-w-5xl flex-col gap-8 px-4 py-6 text-gray-900 dark:text-gray-100 md:px-6 md:py-10">
                <section class="rounded-2xl border border-gray-200 bg-white p-6 shadow-sm dark:border-gray-800 dark:bg-gray-900">
                        <div class="flex flex-col gap-4 md:flex-row md:items-start md:justify-between">
                                <div class="space-y-3">
                                        <div class="inline-flex items-center gap-2 rounded-full bg-purple-100 px-3 py-1 text-xs font-semibold uppercase tracking-wide text-purple-700 dark:bg-purple-500/10 dark:text-purple-300">
                                                <Sparkles class="size-4" />
                                                {$i18n.t('New workspace')}
                                        </div>
                                        <h1 class="text-2xl font-semibold md:text-3xl">
                                                {$i18n.t('Codex Agents')}
                                        </h1>
                                        <p class="max-w-2xl text-sm text-gray-600 dark:text-gray-300">
                                                {$i18n.t('Build AI coding assistants and automations with dedicated agent workflows designed for repositories, CI, and collaborative reviews.')}
                                        </p>
                                        <div class="flex flex-wrap gap-3 pt-1 text-sm">
                                                <button
                                                        class="flex items-center gap-2 rounded-full bg-gray-900 px-4 py-2 font-medium text-white transition hover:bg-gray-800 dark:bg-white dark:text-gray-900 dark:hover:bg-gray-100"
                                                        on:click={openCreateAgentModal}
                                                >
                                                        <CommandLine class="size-4" />
                                                        {$i18n.t('Create Agent')}
                                                </button>
                                                <a
                                                        class="flex items-center gap-2 rounded-full border border-gray-200 px-4 py-2 font-medium text-gray-700 transition hover:border-gray-300 hover:text-gray-900 dark:border-gray-700 dark:text-gray-200 dark:hover:border-gray-500"
                                                        href="https://github.com/open-webui"
                                                        rel="noreferrer"
                                                        target="_blank"
                                                >
                                                        <Wrench class="size-4" />
                                                        {$i18n.t('Open documentation')}
                                                </a>
                                        </div>
                                </div>
                                <div class="flex flex-col gap-2 rounded-xl border border-gray-100 bg-gray-50 p-4 text-sm text-gray-600 dark:border-gray-800 dark:bg-gray-950 dark:text-gray-300 md:w-64">
                                        <div class="font-semibold">{$i18n.t('Session insights')}</div>
                                        <div class="flex items-center justify-between">
                                                <span>{$i18n.t('Active agents')}</span>
                                                <Badge type="info" content={$i18n.t('Beta')} />
                                        </div>
                                        <div class="flex items-center justify-between">
                                                <span>{$i18n.t('Live sessions')}</span>
                                                <span class="font-mono text-sm text-gray-800 dark:text-gray-200">0</span>
                                        </div>
                                        <div class="flex items-center justify-between">
                                                <span>{$i18n.t('Repository connections')}</span>
                                                <span class="font-mono text-sm text-gray-800 dark:text-gray-200">0</span>
                                        </div>
                                        <div class="flex items-center justify-between">
                                                <span>{$i18n.t('Coding models available')}</span>
                                                <span class="font-mono text-sm text-gray-800 dark:text-gray-200">{$models.length}</span>
                                        </div>
                                        <p class="text-xs text-gray-500 dark:text-gray-400">
                                                {$i18n.t('Connect a repository or upload code to unlock real-time insights.')}
                                        </p>
                                </div>
                        </div>
                </section>

                <section class="grid gap-4 md:grid-cols-3">
                        {#each capabilityHighlights as capability}
                                <div class="flex flex-col gap-3 rounded-2xl border border-gray-200 bg-white p-4 shadow-sm transition hover:-translate-y-1 hover:shadow-lg dark:border-gray-800 dark:bg-gray-900">
                                        <div class="flex size-10 items-center justify-center rounded-xl bg-gray-100 text-gray-700 dark:bg-gray-800 dark:text-gray-200">
                                                <svelte:component this={capability.icon} class="size-5" />
                                        </div>
                                        <div>
                                                <h2 class="text-base font-semibold">{$i18n.t(capability.title)}</h2>
                                                <p class="pt-1 text-sm text-gray-600 dark:text-gray-300">
                                                        {$i18n.t(capability.description)}
                                                </p>
                                        </div>
                                </div>
                        {/each}
                </section>

                <section class="grid gap-4 md:grid-cols-[2fr,1fr]">
                        <div class="rounded-2xl border border-gray-200 bg-white p-6 shadow-sm dark:border-gray-800 dark:bg-gray-900">
                                <h2 class="text-lg font-semibold">{$i18n.t('Quick start')}</h2>
                                <p class="pb-4 text-sm text-gray-600 dark:text-gray-300">
                                        {$i18n.t('Follow these steps to launch your first Codex Agent session in minutes.')}
                                </p>
                                <ol class="space-y-4">
                                        {#each quickStartSteps as step, index}
                                                <li class="flex gap-3">
                                                        <span class="mt-0.5 flex size-8 items-center justify-center rounded-full bg-gray-900 text-sm font-semibold text-white dark:bg-white dark:text-gray-900">
                                                                {index + 1}
                                                        </span>
                                                        <div>
                                                                <div class="font-medium">{$i18n.t(step.title)}</div>
                                                                <p class="text-sm text-gray-600 dark:text-gray-300">
                                                                        {$i18n.t(step.description)}
                                                                        {#if index === 0 && $models.length}
                                                                                <span class="block pt-1 text-xs text-gray-500 dark:text-gray-400">
                                                                                        {$i18n.t('Available now: {{models}}', {
                                                                                                models: topModelSummary
                                                                                        })}
                                                                                </span>
                                                                        {/if}
                                                                </p>
                                                        </div>
                                                </li>
                                        {/each}
                                </ol>
                        </div>
                        <div class="rounded-2xl border border-dashed border-gray-300 bg-white p-6 shadow-sm dark:border-gray-700 dark:bg-gray-900">
                                <h2 class="text-lg font-semibold">{$i18n.t('Roadmap')}</h2>
                                <p class="pb-4 text-sm text-gray-600 dark:text-gray-300">
                                        {$i18n.t('Preview upcoming integrations designed for secure, offline-friendly deployments.')}
                                </p>
                                <div class="space-y-3">
                                        {#each integrationRoadmap as integration}
                                                <div class="rounded-xl border border-gray-200 bg-gray-50 p-3 dark:border-gray-800 dark:bg-gray-950">
                                                        <div class="flex items-center justify-between gap-2">
                                                                <div class="font-medium">{$i18n.t(integration.name)}</div>
                                                                <Badge type="info" content={$i18n.t(integration.badge)} />
                                                        </div>
                                                        <p class="pt-1 text-xs text-gray-600 dark:text-gray-400">
                                                                {$i18n.t(integration.description)}
                                                        </p>
                                                </div>
                                        {/each}
                                </div>
                        </div>
                </section>
        </div>
{/if}

{#if showWelcomeModal}
        <Modal size="sm" bind:show={showWelcomeModal}>
                <div class="space-y-4 px-5 py-4 text-gray-900 dark:text-gray-100">
                        <div class="flex items-center justify-between">
                                <h2 class="text-lg font-semibold">{$i18n.t('Welcome to Codex Agents')}</h2>
                                <button class="rounded-full p-1 text-gray-500 hover:bg-gray-100 dark:text-gray-400 dark:hover:bg-gray-800" on:click={dismissOnboarding}>
                                        <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 20 20" fill="currentColor" class="size-4">
                                                <path fill-rule="evenodd" d="M4.293 4.293a1 1 0 0 1 1.414 0L10 8.586l4.293-4.293a1 1 0 1 1 1.414 1.414L11.414 10l4.293 4.293a1 1 0 0 1-1.414 1.414L10 11.414l-4.293 4.293a1 1 0 0 1-1.414-1.414L8.586 10 4.293 5.707a1 1 0 0 1 0-1.414Z" clip-rule="evenodd" />
                                        </svg>
                                </button>
                        </div>
                        <p class="text-sm text-gray-600 dark:text-gray-300">
                                {$i18n.t('Use dedicated agent workspaces to generate, review, and ship code securely from your own infrastructure.')}
                        </p>
                        <div class="space-y-2 rounded-xl bg-gray-50 p-3 text-xs text-gray-600 dark:bg-gray-900 dark:text-gray-300">
                                <div class="flex items-center gap-2">
                                        <Badge type="success" content={$i18n.t('Offline first')} />
                                        <span>{$i18n.t('All prompts, artifacts, and credentials stay on your instance by default.')}</span>
                                </div>
                                <div class="flex items-center gap-2">
                                        <Badge type="info" content={$i18n.t('Workspace aware')} />
                                        <span>{$i18n.t('Agents inherit project permissions, so admins stay in control of access.')}</span>
                                </div>
                        </div>
                        <div class="flex justify-end gap-2 text-sm">
                                <button class="rounded-full px-4 py-2 text-gray-500 transition hover:bg-gray-100 dark:text-gray-300 dark:hover:bg-gray-800" on:click={dismissOnboarding}>
                                        {$i18n.t('Not now')}
                                </button>
                                <button class="rounded-full bg-gray-900 px-4 py-2 font-medium text-white transition hover:bg-gray-800 dark:bg-white dark:text-gray-900 dark:hover:bg-gray-100" on:click={() => {
                                                dismissOnboarding();
                                                showCreateAgentModal = true;
                                        }}>
                                        {$i18n.t('Start exploring')}
                                </button>
                        </div>
                </div>
        </Modal>
{/if}

{#if showCreateAgentModal}
        <Modal size="sm" bind:show={showCreateAgentModal}>
                <div class="space-y-4 px-5 py-4 text-gray-900 dark:text-gray-100">
                        <div class="space-y-1">
                                <h2 class="text-lg font-semibold">{$i18n.t('Create a Codex Agent')}</h2>
                                <p class="text-sm text-gray-600 dark:text-gray-300">
                                        {$i18n.t('Choose a starting point. You can customise repositories, tools, and review flows after creation.')}
                                </p>
                        </div>
                        <div class="space-y-3">
                                <button class="w-full rounded-2xl border border-gray-200 p-3 text-left transition hover:border-gray-300 hover:bg-gray-50 dark:border-gray-800 dark:hover:border-gray-600 dark:hover:bg-gray-900" on:click={() => {
                                                startBlankAgent();
                                        }}>
                                        <div class="flex items-center justify-between">
                                                <div>
                                                        <div class="font-medium">{$i18n.t('Blank agent')}</div>
                                                        <p class="text-xs text-gray-600 dark:text-gray-400">
                                                                {$i18n.t('Start from scratch with manual tool and repository configuration.')}
                                                        </p>
                                                </div>
                                                <Badge type="info" content={$i18n.t('Recommended')} />
                                        </div>
                                </button>
                                <button class="w-full rounded-2xl border border-dashed border-gray-300 p-3 text-left text-gray-500 transition hover:border-gray-400 hover:bg-gray-50 dark:border-gray-700 dark:text-gray-400 dark:hover:border-gray-500 dark:hover:bg-gray-900" type="button">
                                        <div class="flex items-center justify-between">
                                                <div>
                                                        <div class="font-medium">{$i18n.t('Import workflow')}</div>
                                                        <p class="text-xs text-gray-500 dark:text-gray-400">
                                                                {$i18n.t('Upload a JSON export to recreate an existing agent setup. Coming soon!')}
                                                        </p>
                                                </div>
                                                <Tooltip content={$i18n.t('Available soon')}>
                                                        <Badge type="muted" content={$i18n.t('Upcoming')} />
                                                </Tooltip>
                                        </div>
                                </button>
                                <div class="space-y-3 rounded-2xl border border-gray-100 bg-gray-50 p-3 dark:border-gray-800 dark:bg-gray-950">
                                        <div class="flex items-center justify-between text-xs font-semibold uppercase tracking-wide text-gray-600 dark:text-gray-300">
                                                <span>{$i18n.t('Coding models')}</span>
                                                <span class="rounded-full bg-gray-900 px-2 py-0.5 text-[0.625rem] font-medium text-white dark:bg-white dark:text-gray-900">
                                                        {$models.length}
                                                </span>
                                        </div>
                                        {#if $models.length}
                                                <div class="space-y-2">
                                                        <ModelSelector bind:selectedModels={agentModels} showSetDefault={false} />
                                                        {#if topModelSummary}
                                                                <p class="text-xs text-gray-500 dark:text-gray-400">
                                                                        {$i18n.t('Popular choices include {{models}}.', {
                                                                                models: topModelSummary
                                                                        })}
                                                                </p>
                                                        {/if}
                                                </div>
                                        {:else}
                                                <p class="text-xs text-gray-500 dark:text-gray-400">
                                                        {$i18n.t('No models detected yet. Visit Settings to connect providers before creating an agent.')}
                                                </p>
                                        {/if}
                                </div>
                        </div>
                        <div class="flex justify-end">
                                <button class="rounded-full px-4 py-2 text-sm text-gray-500 transition hover:bg-gray-100 dark:text-gray-300 dark:hover:bg-gray-800" on:click={closeCreateAgentModal}>
                                        {$i18n.t('Close')}
                                </button>
                        </div>
                </div>
        </Modal>
{/if}
