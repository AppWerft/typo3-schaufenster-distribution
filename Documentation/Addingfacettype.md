# Adding new facet type

## Creating a new typename

First we have to create a new string for type. The name will used in `Resources/Private/Partials/Facets/Facet/` as file name.

### Referencing in typoscript setup

In `plugin.tx_find.settings.facets.type` we can link the partial.

```
  plugin.tx_find.settings.facets {
    1 {
        id = Karte
        field = geoLocationPoint
        type = Heatmap // <= the name
        sortOrder = index
        fetchMaximum = 1000
    }
  }
```

This name (in our case `Heatmap`) references to file name inside folder `Resources/Private/Partials/Facets/Facet/` with name `Heatmap.html`.

Inside the [template partial]() two parts are implemented:

### Javascript embedding

 Example
```
<s:page.script>
	$(function(){/* LOGIC */});
</s:page.script>
```   
For LOGIC there are two solutions:

* calling of function in global namespace (pullation)
* local coding inside the callback

### Interactions

Javascript needs parameters. There are two sources: typoscript and backend data.

#### TypoScript

The keyword is `facetInfo`. With this syntax below you can access properties of `facets`:
`facetInfo.fetchMaximum` or `facetInfo.field`.
Example:

```
var facetFetchMaximum = <f:format.htmlentitiesDecode>{facetInfo.fetchMaximum}</f:format.htmlentitiesDecode>
```

#### Access facetData

With `data:facetData.values` the facet datas from solr are accessible.
This example exposes this data as JSON:

```
facetData:<f:format.htmlentitiesDecode>{s:format.json(data:facetData.values)}</f:format.htmlentitiesDecode>,
```
Here the entire snippet:

```
tx_schaufenster_facetHeatmap.initHeatmap ({
  facetData:<f:format.htmlentitiesDecode>{s:format.json(data:facetData.values)}</f:format.htmlentitiesDecode>,
  facetFetchMaximum:<f:format.htmlentitiesDecode>{facetInfo.fetchMaximum}</f:format.htmlentitiesDecode>
});
```

### Possible pitfall

Since TYPO3 v8 the Screenhelpers escaped all output. I.e. all `"` will outputed as `&quot;`. The `<f:format.htmlentitiesDecode>` decodes the output to get the right syntax.

### HTML embedding

Example

```
<div class="facetMap-container">
	<div id="schaufenster_heatmap" class="heatmapContainer"></div>
</div>
```
The logic inside the script references to DOM, declared in HTML snippet.
