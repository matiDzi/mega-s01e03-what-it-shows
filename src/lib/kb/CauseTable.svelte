<script lang="ts">
  import reports from '#lib/data/reports-4471.json';
  import notes from '#lib/data/desk-notes.json';
  import { groupByCause, ticketCause, uniqueById, type Cause, type Report } from '#lib/kb/causes';
  import { tenantOf } from '#lib/kb/locales';

  let { ticket }: { ticket: number } = $props();

  const merged = $derived(uniqueById(reports as Report[]).filter((r) => r.mergedInto === ticket));
  const reviewedCauses: Record<string, Cause> = {};
  for (const id of notes.calendarDayMismatch) reviewedCauses[id] = 'calendar-day-mismatch';
  for (const id of notes.inviteBeforeBooking) reviewedCauses[id] = 'invite-before-booking';
  const groups = $derived(groupByCause(merged, tenantOf, reviewedCauses));
  const owner = $derived(ticketCause(groups, ticket));
  const tags = (g: { taggedAs: Record<string, number> }) =>
    Object.entries(g.taggedAs)
      .sort((a, b) => b[1] - a[1])
      .map(([tag, n]) => `${tag} ×${n}`)
      .join(' · ');
</script>

<p class="sum">
  <strong>{merged.length}</strong> reports merged into #{ticket} ·
  <strong>{groups.length}</strong>
  {groups.length === 1 ? 'cause' : 'causes'}
</p>

<table>
  <thead>
    <tr><th>evidence group</th><th>reports</th><th>tagged as</th><th>tenants</th><th>seen</th><th>tracking</th></tr>
  </thead>
  <tbody>
    {#each groups as g (g.cause)}
      <tr class:wrong={g.cause !== owner}>
        <td><code>{g.cause}</code></td>
        <td>{g.reports.length}</td>
        <td class="tags">{tags(g)}</td>
        <td>{g.tenants}</td>
        <td>{g.firstSeen} → {g.lastSeen}</td>
        <td>{g.cause === owner ? `#${ticket}` : 'separate issue'}</td>
      </tr>
    {/each}
  </tbody>
</table>

<style>
  .sum { font-size: 1.1rem; }
  .tags { color: var(--muted); font-size: 0.85em; }
  .wrong td { background: var(--warn-bg); }
  .wrong td:last-child { color: var(--warn); font-weight: 600; }
</style>
