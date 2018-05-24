# litatwiki

Wikipedia-Artikel [Österreichische Literatur](https://de.wikipedia.org/wiki/%C3%96sterreichische_Literatur)

## Wikipedia/Mediawiki Exportformat als RDF:

Dokumentation des Wikipedia-Exportformats: (<https://www.mediawiki.org/wiki/Help:Export#Export_format)
XML-Elemente und Attribute sind im Datensatz gemappt auf die nicht dereferenzierbare Base-URI: https://www.mediawiki.org/wiki/Help:Export#Export_format

``PREFIX we: <https://www.mediawiki.org/wiki/Help:Export/Export_format#>``

### Properties

* Zeitstempel: `we:timestamp` – `<https://www.mediawiki.org/wiki/Help:Export/Export_format#timestamp>`
* Link auf andere Seite (out-link): `we:linksTo` – `<https://www.mediawiki.org/wiki/Help:Export/Export_format#linksTo>` (Eigentlich nicht im Exportformat; wird hierfür einen neuen Namespace geben)
* Anzahl der Links: `we:links_count` – `<https://www.mediawiki.org/wiki/Help:Export/Export_format#links_count>` (Eigentlich nicht im Exportformat; wird hierfür einen neuen Namespace geben)
* Größe des Edits in bytes: `we:bytes` – `<https://www.mediawiki.org/wiki/Help:Export/Export_format#bytes>` (Im Exportformat ein Attribut)
* ID des bearbeitenden users: `we:contributor_id` – `<https://www.mediawiki.org/wiki/Help:Export/Export_format#contributor_id>``
* Username des Bearbeiters/der Bearbeiterin: `we:contributor_username` – <https://www.mediawiki.org/wiki/Help:Export/Export_format#contributor_username>
* IP (bei anonymen Edits): `we:ip` `<https://www.mediawiki.org/wiki/Help:Export/Export_format#ip>`
oder
* `<https://www.mediawiki.org/wiki/Help:Export/Export_format#contributor_ip>` -- _Fehler!_

### Artikelversionen
``PREFIX : <https://de.wikipedia.org/w/index.php?title=%C3%96sterreichische_Literatur&amp;oldid=>``

## SPARQL

RDF zum Ausgabeformat kann hier abgerufen werden:

* Endpoint: http://digitalhumanities.germ.univie.ac.at/litatwiki/revisions/query
* [SPARQL Query Interface](https://ingoboerner.github.io/litatwiki/sparql.html)

Andere relevante Endpoints:

* DBPedia: https://dbpedia.org/sparql
* Wikidata: https://query.wikidata.org/sparql

## Beispielabfragen:

### Metadaten zu Artikelversionen
```
PREFIX we: <https://www.mediawiki.org/wiki/Help:Export/Export_format#>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
SELECT ?revision ?timestamp ?userName ?ip ?bytes ?links  WHERE {
  ?revision we:timestamp ?timestamp ;
            we:bytes ?bytes ;
  			we:links_count ?links .
  OPTIONAL {
    ?revision we:ip ?ip.
  }

  OPTIONAL {
  	?revision we:contributor_username ?user;
  }

  BIND(IF(BOUND(?user),?user, '<anon>') AS ?userName) .  
}
ORDER BY DESC(?timestamp)
```

### Alternative Identifier zu DBPedia-Resource und Wikidata ausgehend von `we:linksTo`


#### DBPedia-IDs für die `we:linksTo` verlinkten Artikelseiten, hier für den letzten Edit:

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
```

#### Wikidata-IDs zu den verlinkten Seiten:

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
```

### Verlinkte Autor/inn/enseiten mit zusätzlichen Informationen

Ermittelt werden die Koordinaten der Geburtsorte jener Personen, die mit dem propery 'occupation' (P106) als Schriftsteller (Q36180) versehen sind:

```
PREFIX we: <https://www.mediawiki.org/wiki/Help:Export/Export_format#>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX wd: <http://www.wikidata.org/entity/>
PREFIX wdt: <http://www.wikidata.org/prop/direct/>

SELECT DISTINCT ?page (?altid AS ?wikidata) ?geb ?coords WHERE {
  # Alle Verlinkungen in der aktuellsten Fassung der Seite 'Österreichische Literatur':
  <https://de.wikipedia.org/w/index.php?title=%C3%96sterreichische_Literatur&amp;oldid=176308376> we:linksTo ?page .

  # In DBpedia: Abfrage der Wikidata URI:
  SERVICE <https://dbpedia.org/sparql> {
  ?dbpID <http://www.w3.org/2002/07/owl#sameAs> ?page ;
        <http://www.w3.org/2002/07/owl#sameAs> ?altid
  FILTER(STRSTARTS(STR(?altid),'http://www.wikidata.org/entity/'))
  }

  # Daten zu Geburtstag, Geburtsort und die Koordinaten des Geburtsorts :
  SERVICE <https://query.wikidata.org/sparql> {
    OPTIONAL {
    ?altid wdt:P106 wd:Q36180 ;
     	     wdt:P569 ?geb ;
    	     wdt:P19 ?ort .
  	?ort wdt:P625 ?coords .
    }       
  }  
}

```
