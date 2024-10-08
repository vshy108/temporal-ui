<script lang="ts">
  import { noop } from 'svelte/internal';

  import { goto } from '$app/navigation';
  import { page } from '$app/stores';

  import SchedulesCount from '$lib/components/schedule/schedules-count.svelte';
  import SchedulesTableRow from '$lib/components/schedule/schedules-table-row.svelte';
  import Button from '$lib/holocene/button.svelte';
  import EmptyState from '$lib/holocene/empty-state.svelte';
  import Input from '$lib/holocene/input/input.svelte';
  import Link from '$lib/holocene/link.svelte';
  import PaginatedTable from '$lib/holocene/table/paginated-table/api-paginated.svelte';
  import { translate } from '$lib/i18n/translate';
  import { fetchPaginatedSchedules } from '$lib/services/schedule-service';
  import { coreUserStore } from '$lib/stores/core-user';
  import { schedulesCount } from '$lib/stores/schedules';
  import type { ScheduleListEntry } from '$lib/types';
  import { routeForScheduleCreate } from '$lib/utilities/route-for';
  import { writeActionsAreAllowed } from '$lib/utilities/write-actions-are-allowed';

  $: namespace = $page.params.namespace;

  let coreUser = coreUserStore();
  $: createDisabled = $coreUser.namespaceWriteDisabled(namespace);

  $: onFetch = () => fetchPaginatedSchedules(namespace);

  let search = '';
  $: filteredSchedules = (schedules: ScheduleListEntry[]) =>
    search
      ? schedules.filter((schedule) =>
          schedule.scheduleId.toLowerCase().includes(search.toLowerCase()),
        )
      : schedules;
</script>

{#key namespace}
  <PaginatedTable
    let:visibleItems
    {onFetch}
    total={$schedulesCount}
    aria-label={translate('common.schedules')}
    pageSizeSelectLabel={translate('common.per-page')}
    nextButtonLabel={translate('common.next')}
    previousButtonLabel={translate('common.previous')}
    emptyStateMessage={translate('schedules.empty-state-title')}
    errorMessage={translate('schedules.error-message-fetching')}
  >
    <caption class="sr-only" slot="caption"
      >{translate('common.schedules')}</caption
    >

    <div class="flex flex-col gap-4" slot="header" let:visibleItems>
      <h1 class="flex flex-col gap-0 md:flex-row md:items-center md:gap-2">
        <SchedulesCount />
      </h1>

      <div class="flex flex-row flex-wrap justify-between gap-2">
        {#if visibleItems.length}
          <div class="w-96">
            <Input
              icon="search"
              type="search"
              label={translate('schedules.name')}
              labelHidden
              id="schedule-name-filter"
              placeholder={translate('schedules.name')}
              clearable
              clearButtonLabel={translate('common.clear-input-button-label')}
              bind:value={search}
              on:submit={noop}
            />
          </div>
        {/if}
        {#if !createDisabled && visibleItems.length}
          <Button
            data-testid="create-schedule"
            href={routeForScheduleCreate({ namespace })}
            disabled={!writeActionsAreAllowed()}
          >
            {translate('schedules.create')}
          </Button>
        {/if}
      </div>
    </div>

    <tr slot="headers" class="text-left">
      <th>{translate('common.status')}</th>
      <th>{translate('schedules.name')}</th>
      <th>{translate('common.workflow-type')}</th>
      <th>{translate('schedules.recent-runs')}</th>
      <th>{translate('schedules.upcoming-runs')}</th>
      <th>{translate('schedules.schedule-spec')}</th>
    </tr>
    {#each filteredSchedules(visibleItems) as schedule}
      <SchedulesTableRow {schedule} />
    {:else}
      {#if search}
        <tr>
          <td colspan="6">
            <EmptyState
              title={translate('schedules.empty-state-title')}
              content={translate('schedules.empty-state-description')}
            />
          </td>
        </tr>
      {/if}
    {/each}

    <svelte:fragment slot="empty">
      <EmptyState title={translate('schedules.empty-state-title')}>
        <p>
          {translate('schedules.getting-started-docs-link-preface')}
          <Link newTab href="https://docs.temporal.io/workflows/#schedule"
            >{translate('schedules.getting-started-docs-link')}</Link
          >
          {translate('schedules.getting-started-cli-link-preface')}
          <Link newTab href="https://docs.temporal.io/cli/schedule"
            >Temporal CLI</Link
          >.
        </p>
        {#if !createDisabled}
          <Button
            data-testid="create-schedule"
            on:click={() => goto(routeForScheduleCreate({ namespace }))}
            disabled={!writeActionsAreAllowed()}
          >
            {translate('schedules.create')}
          </Button>
        {/if}
      </EmptyState>
    </svelte:fragment>
  </PaginatedTable>
{/key}
