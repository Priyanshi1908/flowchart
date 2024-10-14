<script>
  export let subcategory;

  import RecursiveCategory from "./RecursiveCategory.svelte";

  function hasNestedSubcategories(subcategory) {
    return (
      subcategory.subcategories &&
      Array.isArray(subcategory.subcategories) &&
      subcategory.subcategories.length > 0
    );
  }

  function isObject(item) {
    return typeof item === "object" && item !== null && "subcategory" in item;
  }
</script>

<ul class="list-disc list-inside ml-4">
  <li class="text-white/70">
    <strong>{subcategory.subcategory}</strong>

    {#if hasNestedSubcategories(subcategory)}
      <ul class="list-disc list-inside ml-4">
        {#each subcategory.subcategories as item}
          {#if isObject(item)}
            <RecursiveCategory subcategory={item} />
          {:else}
            <li class="text-white/70">{item}</li>
          {/if}
        {/each}
      </ul>
    {/if}
  </li>
</ul>
