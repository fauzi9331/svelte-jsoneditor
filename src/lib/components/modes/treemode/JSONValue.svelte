<svelte:options immutable={true} />

<script lang="ts">
  import type { JSONEditorContext, JSONSelection, SearchResultItem } from '$lib/types.js'
  import type { JSONValue, JSONPath } from 'immutable-json-patch'
  import { isEditingSelection, isValueSelection } from '$lib/logic/selection.js'

  export let path: JSONPath
  export let value: JSONValue
  export let context: JSONEditorContext
  export let enforceString: boolean
  export let selection: JSONSelection | null
  export let searchResultItems: SearchResultItem[] | undefined

  $: isEditing = !context.readOnly && isValueSelection(selection) && isEditingSelection(selection)

  $: renderers = context.onRenderValue({
    path,
    value,
    readOnly: context.readOnly,
    enforceString,
    isEditing,
    parser: context.parser,
    normalization: context.normalization,
    selection,
    searchResultItems,
    onPatch: context.onPatch,
    onPasteJson: context.onPasteJson,
    onSelect: context.onSelect,
    onFind: context.onFind,
    findNextInside: context.findNextInside,
    focus: context.focus
  })
</script>

{#each renderers as renderer}
  <svelte:component this={renderer.component} {...renderer.props} />
{/each}
