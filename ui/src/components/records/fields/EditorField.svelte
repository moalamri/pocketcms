<script>
    import { onMount } from "svelte";
    import CommonHelper from "@/utils/CommonHelper";
    import Pocketbase from "@/utils/Pocketbase";
    import Field from "@/components/base/Field.svelte";
    import TinyMCE from "@/components/base/TinyMCE.svelte";
    import RecordFilePicker from "@/components/records/RecordFilePicker.svelte";

    export let field;
    export let value = "";

    let picker;
    let editor;
    let mounted = false;
    let mountedTimeoutId = null;

    $: conf = Object.assign(CommonHelper.defaultEditorOptions(), {
        convert_urls: field.options?.convertUrls,
        relative_urls: false,
    });

    // normalize value
    // (depending on the editor plugins, `undefined` may throw an error in case the TinyMCE text functions are used)
    $: if (typeof value == "undefined") {
        value = "";
    }

    onMount(async () => {
        if (typeof value == "undefined") {
            value = "";
        }

        // slight "offset" the editor mount to avoid blocking the rendering of the other fields
        mountedTimeoutId = setTimeout(() => {
            mounted = true;
        }, 100);

        return () => {
            clearTimeout(mountedTimeoutId);
        };
    });
</script>

<Field class="form-field form-field-editor {field.required ? 'required' : ''}" name={field.name} let:uniqueId>
    <label for={uniqueId}>
        <i class={CommonHelper.getFieldTypeIcon(field.type)} />
        <span class="txt">{field.name}</span>
    </label>
    {#if mounted}
        <TinyMCE
            id={uniqueId}
            {conf}
            bind:value
            on:init={(initEvent) => {
                editor = initEvent.detail.editor;
                editor.on("collections_file_picker", () => {
                    picker?.show();
                });
            }}
        />
    {:else}
        <div class="tinymce-wrapper" />
    {/if}
</Field>

<RecordFilePicker
    bind:this={picker}
    title="Select an image"
    fileTypes={["image"]}
    on:submit={(e) => {
        editor?.execCommand(
            "InsertImage",
            false,
            Pocketbase.files.getUrl(e.detail.record, e.detail.name, {
                thumb: e.detail.size,
            }),
        );
    }}
/>
