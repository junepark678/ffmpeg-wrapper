<script lang="ts">
    export let output_blob: string;
    export let output_blob_type: string;

    $: output_blob, console.log(output_blob);

</script>

<div id="output">
    {#if output_blob_type === "image/gif"}
        <img
            id="output-image"
            class="p-5 w-auto"
            srcset={output_blob}
            alt=""
        />
    {:else}
        <video id="output-video" class="p-5 w-auto" controls>
            <track kind="captions" />
            <source src={output_blob} type={output_blob_type} />
        </video>
    {/if}
</div>