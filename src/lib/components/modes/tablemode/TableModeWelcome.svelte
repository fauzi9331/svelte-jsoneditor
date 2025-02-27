<svelte:options immutable={true} />

<script lang="ts">
  import type { JSONPath, JSONValue } from 'immutable-json-patch'
  import { getIn, isJSONArray, isJSONObject } from 'immutable-json-patch'
  import type { JSONParser, OnChangeMode } from '$lib/types.js'
  import { Mode } from '$lib/types.js'
  import { valueType } from '$lib/utils/typeUtils.js'
  import { findNestedArrays } from '$lib/logic/table.js'
  import { isEmpty } from 'lodash-es'
  import { stringifyJSONPath } from '$lib/utils/pathUtils.js'

  export let text: string | undefined
  export let json: JSONValue | undefined
  export let readOnly: boolean
  export let parser: JSONParser
  export let openJSONEditorModal: (path: JSONPath) => void
  export let onChangeMode: OnChangeMode

  $: action = readOnly ? 'View' : 'Edit'

  let nestedArrayPaths: JSONPath[]
  $: nestedArrayPaths = json
    ? findNestedArrays(json)
        .slice(0, 99)
        .filter((path) => path.length > 0)
    : []
  $: hasNestedArrays = !isEmpty(nestedArrayPaths)

  $: documentType = hasNestedArrays
    ? 'Object with nested arrays'
    : json === undefined && (text === '' || text === undefined)
    ? 'An empty document'
    : isJSONObject(json)
    ? 'An object'
    : isJSONArray(json)
    ? 'An empty array' // note: can also be an array with objects but without properties
    : `A ${valueType(json, parser)}`
</script>

<div class="jse-table-mode-welcome">
  <div class="jse-space jse-before" />

  <div class="jse-nested-arrays">
    <div class="jse-nested-arrays-title">{documentType}</div>
    <div class="jse-nested-arrays-info">
      {#if hasNestedArrays}
        An object cannot be opened in table mode. You can open a nested array instead, or open the
        document in tree mode.
      {:else}
        {documentType} cannot be opened in table mode. You can open the document in tree mode instead.
      {/if}
    </div>
    {#each nestedArrayPaths as nestedArrayPath}
      {@const count = getIn(json, nestedArrayPath).length}

      <button
        type="button"
        class="jse-nested-array-action"
        on:click={() => openJSONEditorModal(nestedArrayPath)}
      >
        {action} "{stringifyJSONPath(nestedArrayPath)}"
        <span class="jse-nested-array-count">({count} {count !== 1 ? 'items' : 'item'})</span>
      </button>
    {/each}
    <button type="button" class="jse-nested-array-action" on:click={() => onChangeMode(Mode.tree)}>
      {action} in tree mode
    </button>
  </div>

  <div class="jse-space jse-after" />
</div>

<style src="./TableModeWelcome.scss"></style>
