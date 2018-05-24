# litatwiki

## SPARQL
[SPARQL-Interface](sparql.html)
Endpoint to Query revisions http://digitalhumanities.germ.univie.ac.at/litatwiki/revisions/query

other relevant endpoints:
DBPedia: https://dbpedia.org/sparql
Wikidata:

## Wikipedia/Mediawiki Exportformat als RDF:

### Namespace

``PREFIX we: <https://www.mediawiki.org/wiki/Help:Export/Export_format#>``

Die URI ist nicht ideal. Das Exportformat ist hier beschrieben: (<https://www.mediawiki.org/wiki/Help:Export#Export_format)

Gegebenfalls können die IDs der revisions abgekürzt werden:

``PREFIX : <https://de.wikipedia.org/w/index.php?title=%C3%96sterreichische_Literatur&amp;oldid=>``

### Properties

* Zeitstempel: `we:timestamp` – `<https://www.mediawiki.org/wiki/Help:Export/Export_format#timestamp>`
* Link auf andere Seite (outlink): `we:linksTo` – `<https://www.mediawiki.org/wiki/Help:Export/Export_format#linksTo>`
* Anzahl der Links: `we:links_count` – `<https://www.mediawiki.org/wiki/Help:Export/Export_format#links_count>`
* Größe des Edits in bytes: `we:bytes` – `<https://www.mediawiki.org/wiki/Help:Export/Export_format#bytes>`
* ID des bearbeitenden users: `we:contributor_id` – `<https://www.mediawiki.org/wiki/Help:Export/Export_format#contributor_id>``
* Username des Bearbeiters/der Bearbeiterin: `we:contributor_username` – <https://www.mediawiki.org/wiki/Help:Export/Export_format#contributor_username>
* IP (bei anonymen Edits): `we:ip` `<https://www.mediawiki.org/wiki/Help:Export/Export_format#ip>`
oder
* `<https://www.mediawiki.org/wiki/Help:Export/Export_format#contributor_ip>` -- _Fehler!_


## Alternative Identifier: zu DBPedia-Resource und Wikidata ausgehen von `we:linksTo`


### DBPedia-IDs, hier für den letzten Edit:

```
PREFIX we: <https://www.mediawiki.org/wiki/Help:Export/Export_format#>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX : <https://de.wikipedia.org/w/index.php?title=%C3%96sterreichische_Literatur&amp;oldid=>
SELECT DISTINCT ?deDBPedia ?dpResource WHERE {

  #Verlinkte Ressourcen in der letzten (?) revision
   :176308376 we:linksTo ?deDBPedia .

  #DBPedia Endopoint: https://dbpedia.org/sparql
  SERVICE <https://dbpedia.org/sparql> {
  	?dpResource <http://www.w3.org/2002/07/owl#sameAs> ?deDBPedia .
  }
}
LIMIT 1000
```

### Wikidata zu den verlinkten Seiten:

```
PREFIX we: <https://www.mediawiki.org/wiki/Help:Export/Export_format#>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX : <https://de.wikipedia.org/w/index.php?title=%C3%96sterreichische_Literatur&amp;oldid=>
SELECT DISTINCT ?deDBPedia ?dpResource (?otherID AS ?wikidata) WHERE {

  #Verlinkte Ressourcen in der letzten (?) revision
   :176308376 we:linksTo ?deDBPedia .

  #DBPedia Endpoint: https://dbpedia.org/sparql
  SERVICE <https://dbpedia.org/sparql> {
  	?dpResource <http://www.w3.org/2002/07/owl#sameAs> ?deDBPedia .
    ?dpResource <http://www.w3.org/2002/07/owl#sameAs> ?otherID .
    #Get only wikidata
    FILTER STRSTARTS(STR(?otherID), 'http://www.wikidata.org/entity/') .
  }
}

LIMIT 1000
```


```
PREFIX we: <https://www.mediawiki.org/wiki/Help:Export/Export_format#>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
SELECT ?timestamp ?bytes WHERE {
  ?revision  we:bytes ?bytes ;
             we:timestamp ?timestamp.
}
ORDER BY ASC(?timestamp)
```
