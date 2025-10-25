<script lang="ts">
	import { toast } from 'svelte-sonner';
	import dayjs from 'dayjs';
	import { createEventDispatcher } from 'svelte';
	import { onMount, getContext } from 'svelte';

        import { updateUserById, getUserPermissionsById, getUserGroupsById } from '$lib/apis/users';

        import { ROLE_DEFINITIONS, ROLE_ORDER } from '$lib/constants';
        import Badge from '$lib/components/common/Badge.svelte';

	import Modal from '$lib/components/common/Modal.svelte';
	import localizedFormat from 'dayjs/plugin/localizedFormat';

	const i18n = getContext('i18n');
	const dispatch = createEventDispatcher();
	dayjs.extend(localizedFormat);

        export let show = false;
        export let selectedUser;
        export let sessionUser;
        export let initialTab: 'profile' | 'access' = 'profile';

        type RoleKey = (typeof ROLE_ORDER)[number];
        type PermissionCategory = Record<string, boolean>;
        type UserPermissionSummary = {
                workspace: PermissionCategory;
                sharing: PermissionCategory;
                chat: PermissionCategory;
                features: PermissionCategory;
        };

        type GroupSummary = {
                id: string;
                name: string;
                description?: string;
                user_ids?: string[];
        };

        const roleOptions = ROLE_ORDER.map((role) => ({
                value: role,
                label: ROLE_DEFINITIONS[role].label,
                description: ROLE_DEFINITIONS[role].description,
                badge: ROLE_DEFINITIONS[role].badge
        }));

        const formatPermissionKey = (key: string) =>
                key
                        .replace(/_/g, ' ')
                        .replace(/\b\w/g, (letter) => letter.toUpperCase());

        let activeTab: 'profile' | 'access' = 'profile';
        let permissions: UserPermissionSummary | null = null;
        let permissionsLoading = false;
        let groups: GroupSummary[] = [];
        let accessLoaded = false;
        let accessError: string | null = null;

        let _user = {
                id: '',
                profile_image_url: '',
                role: 'pending',
                name: '',
                email: '',
                password: ''
        };

        $: selectedRoleMeta =
                ROLE_DEFINITIONS[_user.role as RoleKey] ?? ROLE_DEFINITIONS.pending;

        const submitHandler = async () => {
                const res = await updateUserById(localStorage.token, selectedUser.id, _user).catch((error) => {
                        toast.error(`${error}`);
                });

                if (res) {
                        dispatch('save');
                        show = false;
                }
        };

        const resetAccessState = () => {
                permissions = null;
                groups = [];
                accessLoaded = false;
                accessError = null;
        };

        const loadAccessData = async () => {
                if (!selectedUser) {
                        return;
                }

                permissionsLoading = true;
                accessError = null;

                try {
                        const [userPermissions, userGroups] = await Promise.all([
                                getUserPermissionsById(localStorage.token, selectedUser.id),
                                getUserGroupsById(localStorage.token, selectedUser.id)
                        ]);

                        permissions = userPermissions;
                        groups = userGroups ?? [];
                        accessLoaded = true;
                } catch (error) {
                        console.error(error);
                        accessError = `${error}`;
                } finally {
                        permissionsLoading = false;
                }
        };

        const refreshAccessData = async () => {
                resetAccessState();
                await loadAccessData();
        };

        onMount(() => {
                if (selectedUser) {
                        _user = {
                                ...selectedUser,
                                id: selectedUser.id,
                                password: ''
                        };
                }
        });

        let previousShow = false;

        $: {
                if (show && !previousShow) {
                        activeTab = initialTab;
                }
                previousShow = show;
        }

        $: if (!show) {
                activeTab = 'profile';
                resetAccessState();
        }

        $: if (show && selectedUser && _user.id !== selectedUser.id) {
                _user = {
                        ...selectedUser,
                        id: selectedUser.id,
                        password: ''
                };
                resetAccessState();
        }

        $: if (show && activeTab === 'access' && selectedUser && !permissionsLoading && !accessLoaded) {
                loadAccessData();
        }
</script>

