<script>
  import { onMount } from "svelte";
  import { fetchItem } from "@digitalculture/ochre-sdk";
  import { page } from "$app/stores";
  import { get } from "svelte/store";
  import { goto } from "$app/navigation";

  let item = null;
  let error = null;
  let loading = true;

  let uuid = get(page).params.uuid;

  async function fetchData() {
    loading = true;
    error = null;
    item = null;

    if (!uuid) {
      error = new Error("UUID Missing");
      loading = false;
      return;
    }

    try {
      const { error: fetchError, item: fetchedItem } = await fetchItem(uuid);
      if (fetchError) throw new Error(fetchError.message || "Failed to load data");
      if (!fetchedItem) throw new Error("Cannot find data");
      item = fetchedItem;
    } catch (err) {
      error = err;
    } finally {
      loading = false;
    }
  }

  onMount(() => {
    fetchData();
  });

  function getPropertiesObject(properties) {
    if (!properties) return {};
    return properties.reduce((obj, prop) => {
      let content = prop.values?.[0]?.content;

      if (typeof content === "object" && content !== null) {
        
        if ("label" in content) {
          content = content.label;
        } else if ("value" in content) {
          content = content.value;
        } else if ("url" in content) {
          content = content.url;
        } else {
          content = JSON.stringify(content);
        }
      }

      obj[prop.label] = content || "";
      return obj;
    }, {});
  }

  function getImageUrl(item) {
    if (!item) return null;
    if (item.image && item.image.url) return item.image.url;
    if (item.images && item.images.length > 0 && item.images[0].url) return item.images[0].url;
    if (item.representations && item.representations.length > 0 && item.representations[0].url) return item.representations[0].url;
    return null;
  }

  function goHome() {
    goto('/');
  }
</script>

<button
  on:click={goHome}
  class="px-4 py-2 bg-blue-600 text-white rounded hover:bg-blue-700"
>
  Home
</button>

{#if loading}
  <p class="text-center mt-8">Loading...</p>
{:else if error}
  <p class="text-center mt-8 text-red-600">: {error.message}</p>
{:else if item}
  <div class="max-w-4xl mx-auto p-6 bg-white rounded-lg shadow-md mt-6">
    <h1 class="text-3xl font-bold mb-4">{item.identification?.label || "No label"}</h1>

    {#if getImageUrl(item)}
      <img
        src={getImageUrl(item)}
        alt={item.image?.description || item.identification?.label || "Image"}
        class="w-full max-h-96 object-contain rounded mb-6"
      />
    {:else}
      <p class="text-gray-500 mb-6">[No image available]</p>
    {/if}

    <div class="grid grid-cols-1 md:grid-cols-2 gap-4">
      {#each Object.entries(getPropertiesObject(item.properties)) as [key, value]}
        <div class="p-2 border rounded">
          <span class="font-semibold">{key}:</span> {value}
        </div>
      {/each}
    </div>
  </div>
{/if}
