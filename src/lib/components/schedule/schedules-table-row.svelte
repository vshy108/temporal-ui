<script lang="ts">
  import { page } from '$app/stores';

  import WorkflowStatus from '$lib/components/workflow-status.svelte';
  import Link from '$lib/holocene/link.svelte';
  import { translate } from '$lib/i18n/translate';
  import { relativeTime, timeFormat } from '$lib/stores/time-format';
  import { formatDate } from '$lib/utilities/format-date';
  import {
    routeForEventHistory,
    routeForSchedule,
  } from '$lib/utilities/route-for';

  import ScheduleBasicFrequency from './schedule-basic-frequency.svelte';

  import type { ScheduleActionResult, ScheduleListEntry } from '$types';

  let { namespace } = $page.params;

  export let schedule: ScheduleListEntry;

  $: spec = schedule?.info?.spec;
  $: calendar = spec?.structuredCalendar?.[0];
  $: interval = spec?.interval?.[0];
  $: timezoneName = spec?.timezoneName || 'UTC';

  const sortRecentActions = (recentActions: ScheduleActionResult[]) => {
    return (
      recentActions
        ?.sort(
          (a, b) =>
            new Date(b.actualTime as string).getTime() -
            new Date(a.actualTime as string).getTime(),
        )
        .slice(0, 5) ?? []
    );
  };

  $: route = routeForSchedule({
    namespace,
    scheduleId: schedule?.scheduleId as string,
  });
</script>

<tr>
  <td class="cell">
    <WorkflowStatus status={schedule?.info?.paused ? 'Paused' : 'Running'} />
  </td>
  <td class="cell whitespace-pre-line break-words">
    <Link href={route}>{schedule.scheduleId}</Link>
  </td>
  <td class="cell whitespace-pre-line break-words max-md:hidden">
    {schedule?.info?.workflowType?.name ?? ''}
  </td>
  <td class="cell truncate">
    {#each sortRecentActions(schedule?.info?.recentActions) as run}
      <p>
        <Link
          href={routeForEventHistory({
            namespace,
            workflow: run?.startWorkflowResult?.workflowId,
            run: run?.startWorkflowResult?.runId,
          })}
          >{formatDate(run?.actualTime, $timeFormat, {
            relative: $relativeTime,
          })}</Link
        >
      </p>
    {/each}
  </td>
  <td class="cell truncate">
    {#each schedule?.info?.futureActionTimes?.slice(0, 5) ?? [] as run}
      <div>
        {formatDate(run, $timeFormat, {
          relative: $relativeTime,
          relativeLabel: translate('common.from-now'),
        })}
      </div>
    {/each}
  </td>
  <td class="cell">
    <p>{@html translate('common.timezone', { timezone: timezoneName })}</p>
    <ScheduleBasicFrequency {calendar} {interval} />
  </td>
</tr>

<style lang="postcss">
  .cell {
    @apply p-2 text-left;
  }
</style>