<Modal size="sm" bind:show>
	<div>
		<div class=" flex justify-between dark:text-gray-300 px-5 pt-4 pb-2">
			<div class=" text-lg font-medium self-center">{$i18n.t('Edit User')}</div>
			<button
				class="self-center"
				on:click={() => {
					show = false;
				}}
			>
				<svg
					xmlns="http://www.w3.org/2000/svg"
					viewBox="0 0 20 20"
					fill="currentColor"
					class="w-5 h-5"
				>
					<path
						d="M6.28 5.22a.75.75 0 00-1.06 1.06L8.94 10l-3.72 3.72a.75.75 0 101.06 1.06L10 11.06l3.72 3.72a.75.75 0 101.06-1.06L11.06 10l3.72-3.72a.75.75 0 00-1.06-1.06L10 8.94 6.28 5.22z"
					/>
				</svg>
			</button>
		</div>

		<div class="flex flex-col md:flex-row w-full md:space-x-4 dark:text-gray-200">
			<div class=" flex flex-col w-full sm:flex-row sm:justify-center sm:space-x-6">
                                <form
                                        class="flex flex-col w-full"
                                        on:submit|preventDefault={() => {
                                                if (activeTab === 'profile') {
                                                        submitHandler();
                                                }
                                        }}
                                >
					<div class=" flex items-center rounded-md px-5 py-2 w-full">
						<div class=" self-center mr-5">
							<img
								src={selectedUser.profile_image_url}
								class=" max-w-[55px] object-cover rounded-full"
								alt="User profile"
							/>
						</div>

						<div>
							<div class=" self-center capitalize font-semibold">{selectedUser.name}</div>

							<div class="text-xs text-gray-500">
								{$i18n.t('Created at')}
								{dayjs(selectedUser.created_at * 1000).format('LL')}
							</div>
						</div>
					</div>

                                        <div class=" px-5 pt-3 pb-5">
                                                <div class="flex items-center justify-between mb-3">
                                                        <div class="inline-flex rounded-full bg-gray-100 dark:bg-gray-850 p-0.5 text-xs font-medium">
                                                                <button
                                                                        type="button"
                                                                        class="px-3 py-1 rounded-full transition {activeTab === 'profile'
                                                                                ? 'bg-white dark:bg-gray-900 text-gray-900 dark:text-white shadow-sm'
                                                                                : 'text-gray-500 hover:text-gray-800 dark:text-gray-400 dark:hover:text-gray-200'}"
                                                                        on:click={() => {
                                                                                activeTab = 'profile';
                                                                        }}
                                                                >
                                                                        {$i18n.t('Profile')}
                                                                </button>
                                                                <button
                                                                        type="button"
                                                                        class="px-3 py-1 rounded-full transition {activeTab === 'access'
                                                                                ? 'bg-white dark:bg-gray-900 text-gray-900 dark:text-white shadow-sm'
                                                                                : 'text-gray-500 hover:text-gray-800 dark:text-gray-400 dark:hover:text-gray-200'}"
                                                                        on:click={() => {
                                                                                activeTab = 'access';
                                                                        }}
                                                                >
                                                                        {$i18n.t('Access')}
                                                                </button>
                                                        </div>

                                                        {#if activeTab === 'access'}
                                                                <button
                                                                        type="button"
                                                                        class="flex items-center gap-1 rounded-full border border-gray-200 dark:border-gray-800 px-3 py-1 text-xs text-gray-600 hover:bg-gray-100 dark:text-gray-300 dark:hover:bg-gray-900 transition"
                                                                        on:click={refreshAccessData}
                                                                        disabled={permissionsLoading}
                                                                >
                                                                        <svg
                                                                                xmlns="http://www.w3.org/2000/svg"
                                                                                viewBox="0 0 20 20"
                                                                                fill="currentColor"
                                                                                class="size-3.5"
                                                                        >
                                                                                <path
                                                                                        fill-rule="evenodd"
                                                                                        d="M2.804 10.596a.75.75 0 0 1-.096-1.058A7 7 0 0 1 17.5 10a.75.75 0 1 1-1.5 0 5.5 5.5 0 1 0-9.383 3.893l1.25-1.25a.75.75 0 0 1 1.133.976l-2.5 2.75a.75.75 0 0 1-1.105.028l-2.75-2.75a.75.75 0 0 1 1.06-1.06l1.15 1.15a7.002 7.002 0 0 1-1.051-3.091Z"
                                                                                        clip-rule="evenodd"
                                                                                />
                                                                        </svg>
                                                                        {$i18n.t('Refresh')}
                                                                </button>
                                                        {/if}
                                                </div>

                                                {#if activeTab === 'profile'}
                                                        <div class="flex flex-col space-y-3">
                                                                <div class="flex flex-col w-full">
                                                                        <div class=" mb-1 text-xs text-gray-500">{$i18n.t('Role')}</div>

                                                                        <div class="flex-1">
                                                                                <select
                                                                                        class="w-full dark:bg-gray-900 rounded-sm text-sm bg-transparent disabled:text-gray-500 dark:disabled:text-gray-500 outline-hidden"
                                                                                        bind:value={_user.role}
                                                                                        disabled={_user.id == sessionUser.id}
                                                                                        required
                                                                                >
                                                                                        {#each roleOptions as roleOption}
                                                                                                <option value={roleOption.value}>
                                                                                                        {$i18n.t(roleOption.label)}
                                                                                                </option>
                                                                                        {/each}
                                                                                </select>
                                                                        </div>
                                                                        <div class="mt-2 flex items-start gap-2">
                                                                                <Badge type={selectedRoleMeta.badge} content={$i18n.t(selectedRoleMeta.label)} />
                                                                                <p class="text-xs text-gray-500 dark:text-gray-400 leading-snug">
                                                                                        {$i18n.t(selectedRoleMeta.description)}
                                                                                </p>
                                                                        </div>
                                                                </div>

                                                                <div class="flex flex-col w-full">
                                                                        <div class=" mb-1 text-xs text-gray-500">{$i18n.t('Email')}</div>

                                                                        <div class="flex-1">
                                                                                <input
                                                                                        class="w-full rounded-sm text-sm bg-transparent disabled:text-gray-500 dark:disabled:text-gray-500 outline-hidden"
                                                                                        type="email"
                                                                                        bind:value={_user.email}
                                                                                        placeholder={$i18n.t('Enter Your Email')}
                                                                                        autocomplete="off"
                                                                                        required
                                                                                        disabled={_user.id == sessionUser.id}
                                                                                />
                                                                        </div>
                                                                </div>

                                                                <div class="flex flex-col w-full">
                                                                        <div class=" mb-1 text-xs text-gray-500">{$i18n.t('Name')}</div>

                                                                        <div class="flex-1">
                                                                                <input
                                                                                        class="w-full rounded-sm text-sm bg-transparent outline-hidden"
                                                                                        type="text"
                                                                                        bind:value={_user.name}
                                                                                        placeholder={$i18n.t('Enter Your Name')}
                                                                                        autocomplete="off"
                                                                                        required
                                                                                />
                                                                        </div>
                                                                </div>

                                                                <div class="flex flex-col w-full">
                                                                        <div class=" mb-1 text-xs text-gray-500">{$i18n.t('New Password')}</div>

                                                                        <div class="flex-1">
                                                                                <input
                                                                                        class="w-full rounded-sm text-sm bg-transparent outline-hidden"
                                                                                        type="password"
                                                                                        placeholder={$i18n.t('Enter New Password')}
                                                                                        bind:value={_user.password}
                                                                                        autocomplete="new-password"
                                                                                />
                                                                        </div>
                                                                </div>
                                                        </div>

                                                        <div class="flex justify-end pt-3 text-sm font-medium">
                                                                <button
                                                                        class="px-3.5 py-1.5 text-sm font-medium bg-black hover:bg-gray-900 text-white dark:bg-white dark:text-black dark:hover:bg-gray-100 transition rounded-full flex flex-row space-x-1 items-center"
                                                                        type="submit"
                                                                >
                                                                        {$i18n.t('Save')}
                                                                </button>
                                                        </div>
                                                {:else}
                                                        <div class="space-y-3">
                                                                {#if permissionsLoading}
                                                                        <div class="flex justify-center py-6">
                                                                                <Spinner />
                                                                        </div>
                                                                {:else if accessError}
                                                                        <div class="rounded-lg border border-red-200 dark:border-red-500/40 bg-red-500/10 px-3 py-2 text-xs text-red-600 dark:text-red-300">
                                                                                {accessError}
                                                                        </div>
                                                                {:else if permissions}
                                                                        {#each Object.entries(permissions) as [section, values] (section)}
                                                                                <div class="rounded-lg border border-gray-100 dark:border-gray-800 bg-gray-50/40 dark:bg-gray-900/40 p-3">
                                                                                        <div class="flex items-start justify-between gap-2 mb-2">
                                                                                                <div>
                                                                                                        <div class="text-sm font-semibold capitalize">{$i18n.t(section)}</div>
                                                                                                        <div class="text-[11px] text-gray-500 dark:text-gray-400">
                                                                                                                {$i18n.t('Effective permissions for this category.')}
                                                                                                        </div>
                                                                                                </div>
                                                                                                <Badge
                                                                                                        type={Object.values(values).some(Boolean) ? 'success' : 'muted'}
                                                                                                        content={Object.values(values).some(Boolean) ? $i18n.t('Active') : $i18n.t('Restricted')}
                                                                                                />
                                                                                        </div>
                                                                                        <div class="grid grid-cols-1 gap-2 sm:grid-cols-2">
                                                                                                {#each Object.entries(values) as [key, value] (key)}
                                                                                                        <div class="flex items-center justify-between rounded-md border border-gray-100 dark:border-gray-800 px-2 py-1 text-xs">
                                                                                                                <span class="font-medium text-gray-700 dark:text-gray-200">{formatPermissionKey(key)}</span>
                                                                                                                <span class={value ? 'text-emerald-500 font-semibold' : 'text-gray-500 dark:text-gray-500'}>
                                                                                                                        {$i18n.t(value ? 'Allowed' : 'Disabled')}
                                                                                                                </span>
                                                                                                        </div>
                                                                                                {/each}
                                                                                        </div>
                                                                                </div>
                                                                        {/each}

                                                                        <div class="rounded-lg border border-gray-100 dark:border-gray-800 bg-gray-50/40 dark:bg-gray-900/40 p-3">
                                                                                <div class="flex items-center justify-between mb-2">
                                                                                        <div class="text-sm font-semibold">{$i18n.t('Groups')}</div>
                                                                                        <div class="text-xs text-gray-500 dark:text-gray-400">{groups.length}</div>
                                                                                </div>
                                                                                {#if groups.length > 0}
                                                                                        <div class="flex flex-wrap gap-2">
                                                                                                {#each groups as group (group.id)}
                                                                                                        <div class="rounded-full border border-gray-200 dark:border-gray-800 bg-white dark:bg-gray-950 px-3 py-1">
                                                                                                                <div class="text-xs font-medium text-gray-700 dark:text-gray-200">{group.name}</div>
                                                                                                                {#if group.description}
                                                                                                                        <div class="text-[11px] text-gray-500 dark:text-gray-400">{group.description}</div>
                                                                                                                {/if}
                                                                                                        </div>
                                                                                                {/each}
                                                                                        </div>
                                                                                {:else}
                                                                                        <div class="text-xs text-gray-500 dark:text-gray-400">{$i18n.t('No groups assigned')}</div>
                                                                                {/if}
                                                                        </div>
                                                                {:else}
                                                                        <div class="text-xs text-gray-500 dark:text-gray-400">{$i18n.t('Permissions are not available for this user yet.')}</div>
                                                                {/if}
                                                        </div>
                                                {/if}
                                        </div>
                                </form>
			</div>
		</div>
	</div>
</Modal>

<style>
	input::-webkit-outer-spin-button,
	input::-webkit-inner-spin-button {
		/* display: none; <- Crashes Chrome on hover */
		-webkit-appearance: none;
		margin: 0; /* <-- Apparently some margin are still there even though it's hidden */
	}

	.tabs::-webkit-scrollbar {
		display: none; /* for Chrome, Safari and Opera */
	}

	.tabs {
		-ms-overflow-style: none; /* IE and Edge */
		scrollbar-width: none; /* Firefox */
	}

	input[type='number'] {
		-moz-appearance: textfield; /* Firefox */
	}
</style>
