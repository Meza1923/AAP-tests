# Test inventory for AAP variable precedence

Push this whole repo (or just the `inventory/` folder, adjusting the
Inventory Source's "Inventory file" path accordingly) to your git repo,
then point an AAP Inventory Source ("Sourced from a Project") at
`inventory/hosts.yml`.

Make sure "Overwrite" and "Overwrite variables" are enabled on the source
before syncing, otherwise stale data from previous syncs can mask real
precedence bugs.

## Expected result after sync

| Host    | environment | gpu_count | rack_id | maintenance_window | power_zone |
|---------|-------------|-----------|---------|---------------------|------------|
| node001 | lab         | 4         | rack01  | sunday-02:00        | A          |
| node002 | lab         | 0         | rack01  | monday-04:00        | A          |
| node003 | lab         | 4         | (unset) | sunday-02:00        | (unset)    |

What each row proves:
- node001: group_vars merge correctly across two groups it belongs to
  (rack01 + gpu_nodes), gpu_count comes from gpu_nodes.yml (4), not
  all.yml (0).
- node002: host_vars override group_vars — maintenance_window is
  monday-04:00 from host_vars, not sunday-02:00 from all.yml.
- node003: belongs to rack02 (no group_vars file for rack02) and
  gpu_nodes, so it picks up gpu_count: 4 but has no rack_id/power_zone.

Verify via UI: Inventories -> your inventory -> Hosts -> click a host ->
Details tab (shows merged variables).

Verify via API (more reliable, shows exact merged result used at job-run
time):

    curl -k -H "Authorization: Bearer $AAP_TOKEN" \
      "https://<aap-host>/api/controller/v2/hosts/?name=node001.lab.local"
