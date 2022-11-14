<script>
    import {Proskomma} from 'proskomma';
    import {SofriaRenderFromProskomma} from 'proskomma-json-tools';
    import {thaw} from 'proskomma-freeze';

    const pk = new Proskomma();
    const loaded = (async () => {

        const res = await fetch("eng_web.pkf").then((r) => {
            return r.text();
        });

        if (res.length) {
            await thaw(pk, res);
        }
        const cl = new SofriaRenderFromProskomma(
            {
                proskomma: pk,
                actions: {
                    startDocument: [
                        {
                            description: "Set up; Book heading",
                            test: () => true,
                            action: ({context, workspace}) => {
                                workspace.htmlBits = ["<h2>", context.document.metadata.document.toc, "</h2>\n"];
                            }
                        }
                    ],
                    startParagraph: [
                        {
                            description: "Start HTML para with appropriate class",
                            test: () => true,
                            action: ({context, workspace}) => {
                                const paraClass = context.sequences[0].block.subType.split(':')[1] || context.sequences[0].block.subType;
                                workspace.htmlBits.push(`<p class="${paraClass}">`)
                            }
                        }
                    ],
                    endParagraph: [
                        {
                            description: "End HTML para",
                            test: () => true,
                            action: ({workspace}) => {
                                workspace.htmlBits.push(`</p>\n`)
                            }
                        }
                    ],
                    text: [
                        {
                            description: "Output text",
                            test: () => true,
                            action: ({context, workspace}) => {
                                workspace.htmlBits.push(context.sequences[0].element.text)
                            }
                        }
                    ],
                    mark: [
                        {
                            description: "Output text",
                            test: () => true,
                            action: ({context, workspace}) => {
                                const element = context.sequences[0].element;
                                if (element.subType === 'chapter_label') {
                                    workspace.htmlBits.push(`<h3>Chapter ${element.atts['number']}</h3>\n`);
                                } else if (element.subType === 'verses_label') {
                                    workspace.htmlBits.push(`<b>${element.atts['number']}</b>&nbsp;`);
                                }
                            }
                        }
                    ],
                    endDocument: [
                        {
                            description: "Set up",
                            test: () => true,
                            action: ({workspace, output}) => {
                                output.html = workspace.htmlBits.join(``);
                            }
                        }
                    ]
                },
                debugLevel: 0
            }
        );
        const bookCode = "TIT";
        const chapter = 1;
        const docsResult = pk.gqlQuerySync('{documents { docSetId id bookCode: header(id: "bookCode")} }');
        const bookLookup = {};
        for (const docRecord of docsResult.data.documents) {
            bookLookup[docRecord.bookCode] = docRecord.id;
        }
        const docId = bookLookup[bookCode];
        const output = {};
        cl.renderDocument(
            {docId, config: {chapters: ["1"]}, output}
        );
        return output.html;
    })();
</script>


<h1>Proskomma Svelte Test</h1>

{#await loaded}
    <p>Loading...</p>
{:then rendered}
    {@html rendered}
{/await}
