<script>
    import {Proskomma} from 'proskomma';
    import {thaw} from 'proskomma-freeze';

    const pk = new Proskomma();
    const promise = pk.gqlQuery("{ id }");
    const loaded = (async () => {
        console.log("test");

        const res = await fetch("eng_web.pkf").then((r) => {
            return r.text();
        });

        if(res.length) {
            await thaw(pk, res);
        }
        //console.log(res);
        return await pk.gqlQuery("{docSets { id }}");
    })();
</script>


<h1>Welcome to SvelteKit</h1>
<p>Visit <a href="https://kit.svelte.dev">kit.svelte.dev</a> to read the documentation</p>

{#await promise}
    <p>loading</p>
{:then data} 
    <pre>{JSON.stringify(data, null, 2)}</pre>
{/await}

{#await loaded}
    <p>loading</p>
{:then data} 
    <pre>{JSON.stringify(data, null, 2)}</pre>
{/await}