<style>
  .add-file {
    display: flex;
    flex-direction: row;
    justify-content: space-around;
  }

  button {
    margin: 1em, 0;
    padding-top: 0.5em;
    padding-bottom: 0.5em;
    padding-left: 1em;
    padding-right: 1em;
    background-color: var(--main-bg-color);
    color: var(--main-fg-color);
    border: 1px solid var(--main-fg-color);
    border-radius: 0;
    font-size: smaller;
    overflow: hidden;
    box-shadow: none;
  }
  button:hover,
  button:focus {
    outline: none;
    box-shadow: inset 1px 1px 0 var(--main-fg-color),
      inset -1px -1px 0 var(--main-fg-color);
  }

  button:active {
    box-shadow: inset 2px 2px 0 var(--main-fg-color),
      inset -2px -2px 0 var(--main-fg-color);
  }
</style>

<script>
  import FileBrowser from "../FileBrowser.svelte";
  import Icon from "../Icon.svelte";
  import { selectedProject, settings } from "../stores";
  let files, f;

  $: if (files) {
    f = files[0];
    files = null;
  }
  async function addBinaryToProject() {
    await fetch(
      `${$settings.backendUrl}/add_binary_to_project?id=${$selectedProject.session_id}&name=${f.name}`,
      {
        method: "POST",
        headers: {
          "Content-Type": "application/json",
        },
        body: await f.arrayBuffer(),
      }
    ).then(async (r) => {
      if (!r.ok) {
        throw Error(JSON.stringify(await r.json(), undefined, 2));
      }
      $selectedProject = await fetch(
        `${$settings.backendUrl}/get_project_by_id?id=${$selectedProject.session_id}`
      ).then((r) => {
        if (!r.ok) {
          throw Error(r.statusText);
        }
        return r.json();
      });
      return await r.json();
    });
  }
</script>

<div class="add-file">
  <FileBrowser bind:files="{files}" />
  {#if f}
    <button on:click="{addBinaryToProject}"
      ><Icon url="/icons/binary.svg" /> Add Binary to Project</button
    >
  {/if}
</div>
