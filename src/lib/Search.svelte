<script lang="ts">
  interface Props {
    search: string;
  }

  let { search = $bindable("") }: Props = $props();

  let searchbox = $state<HTMLSpanElement>();
</script>

<!-- svelte-ignore a11y_click_events_have_key_events -->
<!-- svelte-ignore a11y_no_static_element_interactions -->
<div
  class="w-full mt-3 mb-2 text-black flex items-center border-b border-black"
  onclick={() => searchbox?.focus({ focusVisible: true })}
>
  <!-- svelte-ignore a11y_autofocus -->
  <span
    contenteditable="true"
    class="whitespace-nowrap ring-0 outline-0 caret-transparent peer relative after:content-['search'] focus-within:after:hidden after:text-gray-500 {search !==
    ''
      ? 'after:hidden'
      : ''}"
    bind:this={searchbox}
    autofocus
    onmousedown={(e) => {
      e.preventDefault();
    }}
    onkeydown={async (e) => {
      if (e.metaKey || e.ctrlKey || e.shiftKey || e.code === "Enter" || e.code === "Tab")
        return e.preventDefault();
      if (searchbox) {
        const range = document.createRange();
        range.selectNodeContents(searchbox);
        range.collapse(false);
        const selection = window.getSelection();
        selection?.removeAllRanges();
        selection?.addRange(range);
      }
    }}
    oninput={() => {
      search = searchbox?.textContent ?? "";
    }}
    onblur={() => {
      searchbox?.focus();
      if (searchbox?.textContent.trim() === "") searchbox!.innerHTML = "";
    }}
  >
  </span>
  <div
    class="inline-block peer-focus-within:blink peer-focus-within:bg-black h-4 w-2"
  ></div>
</div>
