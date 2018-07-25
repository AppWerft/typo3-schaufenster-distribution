# Adding new facet type

## Creating a new typename

First we have to create a new string for type. The name will used in `Resources/Private/Partials/Facets/Facet/` as classname.

Inside the template two parts are implemented:

### Javascript embedding

 Example
```
<s:page.script name="find_geohash">
	$(function(){/* LOGIC */});
</s:page.script>
```

### HTML embedding

Example

```
<div class="facetMap-container">
	<div id="schaufenster_heatmap" class="heatmapContainer"></div>
</div>
```
The logic inside the script references to DOM, declared in HTML snippet

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
