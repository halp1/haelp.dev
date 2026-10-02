<script lang="ts">
  import Search from "#lib/Search.svelte";
  import { onMount, tick } from "svelte";

  let search = $state("");

  interface Section {
    name: string;
    href?: string;
    copy?: string;
    body?: string;
    subsections?: Section[];
  }

  type IndexedSection = Section & { index: number; subsections?: IndexedSection[] };

  const sections: Section[] = [
    {
      name: "about",
      body: "a casual developer of all types of software",
    },
    {
      name: "presence",
      subsections: [
        {
          name: "github - halp1",
          href: "https://github.com/halp1",
        },
        {
          name: "discord - @haelp",
          copy: "@haelp",
        },
      ],
    },
    {
      name: "projects",
      subsections: [
        {
          name: "triangle.js",
          body: "tetr.io game api interface",
          subsections: [
            { name: "github: halp1/triangle", href: "https://github.com/halp1/triangle" },
            {
              name: "npm: @haelp/teto",
              href: "https://www.npmjs.com/package/@haelp/teto",
            },
            { name: "docs", href: "https://triangle.haelp.dev" },
          ],
        },
        {
          name: "mochbot",
          body: "plays tetr.io better than you",
          subsections: [
            {
              name: "profile",
              href: "https://ch.tetr.io/u/mochbot",
            },
            {
              name: "discord",
              href: "https://discord.gg/3cEKCBGJfT",
            },
          ],
        },
      ],
    },
  ];

  const filter = (sections: Section[], search: string): Section[] =>
    sections
      .map((section) => {
        if (section.name.toLowerCase().includes(search.toLowerCase())) return section;

        const filteredSubsections = section.subsections
          ? filter(section.subsections, search)
          : null;

        if (!filteredSubsections || filteredSubsections.length === 0) return null;

        return { ...section, subsections: filteredSubsections! };
      })
      .filter((item) => item) as Section[];

  let filteredSections = $derived(filter(sections, search));

  const index = (
    sections: Section[],
    search: string,
    counter: { index: number } = { index: 0 },
  ): [IndexedSection[], number] => [
    sections.map((item, i) => ({
      ...item,
      index: item.href || item.copy ? counter.index++ : -1,
      subsections: item.subsections
        ? index(item.subsections, search, counter)[0]
        : undefined,
    })),
    counter.index,
  ];

  let [indexedSections, selectableCount] = $derived(index(filteredSections, search));

  let selection = $state(0);

  onMount(() => {
    const listener = (e: KeyboardEvent) => {
      if (e.code === "ArrowDown") {
        selection += 1;
        if (selection >= selectableCount) selection = 0;
      } else if (e.code === "ArrowUp") {
        selection -= 1;
        if (selection < 0) selection = selectableCount - 1;
      } else if (e.code === "Enter") {
        const walk = (
          sections: IndexedSection[],
          index: number,
        ): IndexedSection | null => {
          return (
            sections.find((section) => section.index === index) ??
            sections.reduce(
              (acc: null | IndexedSection, section) =>
                acc ?? walk(section.subsections ?? [], index),
              null,
            ) ??
            null
          );
        };

        const section = walk(indexedSections, selection);
        if (!section) return;

        if (section.href) {
          window.open(section.href, "_blank", "noopener noreferrer");
        } else if (section.copy) {
          navigator.clipboard.writeText(section.copy);
        }
      }
    };

    window.addEventListener("keydown", listener);

    return () => {
      window.removeEventListener("keydown", listener);
    };
  });

  $effect.pre(() => {
    search;
    selection = 0;
  });
</script>

{#snippet selectable(section: IndexedSection)}
  {@const selected = selection === section.index}
  <!-- svelte-ignore a11y_no_static_element_interactions -->
  <!-- svelte-ignore a11y_mouse_events_have_key_events -->
  <!-- svelte-ignore a11y_click_events_have_key_events -->
  <div
    class="selectable pl-1"
    class:bg-black={selected}
    class:text-white={selected}
    class:cursor-pointer={section.index !== -1}
    onmouseover={() => {
      if (section.index === -1) return;
      selection = section.index;
    }}
    onclick={async () => {
      if (section.index === -1) return;
      selection = section.index;
      await tick();

      window.dispatchEvent(new KeyboardEvent("keydown", { code: "Enter" }));
    }}
  >
    {section.name}

    {#if section.href}↗{/if}
    {#if section.copy}
      ⧉
    {/if}
  </div>
{/snippet}

{#snippet sec(section: IndexedSection)}
  <div style="">
    <div>
      {@render selectable(section)}
    </div>
    <div class="ml-1 flex relative">
      <div
        class="self-stretch bg-black w-0.5"
        style="clip-path: inset(5px 0 10px 0);"
      ></div>
      <div class="flex-1">
        {#if section.body}
          <div class="pl-2 text-gray-700">
            {section.body}
          </div>
        {/if}
        {#if section.subsections}
          {#each section.subsections as subsection, i}
            <div class="flex">
              <div class="bg-black mr-1 w-3 mt-3 h-0.5"></div>
              <div class="flex-1">
                {@render sec(subsection)}
              </div>
            </div>
          {/each}
        {/if}
      </div>
    </div>
  </div>
{/snippet}

<div class="bg-black flex justify-center items-center h-screen font-m">
  <main class="bg-white py-3 px-4 relative w-84">
    <div class="bg-black text-white py-0.5 w-full text-center">haelp.dev</div>
    <Search bind:search />

    <div>
      {#each indexedSections as section}
        {@render sec(section)}
      {/each}
      {#if indexedSections.length === 0}
        <div class="text-center text-gray-700">¯\_(ツ)_/¯</div>
      {/if}
    </div>
  </main>
</div>
