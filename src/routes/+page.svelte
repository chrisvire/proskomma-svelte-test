<script>
    import {Proskomma} from 'proskomma';

    const pk = new Proskomma();
    const promise = pk.gqlQuery("{ id }");
</script>


<h1>Welcome to SvelteKit</h1>
<p>Visit <a href="https://kit.svelte.dev">kit.svelte.dev</a> to read the documentation</p>

{#await promise}
    <p>loading</p>
{:then data} 
    <pre>{JSON.stringify(data, null, 2)}</pre>
{/await}