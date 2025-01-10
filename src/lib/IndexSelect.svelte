<script lang="ts">
    export let columns: string[] = [];
    export let value: string = columns[0];
</script>
<select class="select" bind:value={value}>
    {#each columns as column}
        <option value={column}>{column}</option>
    {/each}
</select>