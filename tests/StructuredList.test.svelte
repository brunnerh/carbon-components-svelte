<script lang="ts">
  import {
    StructuredList,
    StructuredListSkeleton,
    StructuredListBody,
    StructuredListHead,
    StructuredListCell,
    StructuredListRow,
    StructuredListInput,
  } from "carbon-components-svelte";
  import CheckmarkFilled from "carbon-icons-svelte/lib/CheckmarkFilled.svelte";
</script>

<StructuredList
  selection
  selected="row-1-value"
  on:change={(e) => {
    console.log(e.detail); // string
  }}
>
  <StructuredListHead>
    <StructuredListRow head>
      <StructuredListCell head>Column A</StructuredListCell>
      <StructuredListCell head>Column B</StructuredListCell>
      <StructuredListCell head>Column C</StructuredListCell>
    </StructuredListRow>
  </StructuredListHead>
  <StructuredListBody>
    <StructuredListRow>
      <StructuredListCell noWrap>Row 1</StructuredListCell>
      <StructuredListCell>Row 1</StructuredListCell>
      <StructuredListCell>
        Lorem ipsum dolor sit amet, consectetur adipiscing elit. Nunc dui magna,
        finibus id tortor sed, aliquet bibendum augue. Aenean posuere sem vel
        euismod dignissim. Nulla ut cursus dolor. Pellentesque vulputate nisl a
        porttitor interdum.
      </StructuredListCell>
    </StructuredListRow>
    <StructuredListRow>
      <StructuredListCell noWrap>Row 2</StructuredListCell>
      <StructuredListCell>Row 2</StructuredListCell>
      <StructuredListCell>
        Lorem ipsum dolor sit amet, consectetur adipiscing elit. Nunc dui magna,
        finibus id tortor sed, aliquet bibendum augue. Aenean posuere sem vel
        euismod dignissim. Nulla ut cursus dolor. Pellentesque vulputate nisl a
        porttitor interdum.
      </StructuredListCell>
    </StructuredListRow>
    <StructuredListRow>
      <StructuredListCell noWrap>Row 3</StructuredListCell>
      <StructuredListCell>Row 3</StructuredListCell>
      <StructuredListCell>
        Lorem ipsum dolor sit amet, consectetur adipiscing elit. Nunc dui magna,
        finibus id tortor sed, aliquet bibendum augue. Aenean posuere sem vel
        euismod dignissim. Nulla ut cursus dolor. Pellentesque vulputate nisl a
        porttitor interdum.
      </StructuredListCell>
    </StructuredListRow>
  </StructuredListBody>
</StructuredList>

<StructuredList selection selected="row-1-value">
  <StructuredListHead>
    <StructuredListRow head>
      <StructuredListCell head>ColumnA</StructuredListCell>
      <StructuredListCell head>ColumnB</StructuredListCell>
      <StructuredListCell head>ColumnC</StructuredListCell>
      <StructuredListCell head>{""}</StructuredListCell>
    </StructuredListRow>
  </StructuredListHead>
  <StructuredListBody>
    {#each [1, 2, 3] as item}
      <StructuredListRow label for="row-{item}">
        <StructuredListCell>Row {item}</StructuredListCell>
        <StructuredListCell>Row {item}</StructuredListCell>
        <StructuredListCell>
          Lorem ipsum dolor sit amet, consectetur adipiscing elit. Nunc dui
          magna, finibus id tortor sed, aliquet bibendum augue. Aenean posuere
          sem vel euismod dignissim. Nulla ut cursus dolor. Pellentesque
          vulputate nisl a porttitor interdum.
        </StructuredListCell>
        <StructuredListInput
          id="row-{item}"
          value="row-{item}-value"
          title="row-{item}-title"
          name="row-{item}-name"
        />
        <StructuredListCell>
          <CheckmarkFilled
            class="bx--structured-list-svg"
            aria-label="select an option"
            title="select an option"
          />
        </StructuredListCell>
      </StructuredListRow>
    {/each}
  </StructuredListBody>
</StructuredList>

<StructuredListSkeleton rows={3} />
