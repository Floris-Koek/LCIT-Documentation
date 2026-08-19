# Mermaid-diagrammen embedden in Confluence

Deze kennis is getest en bevestigd op een echte pagina in deze Confluence-space (ACR): [Dollevoet Integration — Level 3 Sequence Diagrams (test)](https://virtualsciences.atlassian.net/wiki/spaces/ACR/pages/1288077313). Dit is de exacte embedding-methode die deze skill gebruikt bij het opbouwen van een Confluence-pagina.

## De regel

**Een gewoon code-blok met taal `mermaid` is voldoende.** In HTML-content voor de Confluence-API is dat:

```html
<pre><code class="language-mermaid">
sequenceDiagram
    ...
</code></pre>
```

(In markdown-content voor de Confluence-API is het equivalent een ```` ```mermaid ```` fenced code block.)

Confluence rendert dit **native** als: het diagram zichtbaar bovenaan, met daaronder een **ingebouwde, inklapbare toggle** ("▽ Diagram") die de ruwe Mermaid-broncode toont. Dit gedrag zit al in het gewone code-macro/taal-highlighting van Confluence voor `mermaid` — er is geen aparte app, macro, of extra opmaak voor nodig.

## Wat NIET te doen

Tijdens het testen bleek dat de volgende, op het eerste gezicht logische toevoegingen juist een **dubbele, geneste weergave** opleveren (diagram-in-een-diagram, toggle-in-een-toggle):

- **Geen aparte "Mermaid diagram"-extensiemacro toevoegen** boven het code-blok (bijv. een Forge-app-macro met `data-extension-key`/`data-extension-type="com.atlassian.ecosystem"`). Zo'n macro bestaat in deze space en rendert op zichzelf ook een diagram, maar in combinatie met het code-blok ontstaat dubbele content.
- **Geen eigen `<details><summary>...</summary>...</details>`-wrapper om het code-blok heen bouwen** om een in-/uitklapbaar effect te forceren. Het code-blok heeft die toggle al ingebouwd; een eigen wrapper eromheen nestelt het native diagram + de native toggle binnen jouw eigen toggle.

Kortom: **alleen het kale `<pre><code class="language-mermaid">`-blok, verder niets eromheen.** Dat is tegelijk de eenvoudigste en de correcte oplossing.
