<script>
  import { MultiSelect } from "carbon-components-svelte";

  const items = [
    { id: "0", text: "Slack" },
    { id: "1", text: "Email" },
    { id: "2", text: "Fax" },
  ];

  let multiselect1_selectedIds = ["0"];
  let multiselect2_selectedIds = ["1", "2"];

  const formatSelected = (i) =>
    i.length === 0
      ? "N/A"
      : i.map((id) => items.find((item) => item.id === id).text).join(", ");

  $: primary = formatSelected(multiselect1_selectedIds);
  $: secondary = formatSelected(multiselect2_selectedIds);
</script>

<MultiSelect
  titleText="Primary contact"
  bind:selectedIds={multiselect1_selectedIds}
  label="Select contact methods..."
  {items}
/>

<div>Primary: {primary}</div>

<MultiSelect
  titleText="Secondary contact"
  bind:selectedIds={multiselect2_selectedIds}
  label="Select contact methods..."
  {items}
/>

<div>Secondary: {secondary}</div>

<style>
  div {
    margin: var(--cds-layout-01) 0 var(--cds-layout-03);
  }
</style>
