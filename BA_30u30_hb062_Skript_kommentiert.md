# Bachelorarbeit: Netzwerkanalyse #30u30

**Hochschule der Medien, Stuttgart**
Hannah Bauer, Matr. Nr. 37000
Sommersemester 2022
Stand: 02.06.2022

*Dieses Skript enthält den Code, anhand dessen die erhobenen Daten ausgewertet, analysiert und visualisiert wurden. Es handelt es sich nicht ausschließlich um Ergebnisse, die in der finalen Bachelorarbeit wiedergegeben oder als Grafik eingebunden werden. Stattdessen sind auch Analysen enthalten, die bei der Ergebnisdarstellung letztlich mangels Aussagekraft bzw. Erkenntnisgehalt oder aus anderen Gründen nicht inkludiert wurden.*

## Inhaltsübersicht: TBA
---
# Einstieg: Daten einlesen & Gesamtnetzwerk generieren

Zu Beginn werden die wichtigsten Packages geladen.

```{r wichtigste packages abrufen}

# Laden der wichtigsten Packages
library(igraph)
library(igraphdata)
library(knitr) 
library(rmarkdown) 
library(dplyr)
library(influenceR)

```
Anschließend werden Edge- und Nodelist eingelesen, um daraus das Gesamtnetzwerk zu erstellen.

```{r Gesamtnetzwerk einlesen}

# Einlesen der Edgelist
el <- read.csv("https://raw.githubusercontent.com/hb062/BA_30u30/main/el.csv", header=T, as.is=T, sep = ",", fileEncoding="UTF-8")

head(el) # zeigt die ersten Zeilen der Edgelist an

# Einlesen der Nodelist
nodes <- read.csv("https://raw.githubusercontent.com/hb062/BA_30u30/main/nl.csv", header=T, as.is=T, sep = ",", fileEncoding="UTF-8")

head(nodes) # zeigt die ersten Zeilen der Nodelist an

# Verknüpfen von Edge- und Nodelist zu einer Matrix
edgematrix <-as.matrix(el)
g <- graph_from_data_frame(d=edgematrix, vertices=nodes, directed=TRUE) # erstellt ein gerichtetes Netzwerk

g

```

# Visualisierungsparameter

Im Folgenden werden grundsätzliche Visualisierungseinstellungen definiert, die für alle generierten Netzwerke gelten sollen.

## Grundsätzliche Darstellung
```{r Edge-Parameter}
E(g)$curved=.2 
E(g)$arrow.size <- .00001 # Definiert die Größe der Pfeilspitze auf .00001
E(g)$vertex.label.family="sans" # legt die Schriftartenfamilie fest
E(g)$color="grey40" # definiert die Kantenfarbe auf grau
```

# Two-Mode-Netzwerk: Personen und Organisationen
Neben den Edges sollen auch die Nodes auf eine bestimmte Weise visualisiert werden: Je nach Knotentyp oder anderen Attributen werden die Knoten unterschiedlich eingefärbt oder anhand verschiedener Formen unterscheidbar gemacht.

```{r Visualisierung der verschiedenen Knotentypen}
V(g)[V(g)$type == 1]$shape <- "circle" # Personen als Kreis darstellen
V(g)[V(g)$type == 2]$shape <- "square" # Organisationen als Quadrat darstellen
V(g)[V(g)$member == 1]$color <- "firebrick2" # Knoten der 30u30-Mitglieder rot färben
V(g)[V(g)$member == 2]$color <- "orange" # Knoten der Mentor:innen orange färben

```
Nicht nur zwischen Knoten des Type 1 oder 2 wird optisch unterschieden, auch die Knoten des Type 2 werden untereinander verschieden dargestellt, je nach Organisationstyp.

```{r  Einfärben der unterschiedlichen Organisationstypen}

V(g)[(V(g)$category == 1)]$color <- "dodgerblue4" # Unternehmen dunkelblau
V(g)[(V(g)$category == 2)]$color <- "cadetblue3" # Agenturen hellblau
V(g)[(V(g)$category == 3)]$color <- "seashell2" # NGO / NPO in beige
V(g)[(V(g)$category == 4)]$color <- "lightgoldenrod1" # Polit. Org. in hellgelb
V(g)[(V(g)$category == 5)]$color <- "seagreen" #Hochschulen grün
V(g)[(V(g)$category == 6)]$color <- "violetred" # Vereine pink
V(g)[(V(g)$category == 7)]$color <- "pink1" #Stipendien rosa
V(g)[(V(g)$category == 8)]$color <- "mediumorchid" #Medienunternehmen lila
V(g)[(V(g)$category == 9)]$color <- "gainsboro" #Sonstige grau
V(g)[(V(g)$category == 10)]$color <- "tan" #Forschungsinstitute braun

g
```

## Gesamtnetzwerk plotten
Nach den grundsätzlichen Visualisierungsanpassungen wird nun eine erste Visualisierung des Gesamtnetzwerks erzeugt.

```{r Plot von g}
g

# Generiert einen ersten, simplen Plot
plot(g,
     layout=layout_with_kk, # legt Layout fest
     vertex.label=NA, # blendet Knotenlabels aus
     vertex.size=.2, # setzt Knotengröße auf .2, um alle abbilden zu können
     asp=0) # der gesamte Raum soll für die Visualisierung genutzt werden
```

Zur Überprüfung, ob die Daten korrekt eingelesen wurden, werden die Edge- und Node-Attribute abgerufen.

```{r Attribute von g}

E(g)
V(g)
edge_attr(g)
vertex_attr(g)

```

Gesamtnetzwerk g wird erneut geplottet, mit schönerer Visualisierung.

```{r Schönere Visualisierung von g}
g

g_simple <- simplify(g, remove.multiple = TRUE) # entfernt doppelte Kanten

# Visualisierung des neu generierten Gesamtnetzwerks g
plot(g_simple,
     layout=layout_with_kk, # legt Layout fest
     edge.curved=curve_multiple(g_simple), # verhindert, dass sich Kanten überlagern
     vertex.label=NA, # entfernt die Knotenbeschriftung für bessere Übersichtlichkeit
     edge.arrow.size=0.01, # verkleinert die Pfeilgröße
     edge.arrow.color="white",
     vertex.size=.3, # setzt die Knotengröße auf 0.3
     main="Gesamtnetzwerk der #30u30-Jahrgänge 2017 bis 2021",
     sub="organisat. Verbindungen und Mentoring-Beziehungen", 
     asp=0)

```

# Auswertung des Gesamtnetzwerks: Berechnungen zu einzelnen Bestandtteilen

Nach einer ersten Visualisierung wird nun die Verteilung der einzelnen Attribute überprüft, um die Zusammensetzung des Netzwerks zu analysieren.

```{r Knotentypen}

# Wie viele Knoten des Type 1 und 2 sind im Netzwerk enthalten?
table(nodes$type)

```

## Attribute der Knoten des Type 1 
Zunächst werden Attribute betrachtet, die sich auf natürliche Personen (type 1) beziehen.

```{r Jahrgangsverteilung}

# Schlüsselt Verteilung der Ausprägung des Node-Attr. "entry" auf
# Prüft u. a., ob bei allen Jahrgängen vollständig und korrekt 30 Mitglieder erfasst wurden:
table(nodes$entry)

```

```{r Geschlechterverteilung}
# Wie viele Männer, wie viele Frauen sind im Gesamtnetzwerk enthalten?
table(nodes$sex)

# Geschlechterverteilung bei #30u30-Alumni und Mentor:innen
table(nodes$mentor, nodes$sex)

# Geschlechterverteilung nach Jahrgängen:
table(nodes$mentor, nodes$sex, nodes$entry)

```

```{r Mentor:innen}

# Anzahl der Mentor:innen
table(nodes$mentor)

```

Bei den folgenden Attributen wird nicht zwischen Mentor:innen und #30u30 unterschieden, da diese nur für #30u30 definiert wurden.

```{r Bildungsgrad / Abschluss}

# Verteilung der höchsten Bildungsabschlüsse
table(nodes$education)

# Berechne Anteil der #30u30 mit Hochschulabschluss
146/150

# Anteil der Masterabsolvent:innen an allen Akademiker:innen
103/145

# Anteil der Bachelorabsolvent:innen an allen Akademiker:innen:
38/145

# Bildungsabschlüsse nach Geschlecht
table(nodes$education, nodes$sex)

# Bildungsabschlüsse und Akadem. Auslandserfahrung
table(nodes$education, nodes$exchange)

# Verteilung des Bildungsgrads auf Jahrgänge
table(nodes$education, nodes$entry)

```

```{r Mitgliedschaft in PR-Vereinen}

# Wie viele Personen waren Mitglied, kein Mitglied o. leitendes Mitglied?
table(nodes$membership)

# Mitgliederzahlen in den Jahrgängen
table(nodes$membership, nodes$entry)

# Mitgliederzahlen nach Bildungsgrad
table(nodes$membership, nodes$education)

# Mitgliederzahlen nach Geschlecht
table(nodes$membership, nodes$sex)

```

```{r Stipendien / Begabtenförderung}

# Wie viele Stipendiat:innen und wie viele Nicht-Stipendiat:innen?
table(nodes$scholarship)

# Stipendien nach Jahrgang
table(nodes$scholarship, nodes$entry)

# Stipendien und höchster Abschluss
table(nodes$scholarship, nodes$education)

# Stipendien und Auslandsstudium
table(nodes$scholarship, nodes$exchange)

# Stipendien und Young Prof. des Jahres - handelt es sich dabei v. a. um Stipendiat:innen?
table(nodes$scholarship, nodes$winner)
```

```{r Auslandssemester}

# Wie viele Personen haben ein Auslandssemester, - studium, beides oder keinen Auslandsaufenthalt absolviert?
table(nodes$exchange)

# Auslandssemester nach Jahrgängen
table(nodes$exchange, nodes$entry)

# Auslandssemester und berufliche Auslandsaufenthalte
table(nodes$exchange, nodes$abroad)

```

```{r Arbeitserfahrung im Ausland}

# Wie viele Personen haben (zeitweise) im Ausland gearbeitet?
table(nodes$abroad)

```

```{r Young Professionals des Jahres-Award}
# Reine Anzahl der Gewinner:innen
table(nodes$winner)

# Verteilung der Gewinner:innen auf die Jahrgänge
table(nodes$winner, nodes$entry)

# Geschlechterverteilung der Gewinner:innen
table(nodes$winner, nodes$sex)

# Höchster Bildungsgrad der Gewinner:innen
table(nodes$winner, nodes$education)

# Wie viele der Gewinner:innen haben im Ausland studiert?
table(nodes$winner, nodes$exchange)

# Wie viele der Gewinner:innen haben im Ausland gearbeitet?
table(nodes$winner, nodes$abroad)

```

## Attribute der Knoten des Type 2
Nun wird auch die Verteilung der Attribute bei Knoten des Typs 2 (Organisationen) ausgewertet.

```{r Organisationstyp}

# Anzahl der Organisationstypen im Gesamtnetzwerk?
table(nodes$category)

```

```{r Hochschularten}

# Bei Hochschulen: Welche Hochschulart kommt wie oft im Gesamtnetzwerk vor?
table(nodes$ownership)

```

```{r Sponsoren / Partnerunternehmen}

# Zahl der Organisationen, die Sponsoren von #30u30 sind oder waren:
table(nodes$sponsor)

# Um welche Organisationstypen handelt es sich dabei?
table(nodes$sponsor, nodes$category)

```

# Gesamtnetzwerk der #30u30-Mitglieder
Zur weiteren Analyse wird das Netzwerk der #30u30-Mitglieder der Jahrgänge 2017 - 2021 generiert, ohne deren Mentor:innen.

```{r Erstellen & Plot des #30u30-Mitgliedernetzwerks (Jg. 2017 - 2021)}

g 

members <- delete.vertices(g, V(g)[mentor == 2]) # löscht die Knoten der Mentor:innen aus dem Gesamtnetzwerk

members 

members1 <- delete.vertices(members, V(members)[degree(members, mode="all")=="0"]) # löscht isolierte Knoten ohne Beziehungen

members2 <- simplify(members1, remove.multiple = TRUE) # Mehrfache Beziehungen werden nur einfach angezeigt (Kantenvereinfachung)

V(members2)[mentor==1]$color <- "firebrick1" # färbt Knoten der #30u30 rot

members2

ind <- degree(members2, mode="in")

# Erzeugt einen Plot, in dem Knoten mit hohem Indegree-Wert größer dargestellt werden

plot(members2,
     layout=layout_with_kk,
     vertex.size=ind*1.2,
     vertex.label.cex=.2,
     vertex.label.color="black",
     vertex.label.family="sans",
     edge.arrow.size=.01,
     edge.curved=curve_multiple,
     asp=0)

# Erzeugt den selben Plot, nur ohne Vertex-Labels
plot(members2,
     layout=layout_with_kk,
     vertex.size=ind*1.2,
     vertex.label=NA,
     vertex.frame.color="white",
     edge.arrow.size=.01,
     edge.curved=curve_multiple,
     asp=0)

```
Wie viele Organisationen sind im Mitgliedernetzwerk enthalten?

```{r Anzahl Organisationen}

org_mem <- V(members2)$type == 2
sum(org_mem, na.rm=T) # liefert Summe aller Knoten des Type 2 im Netzwerk

```

Welche Organisationen stehen mit den meisten #30u30-Mitgliedern in Verbindung? Hierzu wird die Indegree-Verteilung betrachtet, d. h. wie viele Beziehungen auf die Knoten eingehen.

```{r Gesamtnetzwerk Indegree-Verteilung}
indmembers <- degree(members2, mode="in")
indmembers
sort(indmembers) # sortiert Knoten anhand ihres Indegree-Werts

```

Erstellt ein Teilnetzwerk populärer Organisationen, in dem nur Organisationen mit indegree > 4 dargestellt werden.

```{r Populäre Organisationen - nur indegree > 4}

members2

members_ind5 <- delete_vertices(members2, V(members2)[(type == 2) & (degree(members2, mode="in")<5)]) # löscht Knoten mit Indegree-Wert unter 5

delete_vertices(members_ind5, V(members_ind5)[degree(members_ind5, mode="all")== 0]) # löscht isolierte Knoten

ind <- degree(members_ind5, mode="in")

# Generiert den indegree-skalierten Plot
plot(members_ind5,
     layout=layout_with_kk,
     vertex.size=ind*1.2,
     vertex.label.color="black",
     vertex.label.cex=.1,
     vertex.label.family="sans",
     vertex.frame.color="white",
     edge.arrow.size=.001,
     edge.arrow.color="white",
     main="Populäre Organisationen im Mitgliedernetzwerk",
     sub="Knoten des Type 2 mit indegree-Wert > 4",
     asp=0)
```

```{r Populäre Organisationen - nur indegree > 9}
members2

members_ind10 <- delete_vertices (members2, V(members2)[(type == 2) & (degree(members2, mode="in")<10)]) # löscht Knoten des Type 2 mit Indegree-Wert unter 10
members_ind10 <- delete_vertices (members_ind10, V(members_ind10)[degree(members_ind10, mode="all")== 0]) # löscht isolierte Knoten

ind <- degree(members_ind10, mode="in")

# Generiert den Plot 
plot(members_ind10,
     layout=layout_with_kk,
     vertex.size=ind*1.2, # skaliert Knotengröße entspr. des Indegree-Werts und multipliert mit Faktor 1,2
     vertex.label.cex=.2,
     vertex.label.color="black",
     vertex.frame.color="white",
     edge.arrow.size=.001,
     edge.arrow.color="white",
     edge.curved=curve_multiple(members_ind10), # verhindert, dass sich Kanten überlagern
     main="Die populärsten Organisationen im Mitgliedernetzwerk",
     sub="Knoten des Type 2 mit indegree-Wert > 9",
     asp=0) # definiert, dass der gesamte Platz ausgenutzt werden soll
```

## Clusteranalyse
Verschiedene Cluster im Netzwerk der Mitglieder anzeigen

```{r Cluster in Netzwerk}

clusters(members2)
# Anzahl der Cluster: 6 - Größe der Cluster: 961, 6, 5, 4, 2, 9

clg_mem <- cluster_walktrap(members2) #cluster_walktrap-Funktion sucht nach eng-verbundenen Subgraphs (communities) im Graphen über "random walks" - basiert auf der Idee, dass kurze Pfade in der selben Community bleiben

membership(clg_mem) # Gibt die Aufteilung der Knoten in Communities an
```

```{r Größe der Cluster berechnen & sortieren}

size_clgmem <- sizes(clg_mem)
sort(size_clgmem)

```

Mitglieder der größten Community

```{r Größte Community}
communities(clg_mem)
# Community no. 8 ist mit 107 Knoten die größte im Mitglieder-Netzwerk:
communities(clg_mem)[[8]]

```

Berechne Modularität im Netzwerk

```{r Modularität Mitgliedernetzwerk}

modularity(clg_mem) # 0,6722954

#Prozentwert
modularity(clg_mem)*100 # 67,23 %
```

```{r Visualisierung der Cluster}

# Um die Überschneidungen im Gesamtnetzwerk besser zu greifen, folgt eine Clusteranalyse:
clg_mem
par(mfrow=c(1,1), mar=c(0,0,1,2))

#Visualisierung der Cluster im Mitgliedernetzwerk
plot(clg_mem, 
     members2, 
     main="Clusteranalyse der #30u30",
     sub="JG 2017 - 2021",
     edge.curved=curve_multiple(members2),
     vertex.frame.color="white",
     edge.arrow.size=.0002, 
     edge.arrow.color="white", 
     edge.curved=.2,
     vertex.size=5,
     vertex.label.degree=0, 
     vertex.label.cex=.15,
     vertex.label.family="sans",
     vertex.label.font=2,
     vertex.label.color="black")

# Cluster-Visualisierung einmal ohne Vertex-Labels
plot(clg_mem, 
     members2, 
     main="Clusteranalyse der #30u30",
     sub="JG 2017 - 2021",
     edge.curved=curve_multiple(members2),
     vertex.frame.color="white",
     edge.arrow.size=.0002, 
     edge.arrow.color="white",
     edge.color="grey",
     edge.curved=.2,
     vertex.size=2,
     vertex.label=NA)

```

## Berechnungen 

### Dichte
Berechne die Dichte des Mitgliedernetzwerks: Wie viele aller potentiellen Beziehungen sind realisiert?

```{r Netzwerkdichte}

edge_density(members2)
  # Ergebnis: 0.001317328

# Prozentwert berechnen 
edge_density(members2)*100
  # Ergebnis: 0,132 %

```
### Degree-Zentralitäts-Werte

```{r Degree-Zentralität}

centralization.degree(members2, mode = "all") # 0,007818382
centralization.degree(members2, mode = "in") # 0,01288145
centralization.degree(members2, mode = "out") # 0,01693825

```

### Closeness-Zentralität
Closeness-Zentralität ist ein Maßstab dafür, wie viele Schritte es braucht, um jeden anderen Knoten von einem bestimmten Knoten im Netzwerk aus zu erreichen

```{r Closeness(-Zentralität)}

clo_mem <- closeness(members2)
sort(clo_mem) # sortiert die Werte

# Zentralisierter Closeness-Wert: Wie dicht stehen die Knoten beieinander?
centr_clo(members2, mode = "all")$centralization
  # Ergebnis: 1,635668
```
### Pfaddistanz
```{r Pfaddistanzen}

# Durchschnittliche Pfaddistanz
average.path.length(members2) # Funktion berechnet PD nur für gerichtete Kanten (entlang der Pfeilrichtung), daher nicht passend

mean_distance(members2, directed=FALSE) # stattdessen wird directed hier auf false gesetzt, sonst würden nur die Pfade einer Person zu einer Organisationen / deren direkten Verbindungen berücksichtigt.
  # Ergebnis:5,97578

# Was ist die längste Pfaddistanz im Mitglieder-Netzwerk?
diameter(members2, directed=FALSE)
  # Ergebnis: 12 Schritte

# Welche zwei Knoten sind am weitesten voneinander entfernt?
farthest_vertices(members2, directed=FALSE)
  # Antwort: 3M, Stipendium der Akademie für emotionale Intelligenz
```

### Betweenness(-Zentralität)

```{r Betweenness}

centr_betw(members2, directed=TRUE)
betw <- betweenness(members2)
sort(betw, decreasing = T) # Sortiert Betweenness-Werte nach ihrer Größe

```
Nach der Berechnung sollen nun auch visuell die Broker-Organisationen hervorgehoben werden.

```{r Plot: Broker-Organisationen}

# Wir können die Ausgabe von betweenness()  einer Variablen im Netz zuordnen und die Knoten entsprechend dimensionieren.
V(members2)$betweenness <- betweenness(members2)

V(members2)$label <- ifelse(V(members2)$type==2, V(members2)$name, NA) # Label nur bei Knoten des Type 2 anzeigen

plot(members2, 
     vertex.label.cex = .6, 
     vertex.label.color = "black", 
     vertex.size = V(members2)$betweenness) # Größe angepasst an betweenness

# Da die  Betweenness-Zentralität sehr groß sein kann, wird sie im folgenden normalisiert, indem man sie durch das Maximum teilt und mit einem Skalar multipliziert, bevor man sie plottet. So erkennen wir die Broker besser.
plot(members2,
     layout=layout_with_kk,
     vertex.label.cex =.3, 
     vertex.label.color = "black", 
     vertex.label.family="sans",
     edge.arrow.size=.00001,
     edge.arrow.color="white",
     asp=0,
     vertex.size = V(members2)$betweenness/max(V(members2)$betweenness) * 10)

```

## Mitgliedernetzwerk: Arbeitsplätze 2022

Was machen die Talente der fünf Jahrgänge heute, im Jahr 2022?

```{r Arbeitsplätze der Mitglieder im 2022}

members

members22 <- subgraph.edges(members, E(members)[year == 2022]) # selektiert nur Beziehungen aus dem Jahr 2022

members22

mem22 <- simplify(members22, remove.multiple = TRUE) # entfernt doppelte Kanten

# Generiert den Plot
plot(mem22,
     layout=layout_with_kk,
     edge.curved=curve_multiple(mem22), # verhindert, dass sich Kanten überlagern
     vertex.frame.color=NA,
     edge.curved=.2,
     vertex.size=3, 
     vertex.label.cex=.2,
     vertex.label.family="sans",
     vertex.label.font=2,
     edge.arrow.size=.001,
     edge.arrow.color="white",
     vertex.label.color="black",
     main="Aktuelle Stationen der #30u30-Jahrgänge 2017 bis 2021",
     sub="Stand März 2022",
     asp=0)
```

Wo überschneiden sich 2022 die Karrieren der #30u30? Hierzu werden nur die Organisationen selektiert, bei der mindestens zwei Personen tätig sind.

```{r Mitgliedernetzwerk 2022 - nur indegree > 1}
mem22

mem22_ind <- delete_vertices (mem22, V(mem22)[(type == 2) & (degree(mem22, mode="in")<2)]) # löscht Knoten des Type 2 mit nur einer Verbindung

mem22_ind2 <- delete_vertices (mem22_ind, V(mem22_ind)[(type == 1) & (degree(mem22_ind, mode="out")== 0)]) # löscht isolierte Personen

ind <- degree(mem22_ind2, mode="in")

# Generiert den Plot
plot(mem22_ind2,
     layout=layout_with_kk,
     vertex.size=ind*1.2, # Knotengröße entspr. des Indegree-Werts und mit Faktor 1,2 skaliert
     vertex.label.color="black",
     vertex.label.cex=.2,
     vertex.label.family="sans",
     vertex.frame.color="white",
     edge.arrow.size=.001,
     edge.arrow.color="white",
     asp=0)

# Anzeigen der Degree-Verteilung & Sortierung: Wie viele Personen haben eine Verbindung zu den einzelnen Organisationen?
ind
sort(ind)
```

### Clusteranalyse: Mitgliedernetzwerk 2022

```{r Cluster / Communities & Dichte & Modularität}

clusters(mem22)
  # Anzahl der Cluster: 102 - Größe der Cluster von 2 bis maximal 64 - bis auf ein Cluster mit 14 und einem mit 64 Knoten umfassen die meisten Cluster zwischen 2 und 4 Knoten, vereinzelt nur 5 bis 7

clg_mem22 <- cluster_walktrap(mem22)

membership(clg_mem22) # Aufteilung der Knoten in Communities
communities(clg_mem22)

# Wie viele Komponenten?
components(mem22)

# Modularität
modularity(clg_mem22) # 0,9505442

# Dichte
edge_density(mem22) # 0.002127255 (= 0,21 %)
```
```{r Visualisierung der Cluster (2022)}

clg_mem22
par(mfrow=c(1,1), mar=c(0,0,1,2))

#Visualisierung der Cluster
plot(clg_mem22, 
     mem22, 
     main="Clusteranalyse der #30u30-Jahrgänge 2017 bis 2021",
     sub="nur Stationen im Jahr 2022",
     edge.curved=curve_multiple(mem22),
     vertex.frame.color="white",
     edge.arrow.size=.0002, 
     edge.arrow.color="white", 
     edge.curved=.2,
     vertex.size=3,
     vertex.label.degree=0, 
     vertex.label.cex=.15,
     vertex.label.family="sans",
     vertex.label.font=2,
     vertex.label.color="black")

# Plot / Clusteranalyse einmal ohne Vertex-Labels
plot(clg_mem22, 
     mem22, 
     main="Clusteranalyse der #30u30-Jahrgänge 2017 bis 2021",
     sub="nur Stationen im Jahr 2022",
     edge.curved=curve_multiple(mem22),
     vertex.frame.color="white",
     edge.arrow.size=.0002, 
     edge.arrow.color="white",
     edge.color="grey",
     edge.curved=.2,
     vertex.size=2,
     vertex.label=NA)

```

# Teilnetzwerk der Frauen & Männer

Frauen sind zwar in der Überzahl in den fünf Jahrgängen, ein Vergleich ist somit begrenzt aussagekräftig, dennoch werden die Teilnetzwerke der Geschlechter an dieser Stelle betrachtet.

## Frauen in den #30u30-Jahrgängen 2017 bis 2021

```{r Frauen-Teilnetzwerk erstellen}

g

women <- delete.vertices(g, V(g)[sex == 2]) # löscht die Männer

women_mem <- delete.vertices(women, V(women)[mentor == 2]) # löscht Mentor*innen

women_connected <- delete.vertices(women_mem, V(women_mem)[degree(women_mem, mode="all")=="0"]) # löscht isolierte Knoten

# Löscht mehrfache Kanten / Kantenvereinfachung
women_simple <- simplify(women_connected, remove.multiple=TRUE)
women_simple

# Generiert den Plot des Frauennetzwerks der #30u30-Jahrgänge 2017 bis 2021
plot(women_simple,
     layout=layout_with_kk,
     edge.curved=curve_multiple(women_simple), # verhindert, dass sich Kanten überlagern
     vertex.frame.color=NA,
     edge.curved=.2,
     vertex.size=1, 
     vertex.label.cex=.15,
     vertex.label.family="sans",
     vertex.label.font=2,
     vertex.label.color="black",
     edge.arrow.size=.00001,
     egde.arrow.color="white",
     main="Netzwerk der Frauen in den #30u30-Jahrgängen 2017 - 2021",
     sub="Anzahl der Frauen: n = 102",
     asp=0)
```

Berechne die Dichte im Frauennetzwerk

```{r Frauennetzwerk: Dichte}

# Dichte
edge_density(women_simple) # 0,001756047 (0,175 %)

```

Berechne mittlere Pfaddistanz und die weiteste Distanz zwischen zwei Knoten.

```{r Frauennetzwerk: Pfaddistanz & Diameter}

# Mittlere Pfaddistanz
mean_distance(women_simple, directed=FALSE) # 6,005449

# Durchmesser des Netzwerks?
diameter(women_simple, directed=FALSE) # 12

# Welche zwei Knoten sind am weitesten entfernt?
farthest_vertices(women_simple, directed=FALSE)
# 3M & APCO Worldwide
```

Wie viele Organisationen sind im Frauennetzwerk enthalten?

```{r Frauennetzwerk: Anzahl der Organisationen}

fem_org <- V(women_simple)$type == 2
sum(fem_org, na.rm=T) # Summe: 606

```
Wie viele Verbindungen hat jede Frau im Durchschnitt?

```{r Frauennetzwerk: Mittlerer Outdegree-Wert}

# Berechne Mittelwert des Outdegree-Werts der #30u30-Frauen
mean(degree(women_simple,v=V(women_simple)[(mentor==1) & (sex==1)]), mode="out") # 8,627451
```

Welche sind die populärsten Organisationen im Frauennetzwerk?
Hierzu erstellen wir einen Plot, in dem die Knotengröße dem indegree-Wert entsprechend groß dargestellt wird.

```{r Frauennetzwerk: Populärste Organisationen}

ind <- degree(women_simple, mode="in")

# Visualisierung des Frauennetzwerks (Knotengröße entspr. Indegree-Wert und  mit Faktor 1,2 skaliert)

plot(women_simple,
     layout=layout_with_kk,
     edge.curved=curve_multiple(women_simple), # verhindert, dass sich Kanten überlagern
     vertex.frame.color="white",
     edge.curved=.2,
     vertex.size=ind*1.2, 
     vertex.label=NA,
     vertex.label.family="sans",
     vertex.label.font=2,
     vertex.label.color="black",
     edge.arrow.size=.00001,
     egde.arrow.color="white",
     main="Netzwerk der Frauen der #30u30-Jahrgänge 2017 - 2021",
     sub="Anzahl der Frauen: n = 102",
     asp=0)
```

Auflistung der Indegree-Werte im Frauennetzwerk

```{r indegrees Frauen}
ind_fem <- degree(women_simple, mode="in")
ind_fem
sort(ind_fem, decreasing=T)

```

Outdegree-Verteilung: Wie viele Beziehungen haben die Frauen jeweils?
```{r outdegrees Frauen}

# Berechnet und sortiert die Outdegree-Werte der weiblichen Mitglieder von #30u30
outd_fem <- degree(women_simple, mode="out")
outd_fem
sort(outd_fem, decreasing = T)

```

Aus wie vielen Clusters und Communities besteht das Frauennetzwerk?
```{r Cluster Frauennetzwerk}

clusters(women_simple)

# Anzahl der Cluster: 4, Größe / $csize [1] 688; 10; 6;  4

clg_fem <- cluster_walktrap(women_simple) 

membership(clg_fem) # gibt die Aufteilung der Knoten in Communities an
communities(clg_fem)

# Modularität
modularity(clg_fem) # 0,6922213

```

Erzeuge das Mentoring-Netzwerk der Frauen

```{r TN: Frauen und Mentor:innen}

g

mentors <- delete.vertices(g, V(g)[type == 2]) # selektiert nur #30u30-Mitglieder und Mentor*innen

women_mentors <- delete.vertices(mentors, V(mentors)[(mentor == 1) & (sex == 2)]) # löscht die Männer von #30u30, sodass nur die Frauen und ihre Mentor:innen übrig bleiben

women_mentoring <- subgraph.edges(women_mentors, E(women_mentors)[relationship == 17]) # selektiert nur Mentoring-Beziehungen

# Färbt die Knoten unterschiedlich ein, um Mentor:innen nach ihrem Geschlecht und von #30u30-Frauen zu unterscheiden

V(women_mentoring)[V(women_mentoring)$sex == 1]$color <- "lightblue" # weibliche Mentorinnen blau
V(women_mentoring)[V(women_mentoring)$sex == 2]$color <- "pink" # Mentoren rosa färben
V(women_mentoring)[V(women_mentoring)$mentor == 1]$color <- "firebrick" # 30u30 Frauen rot hervorheben

women_mentoring

# Erzeugt den Plot
plot(women_mentoring,
     #layout = layout_with_kk,
     vertex.size = 2,
     vertex.label.cex = 0.3,
     vertex.label.color = "black",
     vertex.label.family = "sans",
     edge.arrow.color="white",
     edge.arrow.size=.0000001,
     main = "Mentoring-Netzwerk der Frauen der #30u30-Jahrgänge 2017 bis 2021",
     sub = "rot = #30u30-Mitglieder, blau = Mentorinnen, rosa = Mentoren",
     asp = 0)
```

Wie viele Frauen hatten eine:n Mentor:in?

```{r Anteil der Frauen mit Mentor:in}
women30u30 <- V(women_mentoring)$mentor == 1
sum(women30u30, na.rm=T)
  # Anzahl: 46 (von 102 Frauen) - knapp die Hälfte (45,1 %) hat eine   Mentor:in genannt

# Prozentualer Anteil
(46/102)*100 # Anteil der Frauen mit Mentor:in an allen Frauen: 45,09 %

```
Wie viele Mentor:innen hatten die Frauen insgesamt?

```{r Anzahl der Mentor:innen der Frauen}

mentoren_fem <- V(women_mentoring)$mentor == 2
sum(mentoren_fem, na.rm=T) # Ergebnis: 109 Mentor:innen

```
Wie ist die Geschlechterverteilung der Mentor:innen der Frauen?

```{r Frauennetzwerk: Geschlechterverteilung der Mentor:innen}

mentors_fem_sex <- delete.vertices (women_mentoring, V(women_mentoring)[mentor == 1]) # selektiert Mentor*innen im Frauennetzwerk
mentoren_fem <- V(mentors_fem_sex)$sex == 1
sum(mentoren_fem, na.rm=T) # zählt wie viele der Mentor:innen Frauen sind
  # 61 / 109 Mentor:innen sind Frauen

# Wie viel Prozent sind das?
(61/109)*100  # entspr. 55,96 %
```

Wie viele Mentor*innen hat jede Frau im Schnitt? (Unter Berücksichtigung, dass knapp die Hälfte der Frauen von #30u30 keine:n Mentor:in genannt hatte)

```{r Durchschnittliche Anzahl der Mentor:innen pro #30u30-Frau}

mean(degree(women_mentoring,v=V(women_mentoring)[mentor==1]))
  # durchschnittlich 2,4565 - die Frauen mit Mentor:innen hatten im Schnitt 2,45 Mentor:innen - also mehr als 2

```
## Männer in den #30u30-Jahrgängen 2017 bis 2021

```{r Teilnetzwerk der Männer erstellen}

g

men <- delete.vertices(g, V(g)[sex == 1]) # löscht die Frauen

men_mem <- delete.vertices(men, V(men)[mentor == 2]) # löscht Mentor:innen

men_connected <- delete.vertices(men_mem, V(men_mem)[degree(men_mem, mode="all")=="0"]) # löscht isolierte Knoten

men_simple <- simplify(men_connected, remove.multiple=TRUE) # Kantenvereinfachung
men_simple

# Generiert den Plot des Netzwerks der Männer bei #30u30
plot(men_simple,
     layout=layout_with_kk,
     edge.curved=curve_multiple(men_simple), # verhindert, dass sich Kanten überlagern
     vertex.frame.color=NA,
     edge.curved=.2,
     vertex.size=1, 
     edge.arrow.size=.0000001,
     edge.arrow.color="white",
     vertex.label.cex=.15,
     vertex.label.family="sans",
     vertex.label.font=2,
     vertex.label.color="black",
     main="Netzwerk der Männer der #30u30-Jahrgänge 2017 bis 2021",
     sub="Anzahl der Männer: n = 48",
     asp=0)

```

Dichte im Männernetzwerk:

```{r Männernetzwerk: Dichte}

edge_density(men_simple) # 0,002798222 (= 0,28 %)
```

Mittlere Pfaddistanz, Durchmesser und am weitesten entfernte Knoten im Männernetzwerk:

```{r Männernetzwerk: Pfaddistanz & Diameter}

mean_distance(men_simple, directed=FALSE) # 6,768176

diameter(men_simple, directed=FALSE) # 14

# Welche Knoten sind am weitesten voneinander entfernt?
farthest_vertices(men_simple, directed=FALSE) #30x Friends & BrandFuel LTD

```

Wie viele Stationen hat jeder Mann im Schnitt?

```{r Männernetzwerk: Mittlerer Outdegree-Wert}

mean(degree(men_simple,v=V(men_simple)[(mentor==1) & (sex==2)]), mode="out")
  # 8.627451

```

Welche sind die populärsten Organisationen im Männernetzwerk?
Hierzu erstellen wir einen Plot, in dem die Knotengröße dem indegree-Wert entsprechend groß dargestellt wird.

```{r Populärste Organisationen - Männer}

ind_men <- degree(men_simple, mode="in")

# Visualisierung, Knotengröße entspr. des Indegree-Werts und mit Faktor 1,2 skaliert

plot(men_simple,
     layout=layout_with_kk,
     edge.curved=curve_multiple(men_simple), # verhindert, dass sich Kanten überlagern
     vertex.frame.color="white",
     edge.curved=.2,
     vertex.size=ind_men*1.2, 
     vertex.label=NA,
     vertex.label.family="sans",
     vertex.label.font=2,
     vertex.label.color="black",
     edge.arrow.size=.00001,
     egde.arrow.color="white",
     main="Netzwerk der Männer der #30u30-Jahrgänge 2017 bis 2021",
     sub="Anzahl der Männer: n = 48"
     asp=0)
```

Auflistung der Indegree- und Outdegree-Werte im Netzwerk der #30u30-Männer: Welche Organisationen sind populär und wie viele Verbindungen haben die Männer jeweils?

```{r indegree-Verteilung Frauennetzwerk}
ind_male <- degree(men_simple, mode="in")
ind_male
sort(ind_male)
```

```{r outdegree-Verteilung Männernetzwerk}

# Berechnet und sortiert die Outdegree-Werte der männlichen Mitglieder von #30u30
outd_men <- degree(men_simple, mode="out")
outd_men
sort(outd_men)

```

Erzeugt das Mentoring-Netzwerk der Männer

```{r TN: Männer und Mentor:innen}

g
mentors
men_mentors <- delete.vertices(mentors, V(mentors)[(mentor == 1) & (sex == 1)]) # selektiert nur Männer bei 30u30 und Mentor:innen

men_mentoring <- subgraph.edges(men_mentors, E(men_mentors)[relationship == 17]) # selektiert Mentoring-Bzgh.

# Färbt Knoten so ein, dass Mentor:innen anhand ihres Geschlechts unterscheidbar und von #30u30-Männern abgegrenzt werden
V(men_mentoring)[V(men_mentoring)$sex == 1]$color <- "lightblue" # weibliche Mentorinnen blau
V(men_mentoring)[V(men_mentoring)$sex == 2]$color <- "pink" # Mentoren rosa färben
V(men_mentoring)[V(men_mentoring)$mentor == 1]$color <- "firebrick" # 30u30 Männer rot hervorheben

men_mentoring

# Generiert den Plot
plot(men_mentoring,
     #layout = layout_with_kk,
     edge.curved=curve_multiple(men_mentoring),
     vertex.size = 2,
     edge.arrow.size=0.000001,
     edge.arrow.color="white",
     vertex.label.cex = 0.3,
     vertex.label.color = "black",
     vertex.label.family = "sans",
     main = "Männer der #30u30-Jahrgänge 2017 bis 2021 und ihre Mentor:innen",
     sub = "rot = #30u30-Mitglieder, blau = Mentorinnen, rosa = Mentoren",
     asp = 0)
```

Wie viele Männer haben eine:n Mentor:in genannt?

```{r Anzahl der Männer mit Mentor:in}

men30u30 <- V(men_mentoring)$mentor == 1
sum(men30u30, na.rm=T)
  # Anzahl: 24 (von 48 Männern) - genau die Hälfte hat keine:n Mentor:in genannt

```

Wie viele Mentor:innen hatten die Männer insgesamt?

```{r Anzahl der Mentor:innen im Männernetzwerk}

mentoren_mas <- V(men_mentoring)$mentor == 2
sum(mentoren_mas, na.rm=T) # 53 Mentor:innen im Netzwerk der #30u30-Männer

```

Wie ist die Geschlechterverteilung unter den Mentor:innen der Männer?

```{r Männernetzwerk: Geschlechterverteilung der Mentor:innen}

mentors_mas_sex <- delete.vertices (men_mentoring, V(men_mentoring)[mentor == 1]) # selektiert Mentor*innen im Männernetzwerk

mentoren_mas <- V(mentors_mas_sex)$sex == 1 # wie viele weibliche Mentor*innen?
sum(mentoren_mas, na.rm=T)

# 12 der 53 Mentor*innen sind Frauen -> klarer Unterschied zum Frauennetzwerk, bei denen mehr als 50 % weibliche Mentor*innen waren

# Das entspricht einem Prozentwert von 22,64%:
(12/53)*100

```

Wie viele Mentor:innen hat jeder Mann im Schnitt? (Unter Berücksichtigung: Die Hälfte der Männer von #30u30 hatten keine:n Mentor:in genannt)

```{r Durchschnittliche Zahl der Mentor:innen pro Mann}

mean(degree(men_mentoring,v=V(men_mentoring)[mentor==1]))
# Antwort: Die Männer mit Mentor:in hatten im Schnitt knapp über 2 Mentor:innen (2,25) --> Leicht unter der Zahl der Frauen

```

# Die einzelnen Jahrgänge
Im folgenden werden die fünf Jahrgänge separat analysiert.

## Jahrgang 2017

Nachfolgend werden die einzelnen Jahrgänge in den Blick genommen, angefangen mit der Crew 2017.

### Gesamtnetzwerk des Jahrgangs 2017

Zunächst wird das Teilnetzwerk des Jahrgangs 2017 erstellt. Dies beinhaltet alle Stationen / Verbindungen der Mitglieder bis zum März 2022.

```{r Teilnetzwerk des Jahrgangs 2017}
members # ruft das Gesamtnetzwerk der #30u30 Mitglieder auf

jg17 <- delete.vertices(members, V(members)[entry > 2017]) # selektiert aus dem Gesamtnetzwerk nur die #30u30-Mitglieder, die 2017 aufgenommen wurden (Knoten mit entry = 99 - Organisationen bleiben ebenfalls erhalten, da wir deren Verbindungen zum Jahrgang abbilden wollen)

jg17
jg17_1 <- delete_vertices(jg17, V(jg17)[degree(jg17, mode="all")==0]) # entfernt alle isolierten Knoten

#Generiert eine erste Visualisierung des Jahrgangs 2017
plot(jg17_1,
     layout=layout_with_kk,
     edge.curved=curve_multiple(jg17_1), # verhindert, dass sich Kanten überlagern
     vertex.frame.color="white",
     edge.color="grey",
     edge.curved=.2,
     vertex.size=3,
     vertex.label.degree=0, 
     vertex.label.cex=.15,
     vertex.label.family="sans",
     vertex.label.font=2,
     vertex.label.color="black",
     main="Netzwerk der Crew 2017 der #30u30",
     rescale=T,
     asp=0)
```

Damit mehrjährige Beziehungen nur einfach angezeigt werden, werden multiple Kanten über simplify entfernt und erneut geplottet.

```{r Teilnetzwerk Jahrgang 2017 - Kanten-Vereinfachung}
jg17_1
jg17_2 <- simplify(jg17_1, remove.multiple=TRUE)

# Generiert den Plot des Netzwerks des Jahrgangs 2017, Skalierung der Knoten entspr. des Indegree-Werts
jg17_2
plot(jg17_2,
     layout=layout_with_kk,
     edge.curved=curve_multiple(jg17_2), # verhindert, dass sich Kanten überlagern
     vertex.frame.color=NA,
     edge.curved=.2,
     vertex.size=degree(jg17_2)*1.2, # Knoten werden entsprechend ihres Degree-Wert skaliert
     vertex.label.cex=.15,
     vertex.label.family="sans",
     vertex.label.font=2,
     vertex.label.color="black",
     edge.arrow.size=0.001,
     edge.arrow.color="white",
     main="Netzwerk der Crew 2017 der #30u30",
     sub="Knotengröße entspr. des Indegree-Werts mit Faktor 1,2 skaliert",
     asp=0)
```

Wie viele Organisationen sind im Teilnetzwerk des Jahrgang 2017 enthalten?

```{r Teilnetzwerk Jahrgang 2017 - Anzahl Organisationen}
jg17_org <- V(jg17_1)$type == 2
sum(jg17_org, na.rm=T) # 227 Organisationen 
```
Mit wie vielen Mitgliedern stehen die Organisationen jeweils in Verbindung? Welche sind besonders populär ?

```{r Jahrgang 2017 - Indegree-Verteilung}
jg17_ind <- degree(jg17_2, mode="in")
jg17_ind
sort(jg17_ind, decreasing=T)
```

Wie viele Verbindungen haben die Mitglieder? Wer ist besonders stark vernetzt?

```{r Jahrgang 2017 - Outdegree-Verteilung}
jg17_outd<- degree(jg17_2, mode="out")
jg17_outd
sort(jg17_outd, decreasing = T)
```

Im Folgenden wird ein Teilnetzwerk der Personen erstellt, die bei den populärsten Organisationen beschäftigt waren, um zu prüfen, ob diese auch weitere Parallelen aufweisen.

```{r Teilnetzwerk Jahrgang 2017 - Populärste Organisationen}
rfuhrmann <- subgraph <- make_ego_graph(members, order = 1,  c("Rasmus Fuhrmann"))
rweiss <- subgraph <- make_ego_graph(members, order = 1,  c("Rene Weiss"))
snicolai <- subgraph <- make_ego_graph(members, order = 1,  c("Susanne Nicolai"))
fsievers <- subgraph <- make_ego_graph(members, order = 1,  c("Felix Sievers"))
lbisswanger <- subgraph <- make_ego_graph(members, order = 1,  c("Luisa Bisswanger"))
shoyningen <- subgraph <- make_ego_graph(members, order = 1,  c("Selina von Hoyningen-Huene"))
mschaeffer <- subgraph <- make_ego_graph(members, order = 1,  c("Marina Schäffer"))
efriese <- subgraph <- make_ego_graph(members, order = 1,  c("Eva-Maria Friese"))
jkoester <- subgraph <- make_ego_graph(members, order = 1,  c("Julia Köster"))
shelmhold <- subgraph <- make_ego_graph(members, order = 1,  c("Sarah Helmhold-Bünte"))
aplanz <- subgraph <- make_ego_graph(members, order = 1,  c("Anna Planz"))
ydoepke <- subgraph <- make_ego_graph(members, order = 1,  c("Yannik Döpke"))
mblokhina <- subgraph <- make_ego_graph(members, order = 1,  c("Marina Blokhina"))
sheitzler <- subgraph <- make_ego_graph(members, order = 1,  c("Sophia Heitzler"))
pwolter <- subgraph <- make_ego_graph(members, order = 1,  c("Paul Wolter"))

# Ego-Netzwerke vom Gesamtnetzwerk doppelt subtrahieren (= addieren):

jg17_pop <- members - (members - rweiss[[1]] - rfuhrmann[[1]] - snicolai[[1]] - fsievers[[1]] - lbisswanger[[1]] - shoyningen[[1]] - mschaeffer[[1]] - efriese[[1]] - jkoester[[1]] - shelmhold[[1]] - aplanz[[1]] - ydoepke[[1]] - mblokhina[[1]] - sheitzler[[1]] - pwolter[[1]]) 

jg17_pop

jg17_pop1 <- delete_vertices (jg17_pop, V(jg17_pop)[degree(jg17_pop, mode="all")=="0"]) # Isolierte Nodes ohne Beziehungen löschen

ind_jg17pop <- degree(jg17_pop1, mode="in") 

jg17_pop2 <- simplify(jg17_pop1, remove.multiple = TRUE) # Kantenvereinfachung

# Erstellt den Plot des Subgraph
plot(jg17_pop2,
     layout=layout_with_kk,
     edge.curved=curve_multiple(jg17_pop2), # verhindert, dass sich Kanten überlagern
     vertex.frame.color="white",
     edge.arrow.size=.0001,
     edge.arrow.color="white",
     vertex.size=ind_jg17pop*1.2, # Knoten entspr. des Indegree-Werts und mit Faktor 1,2 skaliert
     edge.color="grey",
     edge.curved=.2,
     vertex.label.degree=0, 
     vertex.label.cex=.15,
     vertex.label.family="sans",
     vertex.label.font=2,
     vertex.label.color="black",
     main="Die populärsten Organisationen im Jahrgang 2017
     und die damit verbundenen Personen",
     asp=0)
```

Clusteranalyse des Teilnetzwerks des Jahrgangs 2017 

```{r Teilnetzwerk Jahrgang 2017 - Clusteranalyse}
jg17_2
gc_jg17 <- cluster_walktrap(jg17_2)

# Berechne Modularität
modularity(gc_jg17) # 0,8778479

membership(gc_jg17)
par(mfrow=c(1,1), mar=c(0,0,1,2))

# Abfrage der Components
components(jg17_2)

#Visualisierung der Cluster
plot(gc_jg17, 
     jg17_2, 
     edge.arrow.size=0.0001,
     edge.arrow.color="white",
     main="Clusteranalyse des #30u30-Jahrgangs 2017",
     sub="Alle Verbindungen bis März 2022",
     edge.curved=curve_multiple(jg17_2),
     vertex.frame.color="white",
     edge.color="grey",
     edge.curved=.2,
     vertex.size=5,
     vertex.label.degree=0, 
     vertex.label.cex=.15,
     vertex.label.family="sans",
     vertex.label.font=2,
     vertex.label.color="black")

# Visualisierung einmal ohne Vertex-Labels
plot(gc_jg17, 
     jg17_2, 
     main="Clusteranalyse des #30u30-Jahrgangs 2017",
     sub="Alle Verbindungen bis März 2022",
     edge.curved=curve_multiple(jg17_2),
     vertex.frame.color="white",
     edge.arrow.size=.00002, 
     edge.arrow.color="white",
     edge.color="grey",
     edge.curved=.2,
     vertex.size=5,
     vertex.label=NA)
```

Wie hoch ist die Dichte im Teilnetzwerk des Jahrgangs 2017?

```{r Jahrgang 2017 - Dichte}
edge_density(jg17_2) # 0,003799854 (= 0,379 %)
```

Mittlere Pfaddistanz, Durchmesser & die am weitesten entfernten Knoten

```{r Jahrgang 2017 - Pfaddistanz}
# Berechne Mittlere Pfaddistanz
mean_distance(jg17_2, directed=FALSE) # 6,182995

# Berechne Durchmesser des Teilnetzwerks
diameter(jg17_2, directed=FALSE)# Längster Pfad: 16 Schritte

# Welche Knoten sind am weitesten voneinander entfernt?
farthest_vertices(jg17_2, directed = FALSE) #arsedition & HS Darmstadt (16 Schritte)
```

Betwenness-Werte im Teilnetzwerk des Jahrgangs 2017: Wer sind die Broker?

```{r Jahrgang 2017 - Betweenness}
betweenness(jg17_2)
centr_betw(jg17_2, directed=TRUE)
betw17 <- betweenness(jg17_2)
sort(betw17)

V(jg17_2)[mentor==1]$color <- "firebrick1"

# Wir können die Ausgabe von betweenness()  einer Variablen im Netz zuordnen und die Knoten entsprechend dimensionieren.
V(jg17_2)$betweenness <- betweenness(jg17_2)

plot(jg17_2, 
     vertex.label.cex = .6, 
     vertex.label.color = "black", 
     vertex.size = V(jg17_2)$betweenness) # Größe angepasst an betweenness

# Da die  Betweenness-Zentralität sehr groß sein kann, wird sie im folgenden normalisiert, indem sie durch das Maximum geteilt und mit einem Skalar multipliziert wird, bevor man sie plottet. So erkennt man Broker besser.
plot(jg17_2,
     layout=layout_with_kk,
     vertex.label.cex = .15, 
     vertex.label.color = "black", 
     vertex.label.family="sans",
     edge.arrow.size=.00001,
     edge.arrow.color="white",
     asp=0,
     vertex.size = V(jg17_2)$betweenness/max(V(jg17_2)$betweenness) * 10,
     main="Broker im Netzwerk des #30u30-Jahrgangs 2017",
     sub="Alle Stationen bis März 2022
     Knotengröße entspr. Betweenness-Wert und mit Faktor 10 skaliert")
```

### Jahrgang 2017 - Stationen im Jahr der Aufnahme
Nun werden nur die Beziehungen selektiert, die in dem Jahr der Aufnahme bei #30u30 bestanden, um zu betrachten, ob es zu diesem Zeitpunkt Überschneidungen gab.

```{r Jahrgang 2017 - Aufnahmejahr - Plot}
jg17_aufnahme <- subgraph.edges(jg17, E(jg17)[year == 2017]) # selektiert die Beziehungen der Mitglieder des Jahrgang 2017 aus dem Jahr 2017

jg17_aufnahme

jg17_aufnahme1 <- simplify(jg17_aufnahme, remove.multiple = TRUE) # entfernt doppelte Kanten

jg17_aufnahme1

# Generiert den Plot
plot(jg17_aufnahme1,
     layout=layout_with_kk,
     edge.curved=curve_multiple(jg17_aufnahme1), # verhindert, dass sich Kanten überlagern
     vertex.frame.color=NA,
     edge.curved=.2,
     edge.color="grey80",
     vertex.size=5, 
     vertex.label.cex=.2,
     vertex.label.family="sans",
     vertex.label.font=2,
     vertex.label.color="black",
     main="Netzwerk der Crew 2017 der #30u30 im Aufnahmejahr 2017",
     edge.arrow.size=0.0001,
     edge.arrow.color="white",
     asp=0)
```

Wie hoch war die Dichte im Aufnahmejahr?

```{r Jahrgang 2017 - Aufnahmejahr - Dichte}
edge_density(jg17_aufnahme1) #0,008276534 (0,82 %)
```

Clusteranalyse des Netzwerks im Aufnahmejahr

```{r Jahrgang 2017 - Aufnahmejahr - Clusteranalyse}

jg17_aufnahme1
gc_jg17a <- cluster_walktrap(jg17_aufnahme1)

# Berechne Modularität
modularity(gc_jg17a) # 0,9534794

membership(gc_jg17a)
par(mfrow=c(1,1), mar=c(0,0,1,2))

# Listet die Components auf
components(jg17_aufnahme1)

#Visualisierung der Cluster
plot(gc_jg17a, 
     jg17_aufnahme1, 
     edge.arrow.size=0.0001,
     edge.arrow.color="white",
     main="Clusteranalyse des #30u30-Jahrgangs 2017",
     sub="Stationen im Aufnahmejahr (2017)",
     edge.curved=curve_multiple(jg17_aufnahme1),
     vertex.frame.color="white",
     edge.color="grey",
     edge.curved=.2,
     vertex.size=5,
     vertex.label.degree=0, 
     vertex.label.cex=.15,
     vertex.label.family="sans",
     vertex.label.font=2,
     vertex.label.color="black")
```

### Jahrgang 2017 - Alle Stationen bis zum Aufnahmejahr
Zuletzt wird das Netzwerk der Jahrgangsmitglieder betrachtet mit all ihren Verbindungen bis zum Jahr der Aufnahme, um die Überschneidungen bis dahin zu beleuchten.

Zunächst wird das entsprechende Teilnetzwerk erstellt.

```{r Jahrgang 2017 - Bis Aufnahme - Plot}
jg17_1

jg17_preentry <- subgraph.edges(jg17_1, E(jg17_1)[year <= 2017]) # selektiert alle Beziehungen, die bis einschließlich 2017 bestanden

jg17_preentry
jg17_preentry1 <- simplify(jg17_preentry, remove.multiple = TRUE) # entfernt doppelte Kanten

jg17_preentry1

# Generiert einen Plot des Netzwerks - ohne Degree-Visualisierung
plot(jg17_preentry1,
     layout=layout_with_kk,
     edge.curved=curve_multiple(jg17_preentry1), # verhindert, dass sich Kanten überlagern
     vertex.frame.color=NA,
     edge.curved=.2,
     edge.color="grey80",
     edge.arrow.size=.0000001,
     edge.arrow.color="white",
     vertex.size=2, 
     vertex.label.cex=.15,
     vertex.label.family="sans",
     vertex.label.font=2,
     vertex.label.color="black",
     main="Netzwerk der Crew 2017 der #30u30",
     sub="Stationen bis zum Aufnahmejahr",
     asp=0)

ind_jg17_pre <- degree(jg17_preentry1, mode="in")

# Generiert einen Plot (Knotengröße entspr. Indegree-Wert und mit Faktor 1,2 skaliert)
plot(jg17_preentry1,
     layout=layout_with_kk,
     edge.curved=curve_multiple(jg17_preentry1), # verhindert, dass sich Kanten überlagern
     vertex.frame.color=NA,
     edge.curved=.2,
     edge.color="grey80",
     vertex.size=ind_jg17_pre*1.2, 
     vertex.label.cex=.15,
     vertex.label.family="sans",
     vertex.label.font=2,
     vertex.label.color="black",
     edge.arrow.size=.000001,
     edge.arrow.color="white",
     main="Netzwerk der Crew 2017 der #30u30",
     sub="Stationen bis zum Jahr der Aufnahme (2017)",
     asp=0)
```

Wie viele Organisationen enthält das Netzwerk bis zum Aufnahmejahr?

```{r Jahrgang 2017 - Bis Aufnahme - Anzahl der Organisationen}
jg17_org_pe <- V(jg17_preentry1)$type == 2
sum(jg17_org_pe, na.rm=T) # 192 Organisationen
```

Wie hoch war die Dichte im Netzwerk bis zum Aufnahmejahr?

```{r Jahrgang 2017 - Bis Aufnahme - Dichte}
edge_density(jg17_preentry1) # 0,004300681 (0,4 %)
```

Wie hoch sind die mittlere Pfaddistanz und der Durchmesser?

```{r Jahrgang 2017 - Bis Aufnahme - Pfaddistanz & Diameter}
# Mittlere Pfaddistanz
mean_distance(jg17_preentry1, directed = FALSE) # 6,004976

# Längster Pfad / Diameter
diameter(jg17_preentry1, directed=FALSE) # 16
farthest_vertices(jg17_preentry1, directed=FALSE) # Arsedition, Hochschule Darmstadt
```

Welche Organisationen waren besonders populär und verbinden viele Talente vor ihrer Aufnahme?

```{r Jahrgang 2017 - Bis Aufnahme - Indegree-Verteilung}

jg17_ind_auf <- degree(jg17_preentry1, mode="in")
jg17_ind_auf
sort(jg17_ind_auf)
```

Welche #30u30-Mitglieder sind bis zur Aufnahme besonders stark vernetzt?

```{r Jahrgang 2017 - Bis Aufnahme - Outdegree-Verteilung}

jg17_outd_pe <- degree(jg17_preentry1, mode="out")
jg17_outd_pe
sort(jg17_outd_pe)

```

Clusteranalyse im Netzwerk bis zur Aufnahme

```{r Jahrgang 2017 - Bis Aufnahme - Clusteranalyse}

jg17_preentry1
gc_jg17pe <- cluster_walktrap(jg17_preentry1)

# Berechne Modularität
modularity(gc_jg17pe) # 0,893174

membership(gc_jg17pe)
par(mfrow=c(1,1), mar=c(0,0,1,2))

#Zeigt (Anzahl und Bestandteile der) Components an
components(jg17_preentry1)

#Visualisierung der Cluster
plot(gc_jg17pe, 
     jg17_preentry1, 
     edge.arrow.size=0.0001,
     edge.arrow.color="white",
     main="Clusteranalyse des #30u30-Jahrgangs 2017",
     sub="Stationen bis zum Aufnahmejahr (2017)",
     edge.curved=curve_multiple(jg17_preentry1),
     vertex.frame.color="white",
     edge.arrow.size=.2, 
     edge.color="grey",
     edge.curved=.2,
     vertex.size=5,
     vertex.label.degree=0, 
     vertex.label.cex=.15,
     vertex.label.family="sans",
     vertex.label.font=2,
     vertex.label.color="black")

```

Betweenness-Werte: Wer waren die Broker im Netzwerk des Jahrgangs 0217 bis zum Jahr der Aufnahme? 

```{r Jahrgang 2017 - Bis Aufnahme - Betweenness}
betweenness(jg17_preentry1)
centr_betw(jg17_preentry1, directed=TRUE)
betw17pe <- betweenness(jg17_preentry1)
sort(betw17pe)

V(jg17_preentry1)[mentor==1]$color <- "firebrick1" # färbt #30u30 rot

  # Wir können die Ausgabe von betweenness()  einer Variablen im Netz zuordnen und die Knoten entsprechend dimensionieren.
V(jg17_preentry1)$betweenness <- betweenness(jg17_preentry1)

plot(jg17_preentry1, 
     vertex.label.cex = .6, 
     vertex.label.color = "black", 
     vertex.size = V(jg17_preentry1)$betweenness) # Größe angepasst an betweenness

# Da die  Betweenness-Zentralität sehr groß sein kann, wird sie im folgenden normalisiert, indem man sie durch das Maximum teilt und mit einem Skalar multipliziert, bevor man sie plottet. So erkennen wir die Broker besser.
plot(jg17_preentry1,
     layout=layout_with_kk,
     vertex.label.cex = .15, 
     vertex.label.color = "black", 
     vertex.label.family="sans",
     edge.arrow.size=.00001,
     edge.arrow.color="white",
     asp=0,
     vertex.size = V(jg17_preentry1)$betweenness/max(V(jg17_preentry1)$betweenness) * 10,
     main="Broker im Netzwerk des #30u30-Jahrgangs 2017",
     sub="Alle Stationen bis zum Aufnahmejahr (2017),
     Knotengröße entspr. dem Betweenness-Wert und mit Faktor 10 skaliert")
```

## Jahrgang 2018

Nun liegt der Fokus auf den Mitgliedern des Jahrgangs 2018.

### Gesamtnetzwerk Jahrgang 2018
Nun wird das Teilnetzwerk des Jahrgangs 2018 erstellt, das alle Stationen / Verbindungen der Mitglieder bis zum März 2022 beinhaltet.

```{r Teilnetzwerk Jahrgang 2018}
members # ruft das Gesamtnetzwerk der #30u30 Mitglieder auf

jg18 <- delete.vertices(members, V(members)[(type == 1) & (entry != 2018)]) # löscht aus dem Gesamtnetzwerk alle #30u30-Mitglieder, die nicht 2018 aufgenommen wurden (Knoten des type 2 bleiben erhalten, da wir deren Verbindungen zum Jahrgang abbilden wollen)

jg18

jg18 <- delete_vertices(jg18, V(jg18)[degree(jg18, mode="all")==0]) # entfernt alle isolierten Knoten

#Generiert eine erste Visualisierung des Jahrgangsnetzwerks 2018
plot(jg18,
     layout=layout_with_kk,
     edge.curved=curve_multiple(jg18), # verhindert, dass sich Kanten überlagern
     vertex.frame.color="white",
     edge.color="grey",
     edge.curved=.2,
     vertex.size=3,
     vertex.label.degree=0, 
     vertex.label.cex=.15,
     vertex.label.family="sans",
     vertex.label.font=2,
     vertex.label.color="black",
     main="Netzwerk der Crew 2018 der #30u30",
     sub="Alle Verbindungen bis März 2022",
     rescale=T,
     asp=0)
```

Damit mehrjährige Beziehungen nur einfach angezeigt werden, werden multiple Kanten über simplify entfernt, die Darstellung der Knoten angepasst und erneut geplottet.

```{r Jahrgang 2018 - Kanten-Vereinfachung & Plot}
jg18
jg18_1 <- simplify(jg18, remove.multiple=TRUE)

# Knoten-Visualisierung
V(jg18_1)[V(jg18_1)$type == 1]$color <- "firebrick1" # Personen werden rot gefärbt
V(jg18_1)[V(jg18_1)$type == 1]$shape <- "circle" # Personen werden als Kreis angezeigt
jg18_1

# Generiert einen schöner visualisierten Plot des Netzwerks des Jahrgangs 2018
plot(jg18_1,
     layout=layout_with_kk,
     edge.curved=curve_multiple(jg18_1), # verhindert, dass sich Kanten überlagern
     vertex.frame.color=NA,
     edge.curved=.2,
     vertex.size=2,
     vertex.label.cex=.15,
     vertex.label.family="sans",
     vertex.label.font=2,
     vertex.label.color="black",
     edge.arrow.size=0.001,
     edge.arrow.color="white",
     main="Netzwerk der Crew 2018 der #30u30",
     sub="Alle Verbindungen bis März 2022",
     asp=0)
```

Wie viele Organisationen sind im Netzwerk des Jahrgangs enthalten?

```{r Jahrgang 2018 - Anzahl der Organisationen}
jg18_org <- V(jg18_1)$type == 2
sum(jg18_org, na.rm=T) # 220 Organisationen 
```
Welche Organisationen sind besonders populär und verbinden viele Talente?

```{r Jahrgang 2018 - Indegree-Verteilung}
jg18_ind <- degree(jg18_1, mode="in")
jg18_ind
sort(jg18_ind)
```

Welche 30u30-Mitglieder des Jahrgangs 2018 sind besonders stark vernetzt?

```{r Jahrgang 2018 - Outdegree-Verteilung}
jg18_outd <- degree(jg18_1, mode="out")
jg18_outd
sort(jg18_outd)
```

Die populärsten Knoten werden im Plot hervorgehoben, indem die Knoten je nach Indegree-Wert größer dargestellt werden.

```{r Jahrgang 2018 - Gesamtnetzwerk - indegree-skaliert}
ind_jg18 <- degree(jg18_1, mode="in")

# Generiert den Plot des Netzwerks des Jahrgangs 2018, die Knoten mit höherem Indegree werden größer dargestellt
plot(jg18_1,
     layout=layout_with_kk,
     edge.curved=curve_multiple(jg18_1), # verhindert, dass sich Kanten überlagern
     vertex.frame.color=NA,
     edge.curved=.2,
     vertex.size=ind_jg18*1.2,
     vertex.label.cex=.15,
     vertex.label.family="sans",
     vertex.label.font=2,
     vertex.label.color="black",
     edge.arrow.size=0.001,
     edge.arrow.color="white",
     main="Netzwerk der Crew 2018 der #30u30",
     sub="Alle Verbindungen bis März 2022, Populäre Knoten mit hohem Indegree-Wert sind größer dargestellt",
     asp=0)
```

Clusteranalyse im Teilnetzwerk des Jahrgangs 2018

```{r Jahrgang 2018 - Clusteranalyse}
# Um die Überschneidungen des Jahrgangs besser zu greifen, folgt eine Clusteranalyse:
jg18_1
gc_jg18 <- cluster_walktrap(jg18_1)
membership(gc_jg18)

# Berechne Modularität
modularity(gc_jg18) # 0,8551368

# Zeigt Components (inkl. Größe und Bestandteilen) an
components(jg18_1)

par(mfrow=c(1,1), mar=c(0,0,1,2))

#Visualisierung der Cluster
plot(gc_jg18, 
     jg18_1, 
     edge.arrow.size=0.0001,
     edge.arrow.color="white",
     main="Clusteranalyse des #30u30-Jahrgangs 2018",
     sub="Alle Stationen bis März 2022",
     edge.curved=curve_multiple(jg18_1),
     vertex.frame.color="white",
     edge.arrow.size=.2, 
     edge.color="grey",
     edge.curved=.2,
     vertex.size=5,
     vertex.label.degree=0, 
     vertex.label.cex=.15,
     vertex.label.family="sans",
     vertex.label.font=2,
     vertex.label.color="black")

#einmal ohne Vertex-Labels
plot(gc_jg18, 
     jg18_1, 
     main="Clusteranalyse des #30u30-Jahrgangs 2018",
     sub="Alle Stationen bis zur Aufnahme",
     edge.curved=curve_multiple(jg18_1),
     vertex.frame.color="white",
     edge.arrow.size=.00002, 
     edge.arrow.color="white",
     edge.color="grey",
     edge.curved=.2,
     vertex.size=5,
     vertex.label=NA)

```

Wie hoch ist die Dichte im Gesamtnetzwerk des Jahrgangs 2018?

```{r Jahrgang 2018 - Dichte}
edge_density(jg18_1) # 0,4080321
#Prozentwert:
edge_density(jg18_1)*100 # 0,408
```
Wie lang sind die mittlere Pfaddistanz und der Durchmesser des Netzwerks des Jahrgangs 2018?

```{r Jahrgang 2018 - Pfaddistanz & Diameter}
#Mittlere Pfaddistanz
mean_distance(jg18_1, directed=FALSE) # 7,009103
# Durchmesser
diameter(jg18_1, directed=FALSE) # Längster Pfad: 16 Schritte
farthest_vertices(jg18_1, directed = FALSE) #Alster Radio und Debattierclub der JGU Mainz (16 Schritte)

```

Betweenness-Werte im Netzwerk des Jahrgangs 2018: Wer sind die Broker?

```{r Jahrgang 2018 - Betweenness}
betweenness(jg18_1)
centr_betw(jg18_1, directed=TRUE)
betw18 <- betweenness(jg18_1)
sort(betw18)

V(jg18_1)[mentor==1]$color <- "firebrick1"

  # Wir können die Ausgabe von betweenness()  einer Variablen im Netz zuordnen und die Knoten entsprechend dimensionieren.
V(jg18_1)$betweenness <- betweenness(jg18_1)

# Visualisierung der Betweenness-Verteilung
plot(jg18_1, 
     vertex.label.cex = .6, 
     vertex.label.color = "black", 
     vertex.size = V(jg18_1)$betweenness) # Größe angepasst an betweenness

# Da die  Betweenness-Zentralität sehr groß sein kann, wird sie im folgenden normalisiert, indem man sie durch das Maximum teilt und mit einem Skalar multipliziert, bevor man sie plottet. So erkennen wir die Broker besser.
plot(jg18_1,
     layout=layout_with_kk,
     vertex.label.cex = .15, 
     vertex.label.color = "black", 
     vertex.label.family="sans",
     edge.arrow.size=.00001,
     edge.arrow.color="white",
     asp=0,
     vertex.size = V(jg18_1)$betweenness/max(V(jg18_3)$betweenness) * 10,
     main="Broker im Netzwerk des #30u30-Jahrgangs 2018",
     sub="Alle Stationen bis März 2022")
```

# Netzwerk des Jahrgangs 2018 - Stationen im Jahr der Aufnahme
Nun werden nur die Beziehungen selektiert, die in dem Jahr der Aufnahme bei #30u30 bestanden, um zu betrachten, ob es zu diesem Zeitpunkt Überschneidungen gab.

```{r Jahrgang 2018 - Aufnahmejahr - Plot}

jg18_aufnahme <- subgraph.edges(jg18, E(jg18)[year == 2018]) # selektiert die Beziehungen aus dem Jahr 2018

jg18_aufnahme

jg18_aufnahme1 <- simplify(jg18_aufnahme, remove.multiple = TRUE) # entfernt doppelte Kanten

jg18_aufnahme1

V(jg18_aufnahme1)[mentor==1]$color <- "firebrick1"

# Generiert den Plot
plot(jg18_aufnahme1,
     layout=layout_with_kk,
     edge.curved=curve_multiple(jg18_aufnahme1), # verhindert, dass sich Kanten überlagern
     vertex.frame.color=NA,
     edge.curved=.2,
     edge.color="grey80",
     vertex.size=5, 
     vertex.label.cex=.2,
     vertex.label.family="sans",
     vertex.label.font=2,
     vertex.label.color="black",
     main="Netzwerk der Crew 2018 der #30u30",
     sub="Nur Verbindungen im Jahr der Aufnahme (2018)",
     edge.arrow.size=0.0001,
     edge.arrow.color="white",
     asp=0)
```

Wie hoch war die Dichte im Aufnahmejahr?

```{r Jahrgang 2018 - Aufnahmejahr - Dichte}
edge_density(jg18_aufnahme1) # 0,008130081 (= 0,8 %)
```

Aus wie vielen Cluster und Communities bestand das Netzwerk im Aufnahmejahr?

```{r Jahrgang 2018 - Aufnahmejahr - Clusteranalyse}
jg18_aufnahme1
gc_jg18a <- cluster_walktrap(jg18_aufnahme1)

# Berechne Modularität
modularity(gc_jg18a) # 0,9465021

membership(gc_jg18a)
par(mfrow=c(1,1), mar=c(0,0,1,2))

# Listet die Components auf:
components(jg18_aufnahme1)

#Visualisierung der Cluster
plot(gc_jg18a, 
     jg18_aufnahme1, 
     edge.arrow.size=0.0001,
     edge.arrow.color="white",
     main="Clusteranalyse des #30u30-Jahrgangs 2018",
     sub="Nur Stationen im Aufnahmejahr (2018)",
     edge.curved=curve_multiple(jg18_aufnahme1),
     vertex.frame.color="white",
     edge.arrow.size=.2, 
     edge.color="grey",
     edge.curved=.2,
     vertex.size=5,
     vertex.label.degree=0, 
     vertex.label.cex=.15,
     vertex.label.family="sans",
     vertex.label.font=2,
     vertex.label.color="black")

```
### Jahrgang 2018 - Alle Stationen bis zum Aufnahmejahr
Zuletzt wird das Netzwerk der Jahrgangsmitglieder betrachtet mit all ihren Verbindungen bis zum Jahr der Aufnahme, um die Überschneidungen bis dahin zu beleuchten.

Zunächst wird das entsprechende Teilnetzwerk erstellt.

```{r Jahrgang 2018 - Bis Aufnahme - Plot}
jg18

jg18_preentry <- subgraph.edges(jg18, E(jg18)[year <= 2018]) # selektiert alle Beziehungen, die bis einschließlich 2018 bestanden

jg18_preentry
jg18_preentry1 <- simplify(jg18_preentry, remove.multiple = TRUE) # entfernt doppelte Kanten

V(jg18_preentry1)[mentor==1]$color <- "firebrick1"

jg18_preentry1

# Generiert einen Plot des Netzwerks - ohne Degree-Visualisierung
plot(jg18_preentry1,
     layout=layout_with_kk,
     edge.curved=curve_multiple(jg18_preentry1), # verhindert, dass sich Kanten überlagern
     vertex.frame.color=NA,
     edge.curved=.2,
     edge.color="grey80",
     edge.arrow.size=.0000001,
     edge.arrow.color="white",
     vertex.size=2, 
     vertex.label.cex=.15,
     vertex.label.family="sans",
     vertex.label.font=2,
     vertex.label.color="black",
     main="Netzwerk der Crew 2018 der #30u30",
     sub="Stationen bis zum Aufnahmejahr (2018)",
     asp=0)

ind_jg18_pre <- degree(jg18_preentry1, mode="in")

# Generiert einen Plot (Knotengröße entspr. Indegree-Wert skaliert)
plot(jg18_preentry1,
     layout=layout_with_kk,
     edge.curved=curve_multiple(jg18_preentry1), # verhindert, dass sich Kanten überlagern
     vertex.frame.color=NA,
     edge.curved=.2,
     edge.color="grey80",
     vertex.size=ind_jg18_pre*1.2, 
     vertex.label.cex=.15,
     vertex.label.family="sans",
     vertex.label.font=2,
     vertex.label.color="black",
     edge.arrow.size=.000001,
     edge.arrow.color="white",
     main="Netzwerk der Crew 2018 der #30u30",
     sub="Alle Stationen bis zum Aufnahmejahr (2018),
     Knotengröße entsprechend des Indegree-Werts und mit Faktor 1,2 skaliert",
     asp=0)
```
Wie viele Organisationen enthält das Netzwerk bis zum Jahr der Aufnahme?

```{r Jahrgang 2018 - Bis Aufnahme - Anzahl der Organisationen}

jg18_org_pe <- V(jg18_preentry1)$type == 2
sum(jg18_org_pe, na.rm=T) # 191 Organisationen

```
Wie hoch ist die Dichte im Netzwerk des Jahrgangs 2018 bis zum Jahr der Aufnahme?

```{r Jahrgang 2018 - Bis Aufnahme - Dichte}
edge_density(jg18_preentry1) # 0,00458659 (= 0,458 %)
```
Wie lang sind die mittlere Pfaddistanz und der Durchmesser des Netzwerks des Jahrgangs 2018 bis zum Aufnahmejahr?

```{r Jahrgang 2018 - Bis Aufnahme - Pfaddistanz & Diameter}
# Mittlere Pfaddistanz
mean_distance(jg18_preentry1, directed = FALSE) # 7,003882

# Längster Pfad / Diameter - am weitesten entfernte Knoten
diameter(jg18_preentry1, directed=FALSE) # 16 Schritte
farthest_vertices(jg18_preentry1, directed=FALSE) # Alsterradio, Debattierclub der JGU Mainz
```

Welche Organisationen waren besonders populär bzw. verbinden bis zur Aufnahme mehrere Talente?

```{r Jahrgang 2018 - Bis Aufnahme - Indegree-Verteilung}
jg18_ind_auf <- degree(jg18_preentry1, mode="in")
jg18_ind_auf
sort(jg18_ind_auf)
```

Welche 30u30-Mitglieder des JG 18 sind bis zum Jahr der Aufnahme besonders stark vernetzt bzw. haben besonders viele Verbindungen?

```{r Jahrgang 2018 - Bis Aufnahme - Outdegree-Verteilung}
jg18_outd_pe <- degree(jg18_preentry1, mode="out")
jg18_outd_pe
sort(jg18_outd_pe)
```

Aus wie viele Komponenten, Cluster und Communities besteht das Netzwerk des Jahrgangs 2018 bis zum Aufnahmejahr?

```{r Jahrgang 2018 - Bis Aufnahme - Clusteranalyse}

jg18_preentry1
gc_jg18pe <- cluster_walktrap(jg18_preentry1)

# Berechne Modularität
modularity(gc_jg18pe) # 0,8536065

membership(gc_jg18pe)
par(mfrow=c(1,1), mar=c(0,0,1,2))

#Zeigt components an
components(jg18_preentry1)

#Visualisierung der Cluster
plot(gc_jg18pe, 
     jg18_preentry1, 
     edge.arrow.size=0.0001,
     edge.arrow.color="white",
     main="Clusteranalyse des #30u30-Jahrgangs 2018",
     sub="Stationen bis zum Aufnahmejahr (2018)",
     edge.curved=curve_multiple(jg18_preentry1),
     vertex.frame.color="white",
     edge.arrow.size=.2, 
     edge.color="grey",
     edge.curved=.2,
     vertex.size=5,
     vertex.label.degree=0, 
     vertex.label.cex=.15,
     vertex.label.family="sans",
     vertex.label.font=2,
     vertex.label.color="black")
```
Betweennness-Werte im Netzwerk des Jahrgangs 2018 bis zum Aufnahmejahr: Wer sind die Broker?

```{r Jahrgang 2018 - Bis Aufnahme - Betweenness}

betweenness(jg18_preentry1)
centr_betw(jg18_preentry1, directed=TRUE)
betw18pe <- betweenness(jg18_preentry1)
sort(betw18pe)

V(jg18_preentry1)[mentor==1]$color <- "firebrick1" # färbt #30u30 rot

  # Wir können die Ausgabe von betweenness()  einer Variablen im Netz zuordnen und die Knoten entsprechend dimensionieren.
V(jg18_preentry1)$betweenness <- betweenness(jg18_preentry1)

plot(jg18_preentry1, 
     vertex.label.cex = .6, 
     vertex.label.color = "black", 
     vertex.size = V(jg18_preentry1)$betweenness) # Größe angepasst an betweenness

# Da die  Betweenness-Zentralität sehr groß sein kann, wird sie im folgenden normalisiert, indem man sie durch das Maximum teilt und mit einem Skalar multipliziert, bevor man sie plottet. So erkennen wir die Broker besser.
plot(jg18_preentry1,
     layout=layout_with_kk,
     vertex.label.cex = .15, 
     vertex.label.color = "black", 
     vertex.label.family="sans",
     edge.arrow.size=.00001,
     edge.arrow.color="white",
     asp=0,
     vertex.size = V(jg18_preentry1)$betweenness/max(V(jg18_preentry1)$betweenness) * 10,
     main="Broker im Netzwerk des #30u30-Jahrgangs 2018",
     sub="Alle Stationen bis zum Aufnahmejahr (2018)")
```

## Jahrgang 2019

### Gesamtnetzwerk Jahrgang 2019
Nun wird das Teilnetzwerk des Jahrgangs 2019 erstellt, das alle Stationen / Verbindungen der Mitglieder bis zum März 2022 beinhaltet.

```{r Teilnetzwerk Jahrgang 2019}
members # ruft das Gesamtnetzwerk der #30u30-Mitglieder auf

jg19 <- delete.vertices(members, V(members)[entry > 2019]) # löscht aus dem Gesamtnetzwerk die #30u30-Mitglieder, die nach 2019 aufgenommen wurden (Knoten mit entry = 99 - Organisationen bleiben ebenfalls erhalten, da wir deren Verbindungen zum Jahrgang abbilden wollen)

jg19_1 <- delete.vertices(jg19, V(jg19) [(mentor == 1) & (entry < 2019)]) # löscht die #30u30-Mitglieder, die vor 2019 aufgenommen wurden

jg19_1

jg19_2 <- delete_vertices(jg19_1, V(jg19_1)[degree(jg19_1, mode="all")==0]) # entfernt alle isolierten Knoten

#Generiert eine erste Visualisierung des Teilnetzwerks des Jahrgangs 2019
plot(jg19_1,
     layout=layout_with_kk,
     edge.curved=curve_multiple(jg19_1), # verhindert, dass sich Kanten überlagern
     vertex.frame.color="white",
     edge.color="grey",
     edge.curved=.2,
     vertex.size=3,
     vertex.label.degree=0, 
     vertex.label.cex=.15,
     vertex.label.family="sans",
     vertex.label.font=2,
     vertex.label.color="black",
     main="Netzwerk der Crew 2019 der #30u30",
     sub="Alle Stationen bis März 2022",
     edge.arrow.size=.00001,
     edge.arrow.color="white",
     rescale=T,
     asp=0)
```

Damit mehrjährige Beziehungen nur einfach angezeigt werden, werden multiple Kanten über simplify entfernt, die Knotenvisualisierung angepasst und erneut geplottet.

```{r Jahrgang 2019 - Kanten-Vereinfachung}
jg19_2

jg19_3 <- simplify(jg19_2, remove.multiple=TRUE) # Kantenvereinfachung

V(jg19_3)[V(jg19_3)$type == 1]$color <- "firebrick1" # Personen werden rot gefärbt
V(jg19_3)[V(jg19_3)$type == 1]$shape <- "circle" # Personen werden als Kreis angezeigt
jg19_3

# Generiert erneut den Plot des Netzwerks des Jahrgangs 2019
plot(jg19_3,
     layout=layout_with_kk,
     edge.curved=curve_multiple(jg19_3), # verhindert, dass sich Kanten überlagern
     vertex.frame.color=NA,
     edge.curved=.2,
     vertex.size=2,
     vertex.label.cex=.15,
     vertex.label.family="sans",
     vertex.label.font=2,
     vertex.label.color="black",
     edge.arrow.size=0.001,
     edge.arrow.color="white",
     main="Netzwerk der Crew 2019 der #30u30",
     sub="Alle Stationen bis März 2022",
     asp=0)
```

Wie viele Organisationen sind insgesamt im Netzwerk des Jahrgangs 2019 enthalten?

```{r Jahrgang 2019 - Anzahl Organisationen}
jg19_org <- V(jg19_3)$type == 2
sum(jg19_org, na.rm=T) # im Netzwerk der Alumni des JG 2019 sind 233 Organisationen enthalten
```

Welche Organisationen haben die meisten eingehenden Verbindungen bzw. verbinden die meisten Talente des Jahrgangs?

```{r Jahrgang 2019 - Indegree-Verteilung}
jg19_ind <- degree(jg19_3, mode="in")
jg19_ind
sort(jg19_ind)
```

Welche #30u30-Mitglieder des Jahrgangs 2019 sind besonders stark vernetzt?

```{r Jahrgang 2019 -  Outdegree-Verteilung}
jg19_outd <- degree(jg19_3, mode="out")
jg19_outd
sort(jg19_outd)
```

Zur Hervorhebung der zentralen, populären Organisationen, die mehrere Personen verbinden, wird die Knotengröße entsprechend des Indegree-Werts skaliert und so geplottet.

```{r Jahrgang 2019 - Gesamtnetzwerk - indegree-skaliert}

ind_jg19 <- degree(jg19_3, mode="in")

# Generiert den Plot des Netzwerks des Jahrgangs 2019, die Knoten mit höherem Indegree werden größer dargestellt
plot(jg19_3,
     layout=layout_with_kk,
     edge.curved=curve_multiple(jg19_3), # verhindert, dass sich Kanten überlagern
     vertex.frame.color=NA,
     edge.curved=.2,
     vertex.size=ind_jg19*1.2,
     vertex.label.cex=.15,
     vertex.label.family="sans",
     vertex.label.font=2,
     vertex.label.color="black",
     edge.arrow.size=0.001,
     edge.arrow.color="white",
     main="Netzwerk der Crew 2019 der #30u30",
     sub="Alle Verbindungen bis März 2022, 
     Knotengröße entspr. dem Indegree-Wert und mit Faktor 1,2 skaliert",
     asp=0)

```

Aus wie vielen Komponenten, Clustern, Communities besteht das Netzwerk des Jahrgangs 2019?

```{r Jahrgang 2019 - Clusteranalyse}
# Um die Überschneidungen des Jahrgangs besser zu greifen, folgt eine Clusteranalyse:
jg19_3
gc_jg19 <- cluster_walktrap(jg19_3)
membership(gc_jg19)

# Berechne Modularität
modularity(gc_jg19) # 0,820236

# Listet die Komponenten des Graphen auf
components(jg19_3)

par(mfrow=c(1,1), mar=c(0,0,1,2))

#Visualisierung der Cluster
plot(gc_jg19, 
     jg19_3, 
     edge.arrow.size=0.0001,
     edge.arrow.color="white",
     main="Clusteranalyse des #30u30-Jahrgangs 2019",
     sub="Alle Verbindungen bis März 2022",
     edge.curved=curve_multiple(jg19_3),
     vertex.frame.color="white",
     edge.color="grey",
     edge.curved=.2,
     vertex.size=5,
     vertex.label.degree=0, 
     vertex.label.cex=.15,
     vertex.label.family="sans",
     vertex.label.font=2,
     vertex.label.color="black")

# Plot / Cluster-Visualisierung einmal ohne Vertex-Labels
plot(gc_jg19, 
     jg19_3, 
     main="Clusteranalyse des #30u30-Jahrgangs 2019",
     sub="Alle Verbindungen bis März 2022",
     edge.curved=curve_multiple(jg19_3),
     vertex.frame.color="white",
     edge.arrow.size=.00002, 
     edge.arrow.color="white",
     edge.color="grey",
     edge.curved=.2,
     vertex.size=5,
     vertex.label=NA)

```
Wie hoch ist die Dichte im Netzwerk des Jahrgangs 2019?

```{r Jahrgang 2019 - Dichte}
edge_density(jg19_3) # 0,004063507

# Prozentwert
edge_density(jg19_3)*100 # 0,41 %
```
Wie lang sind die mittlere Pfaddistanz und der längste Pfad im Netzwerk des Jahrgangs 2019?

```{r Jahrgang 2019 - Pfaddistanz & Diameter}

mean_distance(jg19_3, directed=FALSE) # 6,431584

diameter(jg19_3, directed=FALSE) # Längster Pfad: 14 Schritte

farthest_vertices(jg19_3, directed = FALSE) # Artist Network & Heidelberg Cement (14 Schritte)

```
Betweenness-Werte im Netzwerk des Jahrgangs 2019: Wer sind die Broker?

```{r Jahrgang 2019 - Betweenness}
betweenness(jg19_3)
centr_betw(jg19_3, directed=TRUE)
betw19 <- betweenness(jg19_3)
sort(betw19)

V(jg19_3)[mentor==1]$color <- "firebrick1" #30u30 rot färben

# Wir können die Ausgabe von betweenness()  einer Variablen im Netz zuordnen und die Knoten entsprechend dimensionieren.
V(jg19_3)$betweenness <- betweenness(jg19_3)

# Visualisierung der Betweenness-Verteilung
plot(jg19_3, 
     vertex.label.cex = .6, 
     vertex.label.color = "black", 
     vertex.size = V(jg19_3)$betweenness) # Größe angepasst an betweenness

# Da die  Betweenness-Zentralität sehr groß sein kann, wird sie im folgenden normalisiert, indem man sie durch das Maximum teilt und mit einem Skalar multipliziert, bevor man sie plottet. So erkennen wir die Broker besser.
plot(jg19_3,
     layout=layout_with_kk,
     vertex.label.cex = .15, 
     vertex.label.color = "black", 
     vertex.label.family="sans",
     edge.arrow.size=.00001,
     edge.arrow.color="white",
     asp=0,
     vertex.size = V(jg19_3)$betweenness/max(V(jg19_3)$betweenness) * 10,
     main="Broker im Netzwerk des Jahrgangs 2019",
     sub="Alle Stationen bis März 2022, 
     Knotengröße entspr. dem Betweenness-Wert und Faktor 10 skaliert")
```

# Netzwerk des Jahrgangs 2019 - Stationen im Jahr der Aufnahme
Nun werden nur die Beziehungen selektiert, die in dem Jahr der Aufnahme bei #30u30 bestanden, um zu betrachten, ob es zu diesem Zeitpunkt Überschneidungen gab.

```{r Jahrgang 2019 - Aufnahmejahr - Plot}

jg19_aufnahme <- subgraph.edges(jg19_2, E(jg19_2)[year == 2019]) # selektiert die Beziehungen der Mitglieder aus dem Jahr 2019

jg19_aufnahme

jg19_aufnahme1 <- simplify(jg19_aufnahme, remove.multiple = TRUE) # entfernt doppelte Kanten

jg19_aufnahme1

V(jg19_aufnahme1)[mentor==1]$color <- "firebrick1" #30u30 rot hervorheben

# Generiert den Plot
plot(jg19_aufnahme1,
     layout=layout_with_kk,
     edge.curved=curve_multiple(jg19_aufnahme1), # verhindert, dass sich Kanten überlagern
     vertex.frame.color=NA,
     edge.curved=.2,
     edge.color="grey80",
     vertex.size=5, 
     vertex.label.cex=.2,
     vertex.label.family="sans",
     vertex.label.font=2,
     vertex.label.color="black",
     main="Netzwerk der Crew 2019 der #30u30",
     sub="Nur Verbindungen im Aufnahmejahr (2019)",
     edge.arrow.size=0.0001,
     edge.arrow.color="white",
     asp=0)
```

Wie hoch war die Dichte des Jahrgangs-Netzwerks im Aufnahmejahr?

```{r Jahrgang 2019 - Aufnahmejahr - Dichte}
edge_density(jg19_aufnahme1) # 0,008374963 (= 0,84 %)
```
Aus wie vielen Komponenten, Clustern, Communities besteht das Netzwerk im Aufnahmejahr?

```{r Jahrgang 2019 - Aufnahmejahr - Clusteranalyse}

jg19_aufnahme1
gc_jg19a <- cluster_walktrap(jg19_aufnahme1)

# Berechne Modularität
modularity(gc_jg19a) # 0,8958141

membership(gc_jg19a)
par(mfrow=c(1,1), mar=c(0,0,1,2))

# Listet die Components auf
components(jg19_aufnahme1)

#Visualisierung der Cluster
plot(gc_jg19a, 
     jg19_aufnahme1, 
     edge.arrow.size=0.0001,
     edge.arrow.color="white",
     main="Clusteranalyse des #30u30-Jahrgangs 2019",
     sub="Nur Stationen im Aufnahmejahr (2019)",
     edge.curved=curve_multiple(jg19_aufnahme1),
     vertex.frame.color="white",
     edge.arrow.size=.2, 
     edge.color="grey",
     edge.curved=.2,
     vertex.size=5,
     vertex.label.degree=0, 
     vertex.label.cex=.15,
     vertex.label.family="sans",
     vertex.label.font=2,
     vertex.label.color="black")
```
### Jahrgang 2019 - Alle Stationen bis zum Aufnahmejahr
Zuletzt wird das Netzwerk der Jahrgangsmitglieder betrachtet mit all ihren Verbindungen bis zum Jahr der Aufnahme, um die Überschneidungen bis dahin zu beleuchten.

Zunächst wird das entsprechende Teilnetzwerk erstellt.

```{r Jahrgang 2019 - Bis Aufnahme - Plot}
jg19_2

jg19_preentry <- subgraph.edges(jg19_2, E(jg19_2)[year <= 2019]) # selektiert alle Beziehungen der Mitglieder, die bis einschließlich 2019 bestanden
jg19_preentry

jg19_preentry1 <- simplify(jg19_preentry, remove.multiple = TRUE) # entfernt doppelte Kanten

V(jg19_preentry1)[mentor==1]$color <- "firebrick1" #färbt #30u30 rot

jg19_preentry1

# Generiert den Plot des Jahrgangsnetzwerks 2019 - ohne Degree-Visualisierung
plot(jg19_preentry1,
     layout=layout_with_kk,
     edge.curved=curve_multiple(jg19_preentry1), # verhindert, dass sich Kanten überlagern
     vertex.frame.color=NA,
     edge.curved=.2,
     edge.color="grey80",
     edge.arrow.size=.0000001,
     edge.arrow.color="white",
     vertex.size=2, 
     vertex.label.cex=.15,
     vertex.label.family="sans",
     vertex.label.font=2,
     vertex.label.color="black",
     main="Netzwerk der Crew 2019 der #30u30",
     sub="Alle Stationen bis zum Aufnahmejahr (2019)",
     asp=0)

ind_jg19_pre <- degree(jg19_preentry1, mode="in")

# Generiert einen Plot (Knotengröße entspr. Indegree-Wert skaliert)
plot(jg19_preentry1,
     layout=layout_with_kk,
     edge.curved=curve_multiple(jg19_preentry1), # verhindert, dass sich Kanten überlagern
     vertex.frame.color=NA,
     edge.curved=.2,
     edge.color="grey80",
     vertex.size=ind_jg19_pre*1.2, 
     vertex.label.cex=.15,
     vertex.label.family="sans",
     vertex.label.font=2,
     vertex.label.color="black",
     edge.arrow.size=.000001,
     edge.arrow.color="white",
     main="Netzwerk der Crew 2019 der #30u30",
     sub="Alle Stationen bis zum Aufnahmejahr (2019), 
     Knotengröße entspr. des Degree-Werts mit Faktor 1,2 skaliert",
     asp=0)
```

Wie viele Organisationen sind im Netzwerk des Jahrgangs 2019 bis zum Aufnahmejahr enthalten?

```{r Jahrgang 2019 - Bis Aufnahme - Anzahl der Organisationen}

jg19_org_pe <- V(jg19_preentry1)$type == 2
sum(jg19_org_pe, na.rm=T) # 208 Organisationen

```
Wie hoch ist die Dichte im Netzwerk des Jahrgangs 2019 bis zum Aufnahmejahr?

```{r Jahrgang 2019 - Bis Aufnahme - Dichte}
edge_density(jg19_preentry1) # 0,004396695 (= 0,44 %)
```
Wie lang ist die mittlere Pfaddistanz und wie weit sind die entferntesten Knoten voneinander entfernt (im Netzwerk bis zum Aufnahmejahr)?

```{r Jahrgang 2019 - Bis Aufnahme - Pfaddistanz & Diameter}

# Mittlere Pfaddistanz
mean_distance(jg19_preentry1, directed = FALSE) # 6,848954

# Längster Pfad / Diameter
diameter(jg19_preentry1, directed=FALSE) # 14
farthest_vertices(jg19_preentry1, directed=FALSE) # Anne Frank Stichting & Heidelberg Cement
```

Welche Organisationen verbinden mehrere Talente bis zum Jahr der Aufnahme?
Wo gibt es Überschneidungen?

```{r Jahrgang 2019 - Bis Aufnahme - Indegree-Verteilung}
jg19_ind_auf <- degree(jg19_preentry1, mode="in")
jg19_ind_auf
sort(jg19_ind_auf)
```

Welche Jahrgangsmitglieder haben bis zum Jahr der Aufnahme besonders viele Verbindungen? Wie vernetzt sind die einzelnen Personen?

```{r Jahrgang 2019 - Bis Aufnahme - Outdegree-Verteilung}
jg19_outd_pe <- degree(jg19_preentry1, mode="out")
jg19_outd_pe
sort(jg19_outd_pe)
```

Aus wie vielen Komponenten, Clustern, Commununities besteht das Netzwerk des Jahrgangs 2019 bis zum Aufnahmejahr?

```{r Jahrgang 2019 - Bis Aufnahme - Clusteranalyse}
jg19_preentry1
gc_jg19pe <- cluster_walktrap(jg19_preentry1)

# Berechne Modularität
modularity(gc_jg19pe) # 0,8222067

membership(gc_jg19pe)
par(mfrow=c(1,1), mar=c(0,0,1,2))

#Zeigt components an:
components(jg19_preentry1)

#Visualisierung der Cluster
plot(gc_jg19pe, 
     jg19_preentry1, 
     edge.arrow.size=0.0001,
     edge.arrow.color="white",
     main="Clusteranalyse des #30u30-Jahrgangs 2019",
     sub="Alle Stationen bis zum Aufnahmejahr (2019)",
     edge.curved=curve_multiple(jg19_preentry1),
     vertex.frame.color="white",
     edge.color="grey",
     edge.curved=.2,
     vertex.size=5,
     vertex.label.degree=0, 
     vertex.label.cex=.15,
     vertex.label.family="sans",
     vertex.label.font=2,
     vertex.label.color="black")
```
Betweenness-Werte im Netzwerk des Jahrgangs 2019 bis zum Aufnahmejahr: Wer sind bis dahin die Broker? 

```{r Jahrgang 2019 - Bis Aufnahme - Betweenness}
betweenness(jg19_preentry1)
centr_betw(jg19_preentry1, directed=TRUE)
betw19pe <- betweenness(jg19_preentry1)
sort(betw19pe)

V(jg19_preentry1)[mentor==1]$color <- "firebrick1" # färbt #30u30 rot

# Wir können die Ausgabe von betweenness()  einer Variablen im Netz zuordnen und die Knoten entsprechend dimensionieren.
V(jg19_preentry1)$betweenness <- betweenness(jg19_preentry1)

plot(jg19_preentry1, 
     vertex.label.cex = .6, 
     vertex.label.color = "black", 
     vertex.size = V(jg19_preentry1)$betweenness) # Größe angepasst an betweenness

# Da die  Betweenness-Zentralität sehr groß sein kann, wird sie im folgenden normalisiert, indem man sie durch das Maximum teilt und mit einem Skalar multipliziert, bevor man sie plottet. So erkennen wir die Broker besser.
plot(jg19_preentry1,
     layout=layout_with_kk,
     vertex.label.cex = .15, 
     vertex.label.color = "black", 
     vertex.label.family="sans",
     edge.arrow.size=.00001,
     edge.arrow.color="white",
     asp=0,
     vertex.size = V(jg19_preentry1)$betweenness/max(V(jg19_preentry1)$betweenness) * 10,
     main="Broker im Netzwerk des #30u30-Jahrgangs 2019",
     sub="Alle Stationen bis zum Aufnahmejahr (2019), 
     Knotengröße entspr. Betweenness-Wert und mit Faktor 10 skaliert")
```
## Jahrgang 2020

### Gesamtnetzwerk Jahrgang 2020
Nun wird das Teilnetzwerk des Jahrgangs 2020 erstellt, das alle Stationen / Verbindungen der Mitglieder bis zum März 2022 beinhaltet.

```{r Teilnetzwerk Jahrgang 2020}
members # ruft das Gesamtnetzwerk der #30u30-Mitglieder auf

jg20 <- delete.vertices(members, V(members)[(type == 1) & (entry != 2020)]) # löscht aus dem Gesamtnetzwerk alle #30u30-Mitglieder, die nicht 2020 aufgenommen wurden (Knoten mit entry = 99 - Organisationen bleiben ebenfalls erhalten, da wir deren Verbindungen zum Jahrgang abbilden wollen)

jg20 <- delete_vertices(jg20, V(jg20)[degree(jg20, mode="all")==0]) # entfernt alle isolierten Knoten

#Generiert eine erste Visualisierung des Jahrgangs 2020
plot(jg20,
     layout=layout_with_kk,
     edge.curved=curve_multiple(jg20), # verhindert, dass sich Kanten überlagern
     vertex.frame.color="white",
     edge.color="grey",
     edge.curved=.2,
     vertex.size=3,
     vertex.label.degree=0, 
     vertex.label.cex=.15,
     vertex.label.family="sans",
     vertex.label.font=2,
     vertex.label.color="black",
     main="Netzwerk der Crew 2020 der #30u30",
     sub="Alle Stationen bis März 2022",
     edge.arrow.size=.00001,
     edge.arrow.color="white",
     rescale=T,
     asp=0)
```
Damit mehrjährige Beziehungen nur einfach angezeigt werden, werden multiple Kanten über simplify entfernt, zudem wird die Visualisierung angepasst und erneut geplottet.

```{r Jahrgang 2020 - Kanten-Vereinfachung}
jg20
jg20_1 <- simplify(jg20, remove.multiple=TRUE) # Kantenvereinfachung

V(jg20_1)[V(jg20_1)$type == 1]$color <- "firebrick1" # Personen werden rot gefärbt
V(jg20_1)[V(jg20_1)$type == 1]$shape <- "circle" # Personen werden als Kreis angezeigt
jg20_1

# Generiert schönere Visualisierung des Plot des Jahrgangs-Netzwerks 2020
plot(jg20_1,
     layout=layout_with_kk,
     edge.curved=curve_multiple(jg20_1), # verhindert, dass sich Kanten überlagern
     vertex.frame.color=NA,
     edge.curved=.2,
     vertex.size=2,
     vertex.label.cex=.15,
     vertex.label.family="sans",
     vertex.label.font=2,
     vertex.label.color="black",
     edge.arrow.size=0.001,
     edge.arrow.color="white",
     main="Netzwerk der Crew 2020 der #30u30",
     sub="Alle Stationen bis März 2022",
     asp=0)
```

Wie viele Organisationen enthält das Netzwerk des Jahrgangs 2020 insgesamt?

```{r Jahrgang 2020 - Anzahl Organisationen}
jg20_org <- V(jg20_1)$type == 2
sum(jg20_org, na.rm=T) # 195 Organisationen
```
Welche Organisationen verbinden bis heute mehrere Talente? Wo gibt es Überschneidungen? Welche Organisation hat die meisten eingehenden Verbindungen?

```{r Jahrgang 2020 - Indegree-Verteilung}
jg20_ind <- degree(jg20_1, mode="in")
jg20_ind
sort(jg20_ind, decreasing = T)
```

Wie viele Verbindungen haben die Jahrgangsmitglieder bis heute? Welche 30u30-Mitglieder des JG 2020 sind besonders stark vernetzt?

```{r Jahrgang 2020 -  Outdegree-Verteilung}
jg20_outd <- degree(jg20_1, mode="out")
jg20_outd
sort(jg20_outd, decreasing = T)
```

Zur Hervorhebung der zentralen, populären Organisationen, die mehrere Personen im Jahrgang 2020 verbinden, wird die Knotengröße entsprechend des Indegree-Werts skaliert und so geplottet.

```{r Jahrgang 2020 - Gesamtnetzwerk - indegree-skaliert}
ind_jg20 <- degree(jg20_1, mode="in")

# Generiert den Plot
plot(jg20_1,
     layout=layout_with_kk,
     edge.curved=curve_multiple(jg20_1), # verhindert, dass sich Kanten überlagern
     vertex.frame.color=NA,
     edge.curved=.2,
     vertex.size=ind_jg20*1.2,
     vertex.label.cex=.15,
     vertex.label.family="sans",
     vertex.label.font=2,
     vertex.label.color="black",
     edge.arrow.size=0.001,
     edge.arrow.color="white",
     main="Netzwerk der Crew 2020 der #30u30",
     sub="Alle Stationen bis März 2022, 
     Knotengröße entspr. Indegree-Wert und mit Faktor 1,2 skaliert",
     asp=0)
```

Wie viele Komponenten, Cluster, Communities sind im Netzwerk des Jahrgangs 2020 enthalten?

```{r Jahrgang 2020 - Clusteranalyse}
jg20_1
gc_jg20 <- cluster_walktrap(jg20_1)

membership(gc_jg20)

# Berechne Modularität
modularity(gc_jg20) # 0,8621763

# Listet die Komponenten des Graphen auf
components(jg20_1)

par(mfrow=c(1,1), mar=c(0,0,1,2))

#Visualisierung der Cluster
plot(gc_jg20, 
     jg20_1, 
     edge.arrow.size=0.0001,
     edge.arrow.color="white",
     main="Clusteranalyse des #30u30-Jahrgangs 2020",
     sub="Alle Stationen bis März 2022",
     edge.curved=curve_multiple(jg20_1),
     vertex.frame.color="white",
     edge.color="grey",
     edge.curved=.2,
     vertex.size=5,
     vertex.label.degree=0, 
     vertex.label.cex=.15,
     vertex.label.family="sans",
     vertex.label.font=2,
     vertex.label.color="black")

# Visualisierung einmal ohne Vertex-Labels
plot(gc_jg20, 
     jg20_1,
     main="Clusteranalyse des #30u30-Jahrgangs 2020",
     sub="Alle Stationen bis März 2022",
     edge.curved=curve_multiple(jg20_1),
     vertex.frame.color="white",
     edge.arrow.size=.00001, 
     edge.arrow.color="white",
     edge.color="grey",
     edge.curved=.2,
     vertex.size=5,
     vertex.label=NA)
```
Wie hoch ist die Dichte im Netzwerk des Jahrgangs 2020 heute?

```{r Jahrgang 2020 - Dichte}
edge_density(jg20_1)

# Prozentwert
edge_density(jg20_1)*100 # 0,44 %
```
Wie lang ist die mittlere Pfaddistanz und wie breit der Durchmesser des Netzwerks des Jahrgangs 2020?

```{r Jahrgang 2020 - Pfaddistanz & Diameter }
# Mittlere Pfaddistanz
mean_distance(jg20_1, directed=FALSE) # 5,48019

# Durchmesser
diameter(jg20_1, directed=FALSE) # Längster Pfad: 10 Schritte

farthest_vertices(jg20_1, directed = FALSE) # Hamburger Abendblatt & HS Hannover (10 Schritte)

```

Betweennness-Werte im Netzwerk des Jahrgangs 2020: Wer sind die Broker?

```{r Jahrgang 2020 - Betweenness}
betweenness(jg20_1)
centr_betw(jg20_1, directed=TRUE)
betw20 <- betweenness(jg20_1)
sort(betw20)

V(jg20_1)[mentor==1]$color <- "firebrick1" #30u30 rot färben

# Wir können die Ausgabe von betweenness()  einer Variablen im Netz zuordnen und die Knoten entsprechend dimensionieren.
V(jg20_1)$betweenness <- betweenness(jg20_1)

# Visualisierung der Betweenness-Verteilung
plot(jg20_1, 
     vertex.label.cex = .6, 
     vertex.label.color = "black", 
     vertex.size = V(jg20_1)$betweenness) # Größe angepasst an betweenness

# Da die  Betweenness-Zentralität sehr groß sein kann, wird sie im folgenden normalisiert, indem man sie durch das Maximum teilt und mit einem Skalar multipliziert, bevor man sie plottet. So erkennen wir die Broker besser.
plot(jg20_1,
     layout=layout_with_kk,
     vertex.label.cex = .15, 
     vertex.label.color = "black", 
     vertex.label.family="sans",
     edge.arrow.size=.00001,
     edge.arrow.color="white",
     asp=0,
     vertex.size = V(jg20_1)$betweenness/max(V(jg20_3)$betweenness) * 10,
     main="Broker im Netzwerk des #30u30-Jahrgangs 2020",
     sub="Alle Stationen bis März 2022
     Knotengröße entspr. dem Betweennness-Wert mit Faktor 10 skaliert")
```

### Jahrgang 2020 - Stationen im Jahr der Aufnahme
Nun werden nur die Beziehungen selektiert, die in dem Jahr der Aufnahme bei #30u30 bestanden, um zu betrachten, ob es zu diesem Zeitpunkt Überschneidungen gab.

```{r Jahrgang 2020 - Aufnahmejahr - Plot}
jg20_aufnahme <- subgraph.edges(jg20, E(jg20)[year == 2020]) # selektiert die Beziehungen aus dem Jahr 2020
jg20_aufnahme

jg20_aufnahme1 <- simplify(jg20_aufnahme, remove.multiple = TRUE) # entfernt doppelte Kanten
jg20_aufnahme1

V(jg20_aufnahme1)[mentor==1]$color <- "firebrick1" #30u30 rot hervorheben

# Generiert den Plot
plot(jg20_aufnahme1,
     layout=layout_with_kk,
     edge.curved=curve_multiple(jg20_aufnahme1), # verhindert, dass sich Kanten überlagern
     vertex.frame.color=NA,
     edge.curved=.2,
     edge.color="grey80",
     vertex.size=5, 
     vertex.label.cex=.2,
     vertex.label.family="sans",
     vertex.label.font=2,
     vertex.label.color="black",
     main="Netzwerk der Crew 2020 der #30u30",
     sub="Nur Verbindungen im Jahr der Aufnahme (2020)",
     edge.arrow.size=0.0001,
     edge.arrow.color="white",
     asp=0)

```

Wie hoch war die Dichte im Netzwerk des Jahrgangs 2020 im Jahr der Aufnahme?

```{r Jahrgang 2020 - Aufnahmejahr - Dichte}
edge_density(jg20_aufnahme1) # 0,008641975 (0,86 %)
```
Aus wie vielen Komponenten, Clustern, Communities besteht das Netzwerk des Jahrgangs 2020 im Aufnahmejahr?

```{r Jahrgang 2020 - Aufnahmejahr - Clusteranalyse}
jg20_aufnahme1
gc_jg20a <- cluster_walktrap(jg20_aufnahme1)

# Berechne Modularität
modularity(gc_jg20a) # 0,9355868

membership(gc_jg20a)
par(mfrow=c(1,1), mar=c(0,0,1,2))

# Listet die Components auf:
components(jg20_aufnahme1)

#Visualisierung der Cluster
plot(gc_jg20a, 
     jg20_aufnahme1, 
     edge.arrow.size=0.0001,
     edge.arrow.color="white",
     main="Clusteranalyse des #30u30-Jahrgangs 2020",
     sub="Nur Verbindungen im Aufnahmejahr (2020)",
     edge.curved=curve_multiple(jg20_aufnahme1),
     vertex.frame.color="white",
     edge.color="grey",
     edge.curved=.2,
     vertex.size=5,
     vertex.label.degree=0, 
     vertex.label.cex=.15,
     vertex.label.family="sans",
     vertex.label.font=2,
     vertex.label.color="black")
```
### Jahrgang 2020 - Alle Stationen bis zum Aufnahmejahr
Zuletzt wird das Netzwerk der Jahrgangsmitglieder betrachtet mit all deren Verbindungen bis zum Jahr der Aufnahme, um die Überschneidungen bis dahin zu beleuchten.

Zunächst wird dazu das entsprechende Teilnetzwerk erstellt.

```{r Jahrgang 2020 - Bis Aufnahme - Plot}
jg20_2

jg20_preentry <- subgraph.edges(jg20_2, E(jg20_2)[year <= 2020]) # selektiert alle Beziehungen, die bis einschließlich 2020 bestanden

jg20_preentry
jg20_preentry1 <- simplify(jg20_preentry, remove.multiple = TRUE) # entfernt doppelte Kanten

V(jg20_preentry1)[mentor==1]$color <- "firebrick1" #färbt #30u30 rot

jg20_preentry1

# Generiert einen Plot des Netzwerks - ohne Degree-Visualisierung
plot(jg20_preentry1,
     layout=layout_with_kk,
     edge.curved=curve_multiple(jg20_preentry1), # verhindert, dass sich Kanten überlagern
     vertex.frame.color=NA,
     edge.curved=.2,
     edge.color="grey80",
     edge.arrow.size=.0000001,
     edge.arrow.color="white",
     vertex.size=2, 
     vertex.label.cex=.15,
     vertex.label.family="sans",
     vertex.label.font=2,
     vertex.label.color="black",
     main="Netzwerk der Crew 2020 der #30u30",
     sub="Alle Stationen bis zum Aufnahmejahr (2020)",
     asp=0)

ind_jg20_pre <- degree(jg20_preentry1, mode="in")

# Generiert einen Plot (Knotengröße entspr. Indegree-Wert skaliert)
plot(jg20_preentry1,
     layout=layout_with_kk,
     edge.curved=curve_multiple(jg20_preentry1), # verhindert, dass sich Kanten überlagern
     vertex.frame.color=NA,
     edge.curved=.2,
     edge.color="grey80",
     vertex.size=ind_jg20_pre*1.2, 
     vertex.label.cex=.15,
     vertex.label.family="sans",
     vertex.label.font=2,
     vertex.label.color="black",
     edge.arrow.size=.000001,
     edge.arrow.color="white",
     main="Netzwerk der Crew 2020 der #30u30",
     sub="Alle Stationen bis zum Aufnahmejahr (2020), 
     Knotengröße entspr. dem Indegree-Wert und mit Faktor 1,2 skaliert",
     asp=0)
```
Wie viele Organisationen sind im Netzwerk des Jahrgangs 2020 bis zum Aufnahmejahr enthalten?

```{r Jahrgang 2020 - Bis Aufnahme - Anzahl der Organisationen}
jg20_org_pe <- V(jg20_preentry1)$type == 2
sum(jg20_org_pe, na.rm=T) # 187 Organisationen
```
Wie hoch ist die Dichte im Netzwerk bis zum Jahr der Aufnahme?

```{r Jahrgang 2020 - Bis Aufnahme - Dichte}
edge_density(jg20_preentry1) # 0,004480287 (= 0,44 %)
```
Wie lang ist die mittlere Pfaddistanz und wie breit der Durchmesser des Netzwerks des Jahrgangs 2020 bis zum Aufnahmejahr?

```{r Jahrgang 2020 - Bis Aufnahme - Pfaddistanz & Diameter}
# Mittlere Pfaddistanz
mean_distance(jg20_preentry1, directed = FALSE) # 6,090808

# Längster Pfad / Diameter
diameter(jg20_preentry1, directed=FALSE) # 14 Schritte
farthest_vertices(jg20_preentry1, directed=FALSE) # von Hamburger Abendblatt bis Niedersächs. Landtag
```
Welche Organisationen stechen bis zum Aufnahmejahr heraus, weil sie mehrere Mitglieder verbinden? Welche sind die populärsten Organisationen?

```{r Jahrgang 2020 - Bis Aufnahme - Indegree-Verteilung}
jg20_ind_auf <- degree(jg20_preentry1, mode="in")
jg20_ind_auf
sort(jg20_ind_auf, decreasing = T)
```

Wie viele Verbindungen haben die einzelnen Mitglieder des Jahrgangs 2020 bis zum Aufnahmejahr? Welche Mitglieder sind zum Zeitpunkt der Aufnahme besonders stark vernetzt?

```{r Jahrgang 2020 - Bis Aufnahme - Outdegree-Verteilung}
jg20_outd_pe <- degree(jg20_preentry1, mode="out")
jg20_outd_pe
sort(jg20_outd_pe, decreasing = T)
```

Aus wie vielen Komponenten, Clustern, Communities besteht das Netzwerk des Jahrgangs 2020 bis zum Aufnahmejahr?

```{r Jahrgang 2020 - Bis Aufnahme - Clusteranalyse}
jg20_preentry1
gc_jg20pe <- cluster_walktrap(jg20_preentry1)

# Berechne Modularität
modularity(gc_jg20pe) # 0,8716099

membership(gc_jg20pe)
par(mfrow=c(1,1), mar=c(0,0,1,2))

#Zeigt components an:
components(jg20_preentry1)

#Visualisierung der Cluster
plot(gc_jg20pe, 
     jg20_preentry1, 
     edge.arrow.size=0.0001,
     edge.arrow.color="white",
     main="Clusteranalyse des #30u30-Jahrgangs 2020",
     sub="Alle Stationen bis zum Aufnahmejahr (2020)",
     edge.curved=curve_multiple(jg20_preentry1),
     vertex.frame.color="white",
     edge.color="grey",
     edge.curved=.2,
     vertex.size=5,
     vertex.label.degree=0, 
     vertex.label.cex=.15,
     vertex.label.family="sans",
     vertex.label.font=2,
     vertex.label.color="black")
```

Betweenness-Werte im Netzwerk des Jahrgangs 2020 bis zum Aufnahmejahr: Wer sind die Broker?

```{r Jahrgang 2020 - Bis Aufnahme - Betweenness}
betweenness(jg20_preentry1)
centr_betw(jg20_preentry1, directed=TRUE)
betw20pe <- betweenness(jg20_preentry1)
sort(betw20pe, decreasing = T)

V(jg20_preentry1)[mentor==1]$color <- "firebrick1" # färbt #30u30 rot

# Wir können die Ausgabe von betweenness()  einer Variablen im Netz zuordnen und die Knoten entsprechend dimensionieren.
V(jg19_preentry1)$betweenness <- betweenness(jg19_preentry1)

plot(jg19_preentry1, 
     vertex.label.cex = .6, 
     vertex.label.color = "black", 
     vertex.size = V(jg19_preentry1)$betweenness) # Größe angepasst an betweenness

# Da die  Betweenness-Zentralität sehr groß sein kann, wird sie im folgenden normalisiert, indem man sie durch das Maximum teilt und mit einem Skalar multipliziert, bevor man sie plottet. So erkennen wir die Broker besser.
plot(jg19_preentry1,
     layout=layout_with_kk,
     vertex.label.cex = .15, 
     vertex.label.color = "black", 
     vertex.label.family="sans",
     edge.arrow.size=.00001,
     edge.arrow.color="white",
     asp=0,
     vertex.size = V(jg19_preentry1)$betweenness/max(V(jg19_preentry1)$betweenness) * 10,
     main="Broker im Netzwerk des #30u30-Jahrgangs 2020",
     sub="Alle Stationen bis zum Aufnahmejahr (2020), 
     Knotengröße entspr. Betweenness-Wert und mit Faktor 10 skaliert")
```
## Jahrgang 2021

### Gesamtnetzwerk Jahrgang 2021
Zuletzt wird das Teilnetzwerk des Jahrgangs 2021 erstellt, das alle Stationen bzw. Verbindungen der Mitglieder bis zum März 2022 beinhaltet.

```{r Teilnetzwerk Jahrgang 2021}
members # ruft das Gesamtnetzwerk der #30u30-Mitglieder auf

jg21 <- delete.vertices(members, V(members)[(type == 1) & (entry < 2021)]) # löscht aus dem Gesamtnetzwerk die #30u30-Mitglieder, die vor 2021 aufgenommen wurden (Knoten mit entry = 99 - Organisationen bleiben erhalten, da wir deren Verbindungen zum Jahrgang abbilden wollen)
jg21

jg21_1 <- delete_vertices(jg21, V(jg21)[degree(jg21, mode="all")==0]) # entfernt alle isolierten Knoten

#Generiert eine erste Visualisierung des Jahrgangs 2021
plot(jg21_1,
     layout=layout_with_kk,
     edge.curved=curve_multiple(jg21_1), # verhindert, dass sich Kanten überlagern
     vertex.frame.color="white",
     edge.color="grey",
     edge.curved=.2,
     vertex.size=3,
     vertex.label.degree=0, 
     vertex.label.cex=.15,
     vertex.label.family="sans",
     vertex.label.font=2,
     vertex.label.color="black",
     main="Netzwerk der Crew 2021 der #30u30",
     sub="Alle Verbindungen bis März 2022",
     edge.arrow.size=.00001,
     edge.arrow.color="white",
     rescale=T,
     asp=0)
```

Damit mehrjährige Beziehungen nur einfach angezeigt werden, werden multiple Kanten über simplify entfernt, zudem wird die Visualisierung der Knoten angepasst und dann erneut geplottet.

```{r Jahrgang 2021 - Kanten-Vereinfachung}
jg21_2 <- simplify(jg21_1, remove.multiple=TRUE) # löscht mehrfache Kanten

V(jg21_2)[V(jg21_2)$type == 1]$color <- "firebrick1" # Personen werden rot gefärbt
V(jg21_2)[V(jg21_2)$type == 1]$shape <- "circle" # Personen werden als Kreis angezeigt

# Generiert einen schöneren Plot des Netzwerks des Jahrgangs 2021
jg21_2

plot(jg21_2,
     layout=layout_with_kk,
     edge.curved=curve_multiple(jg21_2), # verhindert, dass sich Kanten überlagern
     vertex.frame.color=NA,
     edge.curved=.2,
     vertex.size=2,
     vertex.label.cex=.15,
     vertex.label.family="sans",
     vertex.label.font=2,
     vertex.label.color="black",
     edge.arrow.size=0.001,
     edge.arrow.color="white",
     main="Netzwerk der Crew 2021 der #30u30",
     sub="Alle Stationen bis März 2021",
     asp=0)
```

Wie viele Organisationen sind insgesamt im Netzwerk des Jahrgangs 2021 enthalten?

```{r Jahrgang 2021 - Anzahl Organisationen}
jg21_org <- V(jg21_2)$type == 2
sum(jg21_org, na.rm=T) # 229 Organisationen 
```
Mit wie vielen Personen stehen die Organisationen jeweils in Verbindung? Welche sind besonders populär und verbinden bis heute mehrere Personen?

```{r Jahrgang 2021 - Indegree-Verteilung}
jg21_ind <- degree(jg21_2, mode="in")
jg21_ind
sort(jg21_ind, decreasing = T)
```

Wie viele Beziehungen gehen von den #30u30 des Jahrgangs aus? Wer ist bis heute besonders stark vernetzt?

```{r Jahrgang 2021 -  Outdegree-Verteilung}
jg21_outd <- degree(jg21_2, mode="out")
jg21_outd
sort(jg21_outd, decreasing = T)
```

Um die populären Organisationen hervorzuheben, wird das Jahrgangsnetzwerk erneut geplottet und dabei die Knoten bei hohem Indegree-Wert größer dargestellt.

```{r Jahrgang 2021 - Gesamtnetzwerk - indegree-skaliert}
ind_jg21 <- degree(jg21_2, mode="in")

# Generiert den Plot, bei dem Knotengröße je nach Indegree-Wert skaliert wird
plot(jg21_2,
     layout=layout_with_kk,
     edge.curved=curve_multiple(jg21_2), # verhindert, dass sich Kanten überlagern
     vertex.frame.color=NA,
     edge.curved=.2,
     vertex.size=ind_jg21*1.2,
     vertex.label.cex=.15,
     vertex.label.family="sans",
     vertex.label.font=2,
     vertex.label.color="black",
     edge.arrow.size=0.001,
     edge.arrow.color="white",
     main="Netzwerk der Crew 2021 der #30u30",
     sub="Alle Stationen bis März 2022, 
     Knotengröße entspr. Indegree-Wert und mit Faktor 1,2 skaliert",
     asp=0)
```
Aus wie vielen Komponenten, Clustern, Communities besteht das Netzwerk des Jahrgangs 2021 bis heute?

```{r Jahrgang 2021 - Clusteranalyse}
jg21_2
gc_jg21 <- cluster_walktrap(jg21_2)
membership(gc_jg21)

# Berechne Modularität
modularity(gc_jg21) # 0,8043439

# Listet die Komponenten des Graphen auf
components(jg21_2)
par(mfrow=c(1,1), mar=c(0,0,1,2))

#Visualisierung der Cluster
plot(gc_jg21, 
     jg21_2, 
     edge.arrow.size=0.0001,
     edge.arrow.color="white",
     main="Clusteranalyse des #30u30-Jahrgangs 2021",
     sub="Alle Stationen bis März 2022",
     edge.curved=curve_multiple(jg21_2),
     vertex.frame.color="white",
     edge.arrow.size=.2, 
     edge.color="grey",
     edge.curved=.2,
     vertex.size=5,
     vertex.label.degree=0, 
     vertex.label.cex=.15,
     vertex.label.family="sans",
     vertex.label.font=2,
     vertex.label.color="black")

# Visualisierung noch einmal ohne Vertex-Labels
plot(gc_jg21, 
     jg21_2, 
     main="Clusteranalyse des #30u30-Jahrgangs 2021",
     sub="Alle Stationen bis März 2022",
     edge.curved=curve_multiple(jg21_2),
     vertex.frame.color="white",
     edge.arrow.size=.00002, 
     edge.arrow.color="white",
     edge.color="grey",
     edge.curved=.2,
     vertex.size=5,
     vertex.label=NA)
```
Wie hoch ist die Dichte im Netzwerk des Jahrgangs 2021 mit allen Verbindungen bis März 2022?

```{r Jahrgang 2021 - Dichte}
edge_density(jg21_2) # 0.004115411

# Prozentwert
edge_density(jg21_2)*100 # 0,41 %
```
Wie lang ist die mittlere Pfaddistanz und wie breit ist der Durchmesser des Netzwerks des Jahrgangs 2021?

```{r Jahrgang 2021 - Pfaddistanz & Diameter}
# Mittlere Pfaddistanz
mean_distance(jg21_2, directed=FALSE)  # 5,926684

# Durchmesser / Diameter
diameter(jg21_2, directed=FALSE) # Längster Pfad: 10 Schritte

farthest_vertices(jg21_2, directed = FALSE)
# 3M & Berufsschule für den Großhandel, Außenhandel und Verkehr (10 Schritte)

```
Betweennness-Werte im Netzwerk des Jahrgangs 2021: Wer sind die Broker?

```{r Jahrgang 2021 - Betweenness}
betweenness(jg21_2)
centr_betw(jg21_2, directed=TRUE)
betw21 <- betweenness(jg21_2)
sort(betw21, decreasing = T)

V(jg21_2)[mentor==1]$color <- "firebrick1" #30u30 rot färben

# Wir können die Ausgabe von betweenness()  einer Variablen im Netz zuordnen und die Knoten entsprechend dimensionieren.
V(jg21_2)$betweenness <- betweenness(jg21_2)

# Visualisierung der Betweenness-Verteilung
plot(jg21_2, 
     vertex.label.cex = .6, 
     vertex.label.color = "black", 
     vertex.size = V(jg21_2)$betweenness) # Größe angepasst an betweenness

# Da die  Betweenness-Zentralität sehr groß sein kann, wird sie im folgenden normalisiert, indem man sie durch das Maximum teilt und mit einem Skalar multipliziert, bevor man sie plottet. So erkennen wir die Broker besser.
plot(jg21_2,
     layout=layout_with_kk,
     vertex.label.cex =.15, 
     vertex.label.color = "black", 
     vertex.label.family="sans",
     edge.arrow.size=.00001,
     edge.arrow.color="white",
     asp=0,
     vertex.size = V(jg21_2)$betweenness/max(V(jg21_2)$betweenness)*10,
     main="Broker im Netzwerk des #30u30-Jahrgangs 2021",
     sub="Alle Stationen bis März 2022, 
     Knotengröße entspr. Betweenness-Wert und mit Faktor 10 skaliert")
```

### Jahrgang 2021 - Stationen im Jahr der Aufnahme
Nun werden nur die Beziehungen selektiert, die in dem Jahr der Aufnahme bei #30u30 bestanden, um zu betrachten, ob es zu diesem Zeitpunkt Überschneidungen gab.

```{r Jahrgang 2021 - Aufnahmejahr - Plot}
jg21_aufnahme <- subgraph.edges(jg21_1, E(jg21_1)[year == 2021]) # selektiert die Beziehungen aus dem Jahr 2021
jg21_aufnahme

jg21_aufnahme1 <- simplify(jg21_aufnahme, remove.multiple = TRUE) # entfernt doppelte Kanten
jg21_aufnahme1

V(jg21_aufnahme1)[mentor==1]$color <- "firebrick1" #30u30 rot hervorheben

# Generiert den Plot
plot(jg21_aufnahme1,
     layout=layout_with_kk,
     edge.curved=curve_multiple(jg21_aufnahme1), # verhindert, dass sich Kanten überlagern
     vertex.frame.color=NA,
     edge.curved=.2,
     edge.color="grey80",
     vertex.size=5, 
     vertex.label.cex=.2,
     vertex.label.family="sans",
     vertex.label.font=2,
     vertex.label.color="black",
     main="Netzwerk der Crew 2021 der #30u30",
     sub="Nur Verbindungen im Aufnahmejahr (2021)",
     edge.arrow.size=0.0001,
     edge.arrow.color="white",
     asp=0)

```
Wie dicht ist das Netzwerk im Jahr der Aufnahme?

```{r Jahrgang 2021 - Aufnahmejahr - Dichte}
edge_density(jg21_aufnahme1) # 0,00822884 (= 0,82 %)
```
Aus wie vielen Komponenten, Clustern, Communities besteht das Netzwerk im Aufnahmejahr?

```{r Jahrgang 2021 - Aufnahmejahr - Clusteranalyse}
jg21_aufnahme1
gc_jg21a <- cluster_walktrap(jg21_aufnahme1)

# Berechne Modularität
modularity(gc_jg21a) # 0,9171077

membership(gc_jg21a)
par(mfrow=c(1,1), mar=c(0,0,1,2))

#Visualisierung der Cluster
plot(gc_jg21a, 
     jg21_aufnahme1, 
     edge.arrow.size=0.0001,
     edge.arrow.color="white",
     main="Clusteranalyse des #30u30-Jahrgangs 2021",
     sub="Nur Verbindungen im Aufnahmejahr (2021)",
     edge.curved=curve_multiple(jg21_aufnahme1),
     vertex.frame.color="white",
     edge.arrow.size=.2, 
     edge.color="grey",
     edge.curved=.2,
     vertex.size=5,
     vertex.label.degree=0, 
     vertex.label.cex=.15,
     vertex.label.family="sans",
     vertex.label.font=2,
     vertex.label.color="black")
```
### Jahrgang 2021 - Alle Stationen bis zum Aufnahmejahr
Zuletzt wird das Netzwerk der Jahrgangsmitglieder betrachtet mit all deren Verbindungen bis zum Jahr der Aufnahme, um die Überschneidungen bis dahin zu beleuchten.

Zunächst wird dazu das entsprechende Teilnetzwerk erstellt.

```{r Jahrgang 2021 - Bis Aufnahme - Plot}
jg21_1
jg21_preentry <- subgraph.edges(jg21_1, E(jg21_1)[year <= 2021]) # selektiert alle Beziehungen, die bis einschließlich 2021 bestanden
jg21_preentry

jg21_preentry1 <- simplify(jg21_preentry, remove.multiple = TRUE) # entfernt doppelte Kanten
V(jg21_preentry1)[mentor==1]$color <- "firebrick1" #färbt #30u30 rot
jg21_preentry1

# Generiert einen Plot des Netzwerks - ohne Degree-Visualisierung
plot(jg21_preentry1,
     layout=layout_with_kk,
     edge.curved=curve_multiple(jg21_preentry1), # verhindert, dass sich Kanten überlagern
     vertex.frame.color=NA,
     edge.curved=.2,
     edge.color="grey80",
     edge.arrow.size=.0000001,
     edge.arrow.color="white",
     vertex.size=2, 
     vertex.label.cex=.15,
     vertex.label.family="sans",
     vertex.label.font=2,
     vertex.label.color="black",
     main="Netzwerk der Crew 2021 der #30u30",
     sub="Alle Verbindungen bis zum Aufnahmejahr (2021)",
     asp=0)

ind_jg21_pre <- degree(jg21_preentry1, mode="in")

# Generiert einen Plot (Knotengröße entspr. Indegree skaliert)
plot(jg21_preentry1,
     layout=layout_with_kk,
     edge.curved=curve_multiple(jg21_preentry1), # verhindert, dass sich Kanten überlagern
     vertex.frame.color=NA,
     edge.curved=.2,
     edge.color="grey80",
     vertex.size=ind_jg21_pre*1.2, 
     vertex.label.cex=.15,
     vertex.label.family="sans",
     vertex.label.font=2,
     vertex.label.color="black",
     edge.arrow.size=.000001,
     edge.arrow.color="white",
     main="Netzwerk der Crew 2021 der #30u30",
     sub="Stationen bis zum Aufnahmejahr (2021), 
     Knotengröße entspr. des Indegree-Werts mit Faktor 1,2 skaliert",
     asp=0)
```

Wie viele Organisationen sind im Netzwerk des Jahrgangs 2021 bis zum Aufnahmejahr enthalten?

```{r Jahrgang 2021 - Bis Aufnahme - Anzahl der Organisationen}
jg21_org_pe <- V(jg21_preentry1)$type == 2
sum(jg21_org_pe, na.rm=T) # 228 Organisationen
```
Wie dicht ist das Netzwerk des Jahrgangs 2021 bis zum Jahr der Aufnahme?

```{r Jahrgang 2021 - Bis Aufnahme - Dichte}
edge_density(jg21_preentry1) # 0,004132356 (= 0,41 %)
```
Wie lang ist die mittlere Pfaddistanz und wie breit der Durchmesser des Netzwerks des Jahrgangs 2021 bis zum Aufnahmejahr?

```{r Jahrgang 2021 - Bis Aufnahme - Pfaddistanz & Diameter}
# Mittlere Pfaddistanz
mean_distance(jg21_preentry1, directed = FALSE) # 5,920431
# Längster Pfad / Diameter
diameter(jg21_preentry1, directed=FALSE) # 10 Schritte
farthest_vertices(jg21_preentry1, directed=FALSE) # 3M & Berufsschule für den Großhandel, Außenhandel & Verkehr
```
Welche der 228 Organisationen verbinden bis zum Aufnahmejahr mehrere Talente? Welche sind besonders populär?

```{r Jahrgang 2021 - Bis Aufnahme - Indegree-Verteilung}
jg21_ind_auf <- degree(jg21_preentry1, mode="in")
jg21_ind_auf
sort(jg21_ind_auf, decreasing = T)
```

Ausgehende Beziehungen: Welche Personen hatten bis zum Aufnahmejahr besonders viele Verbindungen?

```{r Jahrgang 2021 - Bis Aufnahme - Outdegree-Verteilung}
jg21_outd_pe <- degree(jg21_preentry1, mode="out")
jg21_outd_pe
sort(jg21_outd_pe)
```

Aus wie vielen Komponenten, Clustern, Communities besteht das Netzwerk des Jahrgangs 2021 bis zum Aufnahmejahr?

```{r Jahrgang 2021 - Bis Aufnahme - Clusteranalyse}
jg21_preentry1
gc_jg21pe <- cluster_walktrap(jg21_preentry1)

# Berechne Modularität
modularity(gc_jg21pe) # 0,8037257

membership(gc_jg21pe)
par(mfrow=c(1,1), mar=c(0,0,1,2))

#Zeigt components an
components(jg21_preentry1)

#Visualisierung der Cluster
plot(gc_jg21pe, 
     jg21_preentry1, 
     edge.arrow.size=0.0001,
     edge.arrow.color="white",
     main="Clusteranalyse des #30u30-Jahrgangs 2021",
     sub="Alle Stationen bis zum Aufnahmejahr (2021)",
     edge.curved=curve_multiple(jg21_preentry1),
     vertex.frame.color="white",
     edge.arrow.size=.2, 
     edge.color="grey",
     edge.curved=.2,
     vertex.size=5,
     vertex.label.degree=0, 
     vertex.label.cex=.15,
     vertex.label.family="sans",
     vertex.label.font=2,
     vertex.label.color="black")
```
Betweenness-Werte im Netzwerk des Jahrgangs 2021 bis zum Aufnahmejahr: Wer sind die Broker?

```{r Jahrgang 2021 - Bis Aufnahme - Betweenness}
betweenness(jg21_preentry1)
centr_betw(jg21_preentry1, directed=TRUE)
betw21pe <- betweenness(jg21_preentry1)
sort(betw21pe, decreasing = T)

V(jg21_preentry1)[mentor==1]$color <- "firebrick1" # färbt #30u30 rot

# Wir können die Ausgabe von betweenness()  einer Variablen im Netz zuordnen und die Knoten entsprechend dimensionieren.
V(jg21_preentry1)$betweenness <- betweenness(jg21_preentry1)

plot(jg21_preentry1, 
     vertex.label.cex = .6, 
     vertex.label.color = "black", 
     vertex.size = V(jg21_preentry1)$betweenness) # Größe angepasst an betweenness

# Da die  Betweenness-Zentralität sehr groß sein kann, wird sie im folgenden normalisiert, indem man sie durch das Maximum teilt und mit einem Skalar multipliziert, bevor man sie plottet. So erkennen wir die Broker besser.
plot(jg21_preentry1,
     layout=layout_with_kk,
     vertex.label.cex = .15, 
     vertex.label.color = "black", 
     vertex.label.family="sans",
     edge.arrow.size=.00001,
     edge.arrow.color="white",
     asp=0,
     vertex.size = V(jg21_preentry1)$betweenness/max(V(jg21_preentry1)$betweenness) * 10,
     main="Broker im Netzwerk des #30u30-Jahrgangs 2021",
     sub="Alle Stationen bis zum Aufnahmejahr (2021), 
     Knotengröße entspr. Betweenness-Wert und mit Faktor 10 skaliert")
```
# Vergleiche zwischen den Jahrgängen

## Die vernetztesten Mitglieder

Luisa Bisswanger, Lena Stork, Christoph Güttner, Judith Götter, Gertrud Kohl und Isabell Fries sind die Mitglieder aus den fünf Jahrgängen, die in ihrem Jahrgang jeweils den höchsten Outdegree-Wert, also die meisten Verbindungen haben. Aufgrund dieser Besonderheit soll im Folgenden ein Subgraph von deren Verbindungen generiert werden, um mögliche Parallelen zu analysieren.

Zunächst werden dazu die Ego-Netzwerke der Personen mit den höchsten Outdegrees erstellt, dann zu einem Graph zusammengefügt.

```{r Subgraph der Mitglieder mit höchsten Outdegrees}
lbisswanger <- subgraph <- make_ego_graph(members, order = 1,  c("Luisa Bisswanger"))
lstork <- subgraph <- make_ego_graph(members, order = 1,  c("Lena Stork"))
cguettner <- subgraph <- make_ego_graph(members, order = 1,  c("Christoph Güttner"))
jgoetter <- subgraph <- make_ego_graph(members, order = 1,  c("Judith Götter"))
gkohl <- subgraph <- make_ego_graph(members, order = 1,  c("Gertrud Kohl"))
ifries <- subgraph <- make_ego_graph(members, order = 1,  c("Isabell Fries"))

# Zu einem Graph zusammenfügen, indem die Ego-Netzwerke vom Gesamtnetzwerk doppelt subtrahiert (im Umkehrschluss addiert) werden
topoutdegree <- members - (members - lbisswanger[[1]] - lstork[[1]] - cguettner[[1]] - jgoetter[[1]] - gkohl[[1]] - ifries[[1]]) 

topoutdegree

topoutdegree1 <- delete_vertices (topoutdegree, V(topoutdegree)[degree(topoutdegree, mode="all")=="0"]) # Isolierte Knoten löschen

ind_topoutd <- degree(topoutdegree1, mode="in")

topoutdegree2 <- simplify(topoutdegree1, remove.multiple = TRUE) # Kantenvereinfachung
```

Visualisierungsanpassung: Zur besseren Unterscheidung werden die Mitglieder der einzelnen Jahrgänge verschieden eingefärbt.

```{r Einfärben der Knoten je nach Jahrgang}
V(topoutdegree2)[V(topoutdegree2)$entry == 2017]$color <- "firebrick1" # JG 17 wird gelb
V(topoutdegree2)[V(topoutdegree2)$entry == 2018]$color <- "firebrick2" # JG 18 wird orange
V(topoutdegree2)[V(topoutdegree2)$entry == 2019]$color <- "firebrick3" # JG 19 wird dunkelorange
V(topoutdegree2)[V(topoutdegree2)$entry == 2020]$color <- "firebrick" # JG 20 wird rot
V(topoutdegree2)[V(topoutdegree2)$entry == 2021]$color <- "firebrick4" # JG 21 wird dunkelrot
```

Erstellt den kombinierten Graphen der vernetztesten Mitglieder.

```{r Plot der vernetztesten Mitglieder}
plot(topoutdegree2,
     layout=layout_with_kk,
     edge.curved=curve_multiple(topoutdegree2), # verhindert, dass sich Kanten überlagern
     vertex.frame.color="white",
     edge.arrow.size=.0001,
     edge.arrow.color="white",
     vertex.size=ind_topoutd*.8,
     edge.color="grey",
     edge.curved=.2,
     vertex.label.degree=0, 
     vertex.label.cex=.15,
     vertex.label.family="sans",
     vertex.label.font=2,
     vertex.label.color="black",
     main="Die #30u30-Mitglieder der Jahrgänge 2017 bis 2021
     mit den meisten ausgehenden Verbindungen",
     sub="Knotengröße entspr. dem Indegree-Wert und mit Faktor 0,8 skaliert",
     asp=0)
```

## Alle Jahrgänge im Vergleich: Die populärsten & zentralsten Organisationen bis zum Aufnahmjahr

Nachdem bereits betrachtet wurde, welche Organisationen in den einzelnen Jahrgängen hohe Indegree- oder Vetweennness-Werte aufweisen, soll nun ein Subgraph erstellt werden, in dem die Netzwerke aller Jahrgänge zum Zeitpunkt der Aufnahme (inkl. aller organisat. Verflechtungen bis zum Aufnahmejahr) dargestellt werden. So erkennen wir, was die Jahrgänge untereinander verbindet.

Zunächst werden die Netzwerke der einzelnen Jahrgänge bis zum jeweiligen Aufnahmejahr zu einem Graphen zusammengefügt.

```{r Zusammenfügen der Jahrgangsnetzwerke bis zum jew. Aufnahmejahr}
# Dazu werden die zuvor erstellten Teilnetzwerke vom Gesamtnetzwerk der #30u30-Mitglieder doppelt subtrahiert (= also addiert):
members_preentry <- members - (members - jg17_preentry - jg18_preentry - jg19_preentry - jg20_preentry - jg21_preentry) 

members_preentry

members_preentry1 <- delete_vertices (members_preentry, V(members_preentry)[degree(members_preentry, mode="all")=="0"]) # Isolierte Nodes löschen

members_preentry2 <- simplify(members_preentry1, remove.multiple = TRUE) # Kantenvereinfachung
```

Wie viele Organisationen enthält das kombinierte Netzwerk aller Jahrgänge mit deren Verbindungen bis zum Aufnahmejahr?

```{r Jahrgangs-Verbindungen bis zur Aufnahme: Anzahl Organisationen}
mem_pe_org <- V(members_preentry2)$type == 2
sum(mem_pe_org, na.rm=T) # Ergebnis: 770

# zusätzlicher Check: Sind alle #30u30 im Netzwerk enthalten? Abfrage: Wie viele Knoten des Type 1 (Personen) sind im Netzwerk enthalten?
mem_pe <- V(members_preentry2)$type == 1
sum(mem_pe, na.rm=T) # Ergebnis: 150 - alle Mitglieder der 5 Jahrgänge sind enthalten
```
Um welche Art von Organisation handelt es sich bei den 770 Organisationen? Wie viele Unternehmen, Agenturen etc. enthält das Netzwerk?

```{r Anzahl der verschiedenen Organisationstypen}
# Wie viele Unternehmen sind im Netzwerk enthalten?
all_org1 <- V(members_preentry2)$category == 1
sum(all_org1, na.rm=T) # Ergebnis: 186

# Wie viele Agenturen sind im Netzwerk enthalten?
all_org2 <- V(members_preentry2)$category == 2
sum(all_org2, na.rm=T) # Ergebnis: 168

# Wie viele NGOs/NPOs sind im Netzwerk enthalten?
all_org3 <- V(members_preentry2)$category == 3
sum(all_org3, na.rm=T) # Ergebnis: 13

# Wie viele Polit. Organisationen sind im Netzwerk enthalten?
all_org4 <- V(members_preentry2)$category == 4
sum(all_org4, na.rm=T) # Ergebnis: 28

# Wie viele Hochschulen sind im Netzwerk enthalten?
all_org5 <- V(members_preentry2)$category == 5
sum(all_org5, na.rm=T) # Ergebnis: 190

# Wie viele Vereine / Verbände sind im Netzwerk enthalten?
all_org6 <- V(members_preentry2)$category == 6
sum(all_org6, na.rm=T) # Ergebnis: 30

# Wie viele Stiftungen sind im Netzwerk enthalten?
all_org7 <- V(members_preentry2)$category == 7
sum(all_org7, na.rm=T) # Ergebnis: 25

# Wie viele Medienunternehmen sind im Netzwerk enthalten?
all_org8 <- V(members_preentry2)$category == 8
sum(all_org8, na.rm=T) # Ergebnis: 90

# Wie viele sonstige Org. sind im Netzwerk enthalten?
all_org9 <- V(members_preentry2)$category == 9
sum(all_org9, na.rm=T) # Ergebnis: 18

# Wie viele Forschungsinstitutionen sind im Netzwerk enthalten?
all_org10 <- V(members_preentry2)$category == 10
sum(all_org10, na.rm=T) # Ergebnis: 22
```
### Indegree-Verteilung bis zum Aufnahmejahr bei allen Jahrgängen

Abfrage der Indegree-Werte um herauszufinden: Welche Organisationen sind jahrgangsübergreifend am populärsten?

```{r Indegree-Verteilung}
ind_mem_pe <- degree(members_preentry2, mode="in")
ind_mem_pe
# Sortiert die Indegree-Werte nach Größe (absteigend)
sort(ind_mem_pe, decreasing = T)
```

Bevor geplottet und die Indegree-Verteilung visualisert wird, werde einige Visualisierungsanpassungen vorgenommen. Der Übersicht halber wird das Label nur bei Knoten des Type 2 (Organisationen) angezeigt, da hier vor allem interessant ist, welche Organisationen populär sind - die Mitglieder eher weniger.

```{r Visualisierungsanpassung}
V(members_preentry2)$label <- ifelse(V(members_preentry2)$type==2, V(members_preentry2)$name, NA) # Labels nur bei Organisationen anzeigen

V(members_preentry2)[mentor==1]$color <- "firebrick1" # färbt Knoten der #30u30 rot
```
Erstellt den Plot, in dem die Indegree-Verteilung visualisiert wird, indem Knoten entsprechend ihres Indegree-Werts unterschiedlich groß dargestellt werden. Das Label wird zudem nur bei Knoten ab einem best. Indegree-Wert angezeigt, um zentrale Knoten besser zu erkennen.

```{r Plot: Indegree-Verteilung - Knotenbeschriftung bei indegree > 4}
plot(members_preentry2,
     layout=layout_with_fr,
     edge.curved=curve_multiple(members_preentry2), # verhindert, dass sich Kanten überlagern
     vertex.frame.color="white",
     edge.arrow.size=.0001,
     edge.arrow.color="white",
     vertex.size=ind_mem_pe*.4,
     vertex.label=ifelse(ind_mem_pe>4, V(members_preentry2)$name, NA), # Label wird nur bei Knoten mit einem indegree-Wert von über 4 angezeigt
     edge.color="grey",
     edge.curved=.2,
     vertex.label.degree=.8, 
     vertex.label.cex=.15,
     vertex.label.family="sans",
     vertex.label.font=2,
     vertex.label.dist=.2,
     vertex.label.color="black",
     asp=0,
     main="Populäre Organisationen in den #30u30-Jahrgängen 2017 - 2021",
     sub="Verbindungen bis zum jew. Aufnahmejahr,
     Knotengröße entspr. dem Indegree-Wert und mit Faktor 0,4 skaliert")
```
Um sich weiter auf populäre Organisationen zu fokussieren, wird erneut geplottet und nur bei Knoten mit einem Indegree-Wert von über 7 (die mehr als 7 Personen verbinden) das Knotenlabel angezeigt.

```{r Plot: Indegree-Verteilung - Knotenbeschriftung bei indegree > 7}

plot(members_preentry2,
     layout=layout_with_fr,
     edge.curved=curve_multiple(members_preentry2), # verhindert, dass sich Kanten überlagern
     vertex.frame.color="white",
     edge.arrow.size=.0001,
     edge.arrow.color="white",
     vertex.size=ind_mem_pe*.4,
     vertex.label=ifelse(ind_mem_pe>7, V(members_preentry2)$name, NA), #Vertex-Label wird nur angezeigt, wenn indegree > 7
     edge.color="grey",
     edge.curved=.2,
     vertex.label.degree=.8, 
     vertex.label.cex=.15,
     vertex.label.family="sans",
     vertex.label.font=2,
     vertex.label.dist=.2,
     vertex.label.color="black",
     asp=0,
     main="Populäre Organisationen in den #30u30-Jahrgängen 2017 - 2021",
     sub="Verbindungen bis zum jew. Aufnahmejahr,
     Knotengröße entspr. dem Indegree-Wert und mit Faktor 0,4 skaliert")

```

### Weitere Berechnungen
Vergleichend zu den vorherigen Berechnungen werden nun Maße wie Dichte, Pfaddistanz etc. für das Netzwerk bestehend aus den Verbindungen der Jahrgänge bis zum jeweiligen Aufnahmejahr erstellt.

```{r Alle Jahrgänge - Bis Aufnahme - Dichte}
edge_density(members_preentry2) # 0,001379098 (= 0,138 %)
```
Wie lang ist die mittlere Pfaddistanz und wie weit sind die am weistesten entfernten Knoten voneinander entfernt?

```{r Alle Jahrgänge - Bis Aufnahme - Pfaddistanz & Diameter}
# Mittlere Pfaddistanz berechnen:
mean_distance(members_preentry2, directed=FALSE) # Ergebnis: 6,117697

# Durchmesser / Diameter berechnen
diameter(members_preentry2, directed=FALSE) # 12 Schritte
farthest_vertices(members_preentry2, directed=FALSE) # zwischen 3M und Stipendium der Akademie für emotionale Intelligenz
```
### Clusteranalyse
Aus wie vielen Komponenten, Clustern, Communities besteht das Netzwerk der Jahrgänge, wenn man alle Verbindungen bis zum jeweiligen Aufnahmejahr betrachtet?

```{r Alle Jahrgänge - Bis Aufnahme - Clusteranalyse}
members_preentry2

gc_mem_pe <- cluster_walktrap(members_preentry2)

# Berechne Modularität
modularity(gc_mem_pe) # 0,6857561

membership(gc_mem_pe)
par(mfrow=c(1,1), mar=c(0,0,1,2))

# Liefert Anzahl und Aufteilung der Komponenten
components(members_preentry2)

#Visualisierung der Cluster
plot(gc_mem_pe, 
     members_preentry2, 
     edge.arrow.size=0.0001,
     edge.arrow.color="white",
     main="Clusteranalyse der #30u30-Jahrgänge 2017 bis 2021",
     sub="Alle Stationen bis zum jew. Aufnahmejahr",
     edge.curved=curve_multiple(members_preentry2),
     vertex.frame.color="black",
     edge.color="grey",
     edge.curved=.2,
     vertex.size=3,
     vertex.label=NA)

```
### Betweenness / Broker zwischen den Jahrgängen

Welche Brücken bestehen zwischen den Jahrgängen? Welche Organisationen verbinden Knoten, die sonst keine Verbindung hätten?

```{r Alle Jahrgänge - Bis Aufnahme - Betweenness-Werte}
betweenness(members_preentry2)
centr_betw(members_preentry2, directed=TRUE)
betw_all_pe <- betweenness(members_preentry2)
sort(betw_all_pe)
```

Bevor die Betweenness-Werte / Broker visualisiert werden, werden einige Visualisierungsanpassungen vorgenommen, sodass u. a. Labels nur bei Organisationen ab einem bestimmten Betweenness-Wert angezeigt werden, um den Fokus auf zentrale Broker zu legen.

```{r Alle Jahrgänge - Bis Aufnahme - Betweenness - Visualisierungsanpassung:}
V(members_preentry2)[mentor==1]$color <- "firebrick1" # färbt Knoten der #30u30 rot

# Wir können die Ausgabe von betweenness()  einer Variablen im Netz zuordnen und die Knoten entsprechend dimensionieren.
V(members_preentry2)$betweenness <- betweenness(members_preentry2)

# Labelanpassung
V(members_preentry2)$label <- ifelse(V(members_preentry2)$type==2, V(members_preentry2)$name, NA) # Labels nur bei Organisationen anzeigen
V(members_preentry2)$label <- ifelse(V(members_preentry2)$betweenness > 60000, V(members_preentry2)$label, NA) # Labels nur bei Knoten mit Betweenness-Wert über 60000 anzeigen
```

Erstellt den Plot, in dem Broker hervorgehoben werden.

```{r Plot: Broker im Gesamtnetzwerk der Jahrgänge bis zur Aufnahme}
plot(members_preentry2, 
     vertex.label.cex = .6, 
     vertex.label.color = "black", 
     vertex.size = V(members_preentry2)$betweenness) # Größe angepasst an betweenness

# Da die  Betweenness-Zentralität sehr groß sein kann, wird sie im folgenden normalisiert, indem man sie durch das Maximum teilt und mit einem Skalar multipliziert, bevor man sie plottet. So erkennen wir die Broker besser.
plot(members_preentry2,
     layout=layout_with_fr,
     vertex.label.cex = .3, 
     vertex.label.color = "black", 
     vertex.label.family="sans",
     vertex.label.degree=.2,
     edge.arrow.size=.00001,
     edge.arrow.color="white",
     asp=0,
     vertex.size = V(members_preentry2)$betweenness/max(V(members_preentry2)$betweenness) * 10,
     main="Die zentralen Broker in den #30u30-Jahrgängen 2017 bis 2021",
     sub="Alle Verbindungen bis zum jew. Aufnahmejahr,
     Knotengröße entspr. Betweenness-Wert und mit Faktor 10 skaliert ")
```
# Einzelne Teilnetzwerke

Im folgenden werden einzelne Teilnetzwerke erstellt, in denen nicht nach Jahrgänge getrennt wird, sondern beispielsweise auf berufliche Verbindungen, die Ausbildungsorte oder die Netzwerke derer "gezoomt", die in PR-Initiativen aktiv waren oder Stipendiat:innen waren.

Zunächst werden die beruflichen Stationen in den Fokus genommen.
## Berufliche Stationen: Wo haben die Talente gearbeitet?

Zunächst werden alle Arbeitsbeziehungen der fünf Jahrgänge bis März 2022 betrachtet.

```{r Berufliche Stationen - Bis März 2022: Teilnetzwerk erstellen}
members # ruft das Netzwerk der #30u30-Mitglieder auf

e7 <- delete.edges(members, E(members)[relationship < " 7"]) # löscht alle Kanten mit relationship < 7
e8 <- delete.edges(e7, E(e7)[relationship > "10"]) # löscht alle Kanten mit relationship > 10, sodass die Kanten übrig bleiben, bei denen es sich um Arbeitsbeziehungen handelt

work <- delete.vertices(e8, V(e8)[degree(e8, V(e8), mode="all")==0]) # löscht isolierte Knoten

work_simple <- simplify(work, remove.multiple = T) # Kantenvereinfachung
```

Wie viele eingehende Verbindungen weisen die Organisationen jeweils auf - d. h. wo haben wie viele Personen gearbeitet?

```{r Berufliche Stationen - Bis März 2022: Indegree-Verteilung}
ind_work <- degree(work_simple, V(work_simple), mode="in")
sort(ind_work, decreasing=T)
```

Von welchen Personen gehen besonders viele Verbindungen aus - d. h. wer hat viele berufliche Stationen absolviert?

```{r Berufliche Stationen - Bis März 2022: Outdegree-Verteilung}
outd_work <- degree(work_simple, V(work_simple), mode="out")
sort(outd_work, decreasing=T)

# Mittelwert des Outdegree-Werts - durchschn. Anzahl an Arbeitgebern?
mean(degree(work_simple,v=V(work_simple)[mentor == 1], mode="out")) 
  # 5,686667
```
Generiert den Plot aller Arbeitsbeziehungen der fünf Jahrgänge bis einschl. März 2022. Zur Hervorhebung populärer Organisationen wird das Label nur bei Knoten mit Indegree > 4 angezeigt.

```{r Berufliche Stationen - Bis März 2022: Plot}
plot(work_simple,
     layout=layout_with_kk,
     edge.curved=curve_multiple(work_simple),
     vertex.size=ind_work*.6, # Knotengröße entspr. des indegree-Werts skaliert
     vertex.frame.color="white",
     edge.color="grey",
     edge.curved=.2,
     edge.arrow.size=.00001,
     edge.arrow.color="white",
     vertex.label=ifelse(ind_work>4, V(work_simple)$name, NA),
     vertex.label.degree=0, 
     vertex.label.cex=.25,
     vertex.label.family="sans",
     vertex.label.font=2,
     vertex.label.color="black", 
     asp=0,
     main="Berufliche Stationen der #30u30-Mitglieder der Jahrgänge 2017 - 2021",
     sub="Alle Arbeitsstellen bis März 2022,
     Knotengröße entspr. Indegree-Wert und mit Faktor 0,6 skaliert")
```
Gleicher Plot / Visualisierung, nur ohne Labels:
```{r Berufliche Stationen: Plot - Bis März 2022 - ohne Labels}
plot(work_simple,
     layout=layout_with_kk,
     edge.curved=curve_multiple(work_simple),
     vertex.size=ind_work*.6, # Knotengröße entspr. des indegree-Werts skaliert
     vertex.frame.color="white",
     edge.color="grey",
     edge.curved=.2,
     edge.arrow.size=.00001,
     edge.arrow.color="white",
     vertex.label=NA, 
     asp=0,
     main="Arbeitsstellen der #30u30-Mitglieder der JG 2017 bis 2021",
     sub="Bis März 2022")
```

Wie viele Komponenten, Cluster, Communities finden sich im Arbeitsnetzwerk der fünf Jahrgänge bis März 2022?

```{r Berufliche Stationen - Bis März 2022: Cluster-Analyse}
clusters(work_simple)
cl_work <- cluster_walktrap(work_simple)
  
membership(cl_work)
modularity(cl_work) # 0,7288861
communities(cl_work) # 93

components(work_simple) # 18 components
size_cl_work <- sizes(cl_work)
sort(size_cl_work)

#Visualisierung der Cluster
plot(cl_work, 
     work_simple, 
     edge.arrow.size=0.0001,
     edge.arrow.color="white",
     main="Clusteranalyse der #30u30",
     sub="Arbeitsstellen",
     edge.curved=curve_multiple(work_simple),
     vertex.frame.color="black",
     edge.arrow.size=.2, 
     edge.color="grey",
     edge.curved=.2,
     vertex.size=3,
     vertex.label=NA)

```

#### Berechnungen für das Arbeitsnetzwerk bis März 2022

1. Wie dicht ist das Netzwerk?

```{r Berufliche Stationen - Bis März 2022: Dichte}
edge_density(work_simple) # 0,001467133 (= 0, 146 %)
```

2. Wie lang ist die mittlere Pfaddistanz und wie breit der Netzwerkdurchmesser?
```{r Berufliche Stationen - Bis März 2022: Pfaddistanz & Diameter}
mean_distance(work_simple, directed=F) # 6,803058
diameter(work_simple, directed=F) # 14
farthest_vertices(work_simple, directed=F) # 1und1 & Berkeley Kommunikation
```

Betweenness-Werte im Arbeitsnetzwerk bis März 2022: Wer sind insgesamt die Broker?

1. Berechne Betweenness-Werte:
```{r Berufliche Stationen - Bis März 2022: Betweennness}
# Berechne Betweenness-Zentralitätsmaße
betweenness(work_simple)
centr_betw(work_simple, directed=TRUE)
betw_work <- betweenness(work_simple)
sort(betw_work, decreasing = T)
```
2. Erstelle den Plot, in dem Knoten mit hohem Betweenness-Wett größer visualisiert werden und das Knoten-Label erst ab einem best. Betweenness-Wert angezeigt wird:
```{r Berufliche Stationen - Bis März 2022: Betweennness-Plot}
# Ausgabe von betweenness()  einer Variablen im Netz zuordnen und die Knoten entsprechend dimensionieren.
V(work_simple)$betweenness <- betweenness(work_simple)

V(work_simple)[type == 1]$color <- "firebrick"

# Erste, einfache Visualisierung:
plot(work_simple, 
     vertex.label.cex = .6, 
     vertex.label.color = "black", 
     vertex.size = V(work_simple)$betweenness) # Größe angepasst an betweenness

# Da die Betweenness-Zentralität sehr groß sein kann, wird sie im folgenden normalisiert, indem man sie durch das Maximum teilt und mit einem Skalar multipliziert, bevor man sie plottet. So erkennen wir die Broker besser.
plot(work_simple,
     layout=layout_with_kk,
     edge.curved=curve_multiple(work_simple),
     vertex.label=ifelse(betw_work>30000, V(work_simple)$name, NA),
     vertex.label.cex = .15, 
     vertex.frame.color="white",
     vertex.label.color = "black", 
     vertex.label.family="sans",
     edge.arrow.size=.00001,
     edge.arrow.color="white",
     asp=0,
     vertex.size = V(work_simple)$betweenness/max(V(work_simple)$betweenness) * 5,
     main="Die Broker im Arbeitsnetzwerk der #30u30-Jahrgänge 2017 - 2021",
     sub="Alle Verbindungen bis März 2022, 
     Knotengröße entspr. Indegree-Wert und mit Faktor 5 skaliert")
```

### Verschiedene Beschäftigungsarten

Im Folgenden werden Teilnetzwerke der versch. Beschäftigungsarten erstellt (Praktika/Werkstudiumsstellen, Freelance-Jobs, Festanstellungen). Die Beschäftigungsart Volontariat wird nicht separat betrachtet, da nur 4 / 150 Personen ein Volontariat gemacht haben und falls Überschneidungen bestehen, diese nicht signifikant für das gesamte Netzwerk sind.

#### Praktikumsstellen

Erstellt das Netzwerk der Praktikums- bzw. Werkstudiumsbeziehungen (diese werden zusammengefasst, da die Art der Stelle auf einem ähnlichen Level liegt).
```{r Teilnetzwerk Praktikumsstellen}

internship <- subgraph.edges(work, E(work)[relationship == " 7"]) # selektiert Kanten aus dem Arbeitsnetzwerk, bei denen es sich um Praktikums-/Werkstudiumsbzgh. handelt

internship <- delete_vertices (internship, V(internship)[degree(internship, mode="all")=="0"]) # löscht isolierte Knoten

internship1 <- simplify(internship, remove.multiple = TRUE) # Kantenvereinfachung
```

Bei welchen Arbeitgebern haben mehrere Personen Praktika gemacht oder als Werkstudent:in gearbeitet?

```{r Teilnetzwerk Praktikumsstellen: Indegree-Verteilung}
ind_internship <- degree(internship1, V(internship1), mode="in")
sort(ind_internship, decreasing=T)
```

Wie viele Praktika / Werkstudiumsjobs hatten die einzelnen Personen? Wer hat besonders viele verschiedene Stellen besetzt?
```{r Teilnetzwerk Praktikumsstellen: Outdegree-Verteilung}
outd_internship <- degree(internship1, V(internship1), mode="out")
sort(outd_internship, decreasing=T)

# Bei wie vielen Arbeitgebern haben die Personen im Schnitt Praktika gemacht oder als Werkstudierende gearbeitet (Durchschn. Outdegree-Wert=?
mean(degree(internship1,v=V(internship1)[type == 1], mode="out")) # 3,484848
```

Wie viele Mitglieder haben Praktika absolviert, auf wie viele und welche Art von Organisationen verteilen sie sich?
```{r Teilnetzwerk Praktikumsstellen: Knotentypen}
intern_ppl <- V(internship1)$type == 1
sum(intern_ppl, na.rm=T)  # 132 #30u30-Mitglieder haben Praktika gemacht

intern_org <- V(internship1)$type == 2
sum(intern_org, na.rm=T)  # bei insg. 347 Organisationen

intern_corp <- V(internship1)$category == 1
sum(intern_corp, na.rm=T)  # davon 116 Unternehmen

intern_agency <- V(internship1)$category == 2
sum(intern_agency, na.rm=T) # davon 91 Agenturen

intern_ngo <- V(internship1)$category == 3
sum(intern_ngo, na.rm=T)  # davon 9 NGOs/NPOs

intern_polit <- V(internship1)$category == 4
sum(intern_polit, na.rm=T) # davon 19 polit. Org.

intern_uni <- V(internship1)$category == 5
sum(intern_uni, na.rm=T) # davon 25 Hochschulen

intern_verein <- V(internship1)$category == 6
sum(intern_verein, na.rm=T)  # davon 4 Vereine

intern_stiftung <- V(internship1)$category == 7
sum(intern_stiftung, na.rm=T) # davon 2 Stiftungen

intern_media <- V(internship1)$category == 8
sum(intern_media, na.rm=T) # davon 57 Medienunternehmen

intern_sonst <- V(internship1)$category == 9
sum(intern_sonst, na.rm=T)  # davon 8 sonstige Org.

intern_forsch <- V(internship1)$category == 10
sum(intern_forsch, na.rm=T) # davon 16 Forschungsinstitute

# Wie viele #30u30-Partnerunternehmen waren Praktikumsstellen?
intern_spon <- V(internship1)$sponsor == 1
sum(intern_spon, na.rm=T)  # 6 Sponsoren (von insg. 15)
```

Wie hoch ist die Dichte im Praktikums-/Werkstudiumsnetzwerks?

```{r Teilnetzwerk Praktikumsstellen: Dichte}
edge_density(internship1) # 0,002009067 = 0,2%
```
Wie lang ist die mittlere Pfaddistanz und wie brei der Durchmesser des Praktikums-/Werkstudiumsnetzwerks?
```{r Teilnetzwerk Praktikumsstellen: Pfaddistanz & Diameter}
mean_distance(internship1, directed=F) # 6,812186
diameter(internship1, directed=F) # 14
```
#### Freelance-Stellen
Erstellt das Teilnetzwerk der Personen, die bei bestimmten Organisationen als Freelancer:in gearbeitet haben.
```{r Teilnetzwerk Freelance erstellen}
freelance <- subgraph.edges(work, E(work)[relationship == "10"]) # selektiert nur Kanten, bei denen es sich um Freelance-Beschäftigungen handelt

freelance <- delete_vertices (freelance, V(freelance)[degree(freelance, mode="all")=="0"]) # löscht isolierte Knoten

freelance1 <- simplify(freelance, remove.multiple = TRUE) # Kantenvereinfachung
```

Bei welchen Organisationen haben gleich mehrere Talente auf Freelancer-Basis gearbeitet? Wo gibt es Überschneidungen, wo nicht?
```{r Teilnetzwerk Freelance: Indegree-Verteilung}
ind_freelance <- degree(freelance1, V(freelance1), mode="in")
sort(ind_freelance, decreasing=T)
```

Welche Personen hatten mehrere Freelance-Stellen inne? (Anzahl der ausgehenden Verbindungen von den Personen)
```{r Teilnetzwerk Freelance: Outdegree-Verteilung}
outd_freelance <- degree(freelance1, V(freelance1), mode="out")
sort(outd_freelance, decreasing=T)
```

Wie viele Personen haben als Freelancer gearbeitet und bei wie vielen verschiedenen Organisationen? Um welche Organisationstypen handelt es sich dabei überwiegend?
```{r Teilnetzwerk Freelance: Knotentypen}
freelance_ppl <- V(freelance1)$type == 1
sum(freelance_ppl, na.rm=T)  # 51 #30u30-Mitglieder

freelance_org <- V(freelance1)$type == 2
sum(freelance_org, na.rm=T)  # 66 Organisationen

freelance_corp <- V(freelance1)$category == 1
sum(freelance_corp, na.rm=T)  # davon 10 Unternehmen

freelance_agency <- V(freelance1)$category == 2
sum(freelance_agency, na.rm=T)  # davon 8 Agenturen

freelance_ngo <- V(freelance1)$category == 3
sum(freelance_ngo, na.rm=T)  # davon 2 NGOs/NPOs

freelance_polit <- V(freelance1)$category == 4
sum(freelance_polit, na.rm=T)  # davon 2 polit. Org.

freelance_uni <- V(freelance1)$category == 5
sum(freelance_uni, na.rm=T)  # davon 1 Hochschule

freelance_verein <- V(freelance1)$category == 6
sum(freelance_verein, na.rm=T)  # davon 0 Vereine

freelance_stiftung <- V(freelance1)$category == 7
sum(freelance_stiftung, na.rm=T)  # davon 3 Stiftungen

freelance_media <- V(freelance1)$category == 8
sum(freelance_media, na.rm=T)  # davon 33 Medienunternehmen

freelance_sonst <- V(freelance1)$category == 9
sum(freelance_sonst, na.rm=T)  # davon 5 sonstige Org.

freelance_forsch <- V(freelance1)$category == 10
sum(freelance_forsch, na.rm=T)  # davon 2 Forschungsinstitute

# Wie viele #30u30-Partnerunternehmen sind im Freelance-Teilnetzwerk enthalten?
freelance_spon <- V(freelance1)$sponsor == 1
sum(freelance_spon, na.rm=T)  # 0 Sponsoren 
```
### Festanstellung
Zuletzt werden die Beziehungen betrachtet, bei denen es sich um Festanstellungen der Talente handelt.

Erstellt Teilnetzwerk der Festanstellungs-Beziehungen zwischen #30u30 und Organisationen:
```{r Teilnetzwerk Festanstellungen erstellen}
fulltime <- subgraph.edges(work, E(work)[relationship == " 9"]) # selektiert nur Kanten, bei denen es sich um Festanstellungen handelt

fulltime <- delete_vertices (fulltime, V(fulltime)[degree(fulltime, mode="all")=="0"]) # löscht isolierte Knoten

fulltime1 <- simplify(fulltime, remove.multiple = TRUE) # Kantenvereinfachung
```

Wie viele Personen sind im Teilnetzwerk enthalten und auf wie viele Organisationen verteilen sie sich? Um welche Organisationstypen handelt es sich?
```{r Teilnetzwerk Festanstellung: Knotentypen}
fulltime_ppl <- V(fulltime1)$type == 1
sum(fulltime_ppl, na.rm=T)  # 149 #30u30-Mitglieder

fulltime_org <- V(fulltime1)$type == 2
sum(fulltime_org, na.rm=T)  # 270 Organisationen

fulltime_corp <- V(fulltime1)$category == 1
sum(fulltime_corp, na.rm=T)  # 119 Unternehmen

fulltime_agency <- V(fulltime1)$category == 2
sum(fulltime_agency, na.rm=T)  # 110 Agenturen

fulltime_ngo <- V(fulltime1)$category == 3
sum(fulltime_ngo, na.rm=T)  # 5 NGOs/NPOs

fulltime_polit <- V(fulltime1)$category == 4
sum(fulltime_polit, na.rm=T)  # 10 polit. Org.

fulltime_uni <- V(fulltime1)$category == 5
sum(fulltime_uni, na.rm=T)  # 4 Hochschulen

fulltime_verein <- V(fulltime1)$category == 6
sum(fulltime_verein, na.rm=T)  # 1 Verein

fulltime_stiftung <- V(fulltime1)$category == 7
sum(fulltime_stiftung, na.rm=T)  # 2 Stiftungen

fulltime_media <- V(fulltime1)$category == 8
sum(fulltime_media, na.rm=T)  # 10 Medienunternehmen

fulltime_sonst <- V(fulltime1)$category == 9
sum(fulltime_sonst, na.rm=T)  # 3 sonstige Org.

fulltime_forsch <- V(fulltime1)$category == 10
sum(fulltime_forsch, na.rm=T)  # 6 Forschungsinstitute

# Wie viele #30u30-Partnerunternehmen haben sind im Festanstellungsnetzwerk enthalten?
fulltime_spon <- V(fulltime1)$sponsor == 1
sum(fulltime_spon, na.rm=T)  # 15 Sponsoren (von insg. 15)
```

```{r Teilnetzwerk Festanstellung: Indegree-Verteilung}
ind_ft <- degree(fulltime1, V(fulltime1), mode="in")
sort(ind_ft, decreasing=T)
```

Wie viele Beziehungen gehen von den einzelnen Personen aus - bei wie vielen Organisationen waren sie jeweils festangestellt?
```{r Teilnetzwerk Festanstellung: Outdegree-Verteilung}
outd_ft <- degree(fulltime1, V(fulltime1), mode="out")
sort(outd_ft, decreasing=T)

# Bei wie vielen Arbeitgebern waren die Personen im Schnitt fest angestellt?
mean(degree(fulltime1,v=V(fulltime1)[type == 1], mode="out")) # 2,288591
```
Wie hoch ist die Dichte im Teilnetzwerk der Festanstellungs-Beziehungen?
```{r Teilnetzwerk Festanstellung: Dichte}
edge_density(fulltime1) # 0,001946992 = 0,19%
```
Wie lang ist die Pfaddistanz und wie breit der Durchmesser des Teilnetzwerks der Festanstellungs-Beziehungen?
```{r Teilnetzwerk Festanstellung: Pfaddistanz & Diameter}
mean_distance(fulltime1, directed=F) # 6,610737
diameter(fulltime1, directed=F) # 16
```
### Arbeitsstellen bis zum Aufnahmejahr
Während bis hierhin alle Arbeitsstellen bis März 2022 berücksichtigt wurden, wird nun der Fokus auf die Arbeitsstellen bis zum jew. Aufnahmejahr gelegt, um zu sehen, wo sich die Stationen der Talente vor #30u30 überschneiden.

Erstellt Teilnetzwerk der Arbeitsstellen der 150 Personen bis zum jew. Aufnahmejahr:
```{r Berufliche Stationen: Bis Aufnahmejahr - Teilnetzwerk erstellen}
members_preentry1

pe7 <- delete.edges(members_preentry1, E(members_preentry1)[relationship < " 7"]) # löscht alle Kanten mit relationship < 7 
pe8 <- delete.edges(pe7, E(pe7)[relationship > 10]) # löscht alle Kanten mit relationship > 10, sodass die Kanten übrig bleiben, bei denen es sich um Arbeitsbeziehungen handelt

work_pe <- delete.vertices(pe8, V(pe8)[degree(pe8, mode="in")] == 0) # löscht isolates
```

#### Vergleich der Jahrgänge: Stelle bei Aufnahme und heute
Nun werden für die fünf Jahrgänge jeweils die Arbeitgeber im Aufnahmejahr mit den Arbeitgebern im März 2022 verglichen, in dem beide Zeitpunkte visualisiert werden.

##### Jahrgang 2017
1. Teilnetzwerk der Arbeitsstellen des Jahrgangs 2017 im zum Aufnahmejahr
```{r Berufliche Stationen - Jahrgang 2017 - Aufnahmejahr}
work

work17 <- delete.vertices(work, V(work)[(type == 1) & (entry != 2017)]) #löscht alle Mitglieder außer den #30u30-Jahrgang 2017

work17_entry <- subgraph.edges(work17, E(work17)[year == "2017"]) # selektiert nur Arbeitsbeziehungen im Jahr 2017

V(work17_entry)[mentor==1]$color <- "firebrick1" # färbt Knoten der #30u30 rot

# Generiert den Plot
plot(work17_entry,
     layout=layout_with_kk,
     edge.curved=curve_multiple(work17_entry),
     vertex.size=2,
     vertex.frame.color="white",
     edge.color="grey",
     edge.curved=.2,
     edge.arrow.size=.00001,
     edge.arrow.color="white",
     vertex.label.degree=0, 
     vertex.label.cex=.3,
     vertex.label.family="sans",
     vertex.label.font=2,
     vertex.label.color="black", 
     asp=0,
     main="Arbeitsstellen der #30u30-Mitglieder des Jahrgangs 2017",
     sub="Im Jahr der Aufnahme (2017)")
```

2. Teilnetzwerk der Arbeitsbeziehungen des Jahrgangs 2017 im Jahr 2022 
```{r Berufliche Stationen - Jahrgang 2017 - Stand 2022}
work17
work17_now <- subgraph.edges(work17, E(work17)[year == "2022"]) # selektiert nur Arbeitsbeziehungen im Jahr 2022

V(work17_now)[mentor==1]$color <- "firebrick1" # färbt Knoten der #30u30 rot

# Generiert den Plot
plot(work17_now,
     layout=layout_with_fr,
     edge.curved=curve_multiple(work17_now),
     vertex.size=2,
     vertex.frame.color="white",
     edge.color="grey",
     edge.curved=.2,
     edge.arrow.size=.00001,
     edge.arrow.color="white",
     vertex.label.degree=0, 
     vertex.label.cex=.3,
     vertex.label.family="sans",
     vertex.label.font=2,
     vertex.label.color="black", 
     asp=0,
     main="Arbeitsstellen der #30u30-Mitglieder des Jahrgangs 2017",
     sub="Im Jahr 2022 (Stand März 2022)")
```

##### Jahrgang 2018

1. Teilnetzwerk der Arbeitsstellen des Jahrgangs 2018 im Aufnahmejahr
```{r Berufliche Stationen - Jahrgang 2018 - Aufnahmejahr}
work
work18 <- delete.vertices(work, V(work)[(type == 1) & (entry != 2018)]) #löscht alle Mitglieder außer den #30u30-Jahrgang 2018

work18_entry <- subgraph.edges(work18, E(work18)[year == "2018"]) # selektiert nur Arbeitsbeziehungen im Jahr 2018

V(work18_entry)[mentor==1]$color <- "firebrick1" # färbt Knoten der #30u30 rot

# Generiert den Plot
plot(work18_entry,
     layout=layout_with_fr,
     edge.curved=curve_multiple(work18_entry),
     vertex.size=2,
     vertex.frame.color="white",
     edge.color="grey",
     edge.curved=.2,
     edge.arrow.size=.00001,
     edge.arrow.color="white",
     vertex.label.degree=0, 
     vertex.label.cex=.2,
     vertex.label.family="sans",
     vertex.label.font=2,
     vertex.label.color="black", 
     asp=0,
     main="Arbeitsstellen der #30u30-Mitglieder des Jahrgangs 2018",
     sub="Im Jahr der Aufnahme (2018)")
```

2. Teilnetzwerk der Arbeitsbeziehungen des Jahrgangs 2018 im Jahr 2022 
```{r Berufliche Stationen - Jahrgang 2018 - Stand 2022}
work18

work18_now <- subgraph.edges(work18, E(work18)[year == "2022"]) # selektiert nur Arbeitsbeziehungen im Jahr 2022

V(work18_now)[mentor==1]$color <- "firebrick1" # färbt Knoten der #30u30 rot

# Generiert den Plot
plot(work18_now,
     layout=layout_with_fr,
     edge.curved=curve_multiple(work18_now),
     vertex.size=2,
     vertex.frame.color="white",
     edge.color="grey",
     edge.curved=.2,
     edge.arrow.size=.00001,
     edge.arrow.color="white",
     vertex.label.degree=0, 
     vertex.label.cex=.3,
     vertex.label.family="sans",
     vertex.label.font=2,
     vertex.label.color="black", 
     asp=0,
     main="Arbeitsstellen der #30u30-Mitglieder des Jahrgangs 2018",
     sub="Im Jahr 2022 (Stand März 2022)")
```

##### Jahrgang 2019

1. Teilnetzwerk der Arbeitsbeziehungen des Jahrgangs 2019 im Aufnahmejahr 
```{r Berufliche Stationen - Jahrgang 2019 - Aufnahmejahr}
work
work19 <- delete.vertices(work, V(work)[(type == 1) & (entry != 2019)]) #löscht alle Mitglieder außer den #30u30-Jahrgang 2019

work19_entry <- subgraph.edges(work19, E(work19)[year == "2019"]) # selektiert nur Arbeitsbeziehungen im Jahr 2019

V(work19_entry)[mentor==1]$color <- "firebrick1" # färbt Knoten der #30u30 rot

# Generiert den Plot
plot(work19_entry,
     layout=layout_with_fr,
     edge.curved=curve_multiple(work19_entry),
     vertex.size=2,
     vertex.frame.color="white",
     edge.color="grey",
     edge.curved=.2,
     edge.arrow.size=.00001,
     edge.arrow.color="white",
     vertex.label.degree=0, 
     vertex.label.cex=.25,
     vertex.label.family="sans",
     vertex.label.font=2,
     vertex.label.color="black", 
     asp=0,
     main="Arbeitsstellen der #30u30-Mitglieder des Jahrgangs 2019",
     sub="Im Jahr der Aufnahme (2019)")
```

2. Teilnetzwerk der Arbeitsbeziehungen des Jahrgangs 2019 im Jahr 2022 
```{r Berufliche Stationen - Jahrgang 2019 - Stand 2022}
work19
work19_now <- subgraph.edges(work19, E(work19)[year == "2022"]) # selektiert nur Arbeitsbeziehungen im Jahr 2022

V(work19_now)[mentor==1]$color <- "firebrick1" # färbt Knoten der #30u30 rot

# Generiert den Plot
plot(work19_now,
     layout=layout_with_fr,
     edge.curved=curve_multiple(work19_now),
     vertex.size=2,
     vertex.frame.color="white",
     edge.color="grey",
     edge.curved=.2,
     edge.arrow.size=.00001,
     edge.arrow.color="white",
     vertex.label.degree=0, 
     vertex.label.cex=.2,
     vertex.label.family="sans",
     vertex.label.font=2,
     vertex.label.color="black", 
     asp=0,
     main="Arbeitsstellen der #30u30-Mitglieder des Jahrgangs 2019",
     sub="Im Jahr 2022 (Stand März 2022)")
```

##### Jahrgang 2020

1. Teilnetzwerk der Arbeitsbeziehungen des Jahrgangs 2020 im Aufnahmejahr 
```{r Berufliche Stationen - Jahrgang 2020 - Aufnahmejahr}
work
work20 <- delete.vertices(work, V(work)[(type == 1) & (entry != 2020)]) #löscht alle Mitglieder außer den #30u30-Jahrgang 2020

work20_entry <- subgraph.edges(work20, E(work20)[year == "2020"]) # selektiert nur Arbeitsbeziehungen im Jahr 2020

V(work20_entry)[mentor==1]$color <- "firebrick1" # färbt Knoten der #30u30 rot

# Generiert den Plot
plot(work20_entry,
     layout=layout_with_fr,
     edge.curved=curve_multiple(work20_entry),
     vertex.size=2,
     vertex.frame.color="white",
     edge.color="grey",
     edge.curved=.2,
     edge.arrow.size=.00001,
     edge.arrow.color="white",
     vertex.label.degree=0, 
     vertex.label.cex=.2,
     vertex.label.family="sans",
     vertex.label.font=2,
     vertex.label.color="black", 
     asp=0,
     main="Arbeitsstellen der #30u30-Mitglieder des Jahrgangs 2020",
     sub="Im Jahr der Aufnahme (2020)")
```

2. Teilnetzwerk der Arbeitsbeziehungen des Jahrgangs 2020 im Jahr 2022 
```{r Berufliche Stationen - Jahrgang 2020 - Stand 2022}
work20
work20_now <- subgraph.edges(work20, E(work20)[year == "2022"]) # selektiert nur Arbeitsbeziehungen im Jahr 2022

V(work20_now)[mentor==1]$color <- "firebrick1" # färbt Knoten der #30u30 rot

# Generiert den Plot
plot(work20_now,
     layout=layout_with_fr,
     edge.curved=curve_multiple(work20_now),
     vertex.size=2,
     vertex.frame.color="white",
     edge.color="grey",
     edge.curved=.2,
     edge.arrow.size=.00001,
     edge.arrow.color="white",
     vertex.label.degree=0, 
     vertex.label.cex=.2,
     vertex.label.family="sans",
     vertex.label.font=2,
     vertex.label.color="black", 
     asp=0,
     main="Arbeitsstellen der #30u30-Mitglieder des Jahrgangs 2020",
     sub="Im Jahr 2022 (Stand März 2022)")
```

##### Jahrgang 2021

1. Teilnetzwerk der Arbeitsbeziehungen des Jahrgangs 2021 im Aufnahmejahr 
```{r Berufliche Stationen - Jahrgang 2021 - Aufnahmejahr}
work
work21 <- delete.vertices(work, V(work)[(type == 1) & (entry != 2021)]) #löscht alle Mitglieder außer den #30u30-Jahrgang 2021

work21_entry <- subgraph.edges(work21, E(work21)[year == "2021"]) # selektiert nur Arbeitsbeziehungen im Jahr 2021

V(work21_entry)[mentor==1]$color <- "firebrick1" # färbt Knoten der #30u30 rot

# Generiert den Plot
plot(work21_entry,
     layout=layout_with_fr,
     edge.curved=curve_multiple(work21_entry),
     vertex.size=2,
     vertex.frame.color="white",
     edge.color="grey",
     edge.curved=.2,
     edge.arrow.size=.00001,
     edge.arrow.color="white",
     vertex.label.degree=0, 
     vertex.label.cex=.2,
     vertex.label.family="sans",
     vertex.label.font=2,
     vertex.label.color="black", 
     asp=0,
     main="Arbeitsstellen der #30u30-Mitglieder des Jahrgangs 2021",
     sub="Im Jahr der Aufnahme (2021)")
```

2. Teilnetzwerk der Arbeitsbeziehungen des Jahrgangs 2021 im Jahr 2022 
```{r Berufliche Stationen - Jahrgang 2021 - Stand 2022}
work21
work21_now <- subgraph.edges(work21, E(work21)[year == "2022"]) # selektiert nur Arbeitsbeziehungen im Jahr 2022

V(work21_now)[mentor==1]$color <- "firebrick1" # färbt Knoten der #30u30 rot

# Generiert den Plot
plot(work21_now,
     layout=layout_with_fr,
     edge.curved=curve_multiple(work21_now),
     vertex.size=2,
     vertex.frame.color="white",
     edge.color="grey",
     edge.curved=.2,
     edge.arrow.size=.00001,
     edge.arrow.color="white",
     vertex.label.degree=0, 
     vertex.label.cex=.2,
     vertex.label.family="sans",
     vertex.label.font=2,
     vertex.label.color="black", 
     asp=0,
     main="Arbeitsstellen der #30u30-Mitglieder des Jahrgangs 2021",
     sub="Im Jahr 2022 (Stand März 2022)")
```

## Journalistische Stationen
Nachdem berufliche Stationen insgesamt betrachtet wurden, wird nun speziell auf Beschäftigungen im journalistischen Bereich / in Medienunternehmen gezoomt. 

Das Verhältnis von Journalismus und PR ist ein zentrales, beide Disziplinen verbindet zudem ein gewisses Skillset wie redaktionelle Kompentenz.
Daher wird nun betrachtet, wie viele der #30u30-Mitglieder (Vor-)Erfahrungen im Journalismus aufweisen.

Dazu wird ein Teilnetzwerk erstellt, in dem nur Personen und Knoten, bei denne es sich um Medienunternehmen handelt, vorkommen.

```{r Journ. Stationen: Teilnetzwerk Medienunternehmen}
members

workinmedia <- delete.vertices(members, V(members)[(type == 2) & (category !=8)]) # löscht alle Organisationen außer Medienunternehmen

workinmedia <- delete.vertices(workinmedia, V(workinmedia)[degree(workinmedia, V(workinmedia), mode = "all")== 0]) # löscht Isolates
```

Die Mitglieder verschiedener Jahrgänge werden unterschiedlich eingefärbt.

```{r Journ. Stationen: Visualisierungsanpassung - Jahrgänge einfärben}
V(workinmedia)[V(workinmedia)$type == 1]$shape <- "circle"
V(workinmedia)[V(workinmedia)$entry == 2017]$color <- "firebrick1"
V(workinmedia)[V(workinmedia)$entry == 2018]$color <- "firebrick2" 
V(workinmedia)[V(workinmedia)$entry == 2019]$color <- "firebrick3" 
V(workinmedia)[V(workinmedia)$entry == 2020]$color <- "firebrick"
V(workinmedia)[V(workinmedia)$entry == 2021]$color <- "firebrick4" 
```

Je nach Art der Beschäftigung (Praktika / Werkstudium, Volontariat, Freelance oder Festanstellung) werden die Kanten verschieden eingefärbt.

```{r Journ. Stationen: Visualisierungsanpassung - Edges einfärben}
E(workinmedia)[E(workinmedia)$relationship == " 7"]$color <- "dodgerblue1"
E(workinmedia)[E(workinmedia)$relationship == " 8"]$color <- "palegreen3" 
E(workinmedia)[E(workinmedia)$relationship == " 9"]$color <- "dodgerblue4" 
E(workinmedia)[E(workinmedia)$relationship == "10"]$color <- "palegreen4"
```

Plottet das Netzwerk der #30u30-Mitglieder mit Arbeitsstationen in Medienunternehmen.

```{r Journ. Stationen: Plot}

E(workinmedia)$width <- 1

# Generiert den Plot des Teilnetzwerks
plot(workinmedia,
     layout=layout_with_kk,
     edge.curved=curve_multiple(workinmedia),
     vertex.frame.color="white",
     edge.curved=.2,
     edge.arrow.size=.00001,
     edge.arrow.color="white",
     edge.with=1,
     vertex.label.cex = .2,
     vertex.label.color="black",
     vertex.label.family="sans",
     vertex.size=3,
     asp=0,
     main="Arbeitsbeziehungen zwischen Medienunternehmen
     und #30u30-Mitgliedern der Jahrgänge 2017 - 2021")
```
Wie viele Medienunternehmen sind im Netzwerk enthalten und wie viele #30u30-Mitglieder der einzelnen Jahrgänge sind vertreten?

```{r Journ. Stationen: Knotenverteilung im Netzwerk}
# Wie viele Medienunternehmen sind im Netzwerk enthalten?
medienunternehmen <- V(workinmedia)$category == 8
sum(medienunternehmen, na.rm=T) # Ergebnis: 92

# Wie viele #30u30-Mitglieder der Jahrgänge 2017 bis 2021?
mediaworkers <- V(workinmedia)$type == 1
sum(mediaworkers, na.rm=T) # Ergebnis: 64

# Jahrgangsverteilung: Wie viele aus den einzelnen Jahrgängen haben in Medienunternehmen gearbeitet?

mediaworkers17 <- V(workinmedia)$entry == "2017"
sum(mediaworkers17)  # 12 Personen

mediaworkers18 <- V(workinmedia)$entry == "2018"
sum(mediaworkers18)  # 15 Personen

mediaworkers19 <- V(workinmedia)$entry == "2019"
sum(mediaworkers19)  # 12 Personen

mediaworkers20 <- V(workinmedia)$entry == "2020"
sum(mediaworkers20)  # 10 Personen

mediaworkers21 <- V(workinmedia)$entry == "2021"
sum(mediaworkers21)  # 15 Personen
```
Welches Anstellungsverhältnis dominiert? Haben die Personen eher Praktika oder Student:innen-Jobs oder eine feste Anstellung ausgeübt?

```{r Journ. Stationen: Beziehungsarten im Netzwerk}
E(workinmedia)
media_intern <- E(workinmedia)[relationship == " 7"] # zählt Kanten, bei denen es sich um Praktika / Studijobs handelt
sum(media_intern) # 15368

media_volo <- E(workinmedia)[relationship == " 8"]
sum(media_volo) # zählt Kanten, bei denen es sich um Volos handelt; Ergebnis: 1457

media_fulltime <- E(workinmedia)[relationship == " 9"]
sum(media_fulltime) # zählt Kanten, bei denen es sich um Festanstellungen handelt; Ergebnis: 4352

media_freelance <- E(workinmedia)[relationship == "10"]
sum(media_freelance) # zählt Kanten, bei denen es sich um Freelancejobs handelt; Ergebnis: 22812
```

Der Plot wird erneut mit vereinfachten Kanten generiert.

```{r Journ. Stationen: Plot mit Kantenvereinfachung}

workinmedia1 <- simplify(workinmedia, remove.multiple = T) # Kantenvereinfachung

plot(workinmedia1,
     layout=layout_with_kk,
     edge.curved=curve_multiple(workinmedia1),
     vertex.frame.color="white",
     edge.curved=.2,
     edge.arrow.size=.00001,
     edge.arrow.color="white",
     vertex.label.cex = .2,
     vertex.label.color="black",
     vertex.label.family="sans",
     vertex.size=3,
     asp=0,
     main="Arbeitsbeziehungen zwischen Medienunternehmen
     und #30u30-Mitgliedern der Jahrgänge 2017 - 2021")
```

Wie viele Komponenten, Cluster und Communities enthält das Teilnetzwerk?

```{r Journ. Stationen: Cluster-Analyse}
clusters(workinmedia1)
cl_media <- cluster_walktrap(workinmedia1)
  
membership(cl_media)
communities(cl_media)
size_cl_media <- sizes(cl_media)
sort(size_cl_media)

# Berechne Modularität
modularity(cl_media) # 0,8657078

# Liste Components auf
components(workinmedia) # 29 components

#Visualisierung der Cluster
plot(cl_media, 
     workinmedia1, 
     edge.arrow.size=0.0001,
     edge.arrow.color="white",
     main="Clusteranalyse der #30u30 der Jahrgänge 2017 - 2021
     mit berufichen Stationen in Medienunternehmen",
     edge.curved=curve_multiple(workinmedia1),
     vertex.frame.color="black",
     edge.color="grey",
     edge.curved=.2,
     vertex.size=3,
     vertex.label.family="sans",
     vertex.label.color="black",
     vertex.label.cex=.2)

```
Welches ist die größte Komponente im Netzwerk und welche Knoten enthält sie?

```{r Journ. Stationen: Größte Komponente im Netzwerk - Plot}
media_comp <- decompose(workinmedia1)
media_comp
# Selektiert die größte Komponente mit insg. 63 Knoten
media_comp[[2]]

ind_media_comp <- degree(media_comp[[2]], mode="in")

# Generiert den Plot der größten Komponente
plot(media_comp[[2]],
     layout=layout_with_kk,
     vertex.label.cex=.2,
     vertex.label.color="black",
     vertex.label.family="sans",
     vertex.size=3,
     vertex.frame.color=NA,
     edge.arrow.size=.000001,
     edge.arrow.color="white",
     asp=0,
     main="Größte Komponente im Netzwerk der #30u30 
     der Jahrgänge 2017 - 2021 mit Arbeitsstationen in Medienunternehmen",
     sub="Anzahl der Knoten: n = 63")

# Plot Mit Indegree-Skalierung der Knoten
plot(media_comp[[2]],
     layout=layout_with_kk,
     edge.curved=curve_multiple(media_comp[[2]]),
     vertex.label.cex=.2,
     vertex.label.color="black",
     vertex.label.family="sans",
     vertex.size=ind_media_comp*1.2,
     edge.arrow.size=.000001,
     edge.arrow.color="white",
     asp=0,
     main="Größte Komponente im Netzwerk der #30u30 
     der Jahrgänge 2017 - 2021 mit Arbeitsstationen in Medienunternehmen",
     sub="Anzahl der Knoten: n = 63")

# Wie viele Organisationen enthält die Komponente?
mediacomp2_org <- V(media_comp[[2]])$type == 2
sum(mediacomp2_org)  # 37 Organisationen

# Wie viele Personen enthält die Komponente?
mediacomp2_org <- V(media_comp[[2]])$type == 1
sum(mediacomp2_org)  # 26 Organisationen

# Berechnungen
edge_density(media_comp[[2]]) # 0,01587302 = 1,58 %
mean_distance(media_comp[[2]], directed=F) # 6,173067
diameter(media_comp[[2]], directed = F) # 16
# im Vergleich zu Netzwerk aller Medienunternehmen:
mean_distance(workinmedia1, directed=F) # 5,83497
diameter(workinmedia1, directed=F) # 16
edge_density(workinmedia1) # 0,005252275 = 0,52 %

# Clusteranalyse
clusters(media_comp[[2]])
cl_media_comp <- cluster_walktrap(media_comp[[2]])
  
membership(cl_media_comp)

# Modularität in der Komponente
modularity(cl_media_comp) # 0.7147503
```

```{r Journ. Stationen: Indegree-Verteilung}
ind_media <- degree(workinmedia1, mode="in")
sort(ind_media)
```

Die populärsten Medienunternehmen werden hervorgehoben, indem die Knotengröße ja nach Indegree-Wert skaliert wird.

```{r Journ. Stationen: Indegree-skalierter Plot}
plot(workinmedia1,
     layout=layout_with_kk,
     edge.curved=curve_multiple(workinmedia1),
     vertex.frame.color="white",
     #edge.color="grey",
     edge.curved=.2,
     edge.arrow.size=.00001,
     edge.arrow.color="white",
     vertex.size=ind_media*1.2,
     vertex.label.cex = .2,
     vertex.label.color="black",
     vertex.label.family="sans",
     asp=0,
     main="Arbeitsbeziehungen zwischen Medienunternehmen
     und #30u30-Mitgliedern der Jahrgänge 2017 - 2021",
     sub="Knotengröße entspr. dem Indegree-Wert und mit Faktor 1,2 skaliert")
```

Betweenness-Werte im Netzwerk der Medienunternehmen und dort beschäftigten #30u30-Mitgliedern: Wer sind die Broker?

```{r Journ. Stationen: Betweenness}
betweenness(workinmedia1)
centr_betw(workinmedia1, directed=TRUE)
betw_media <- betweenness(workinmedia1)
sort(betw_media, decreasing = T)

# Ausgabe von betweenness()  einer Variablen im Netz zuordnen und die Knoten entsprechend dimensionieren:
V(workinmedia1)$betweenness <- betweenness(workinmedia1)

plot(workinmedia1, 
     vertex.label.cex = .6, 
     vertex.label.color = "black", 
     vertex.size = V(workinmedia1)$betweenness) # Größe angepasst an betweenness

# Da die  Betweenness-Zentralität sehr groß sein kann, wird sie im folgenden normalisiert, indem man sie durch das Maximum teilt und mit einem Skalar multipliziert, bevor man sie plottet. So erkennen wir die Broker besser.
plot(workinmedia1,
     layout=layout_with_kk,
     edge.curved=curve_multiple(workinmedia1),
     vertex.label.cex = .15, 
     vertex.label.color = "black", 
     vertex.label.family="sans",
     edge.arrow.size=.00001,
     edge.arrow.color="white",
     asp=0,
     vertex.size = V(workinmedia1)$betweenness/max(V(workinmedia1)$betweenness) * 5,
     main="Broker im Netzwerk der Medienunternehmen 
     und dort beschäftigten #30u30 der Jahrgänge 2017 - 2021",
     sub="Knotengröße entspr. Betweenness-Wert und mit Faktor 5 skaliert")
```
### Art der Beschäftigung

Bei einer Tätigkeit in einem Medienunternehmen kann es sich um Praktika oder Werkstudiumsjobs, also Tätigkeiten auf Einstiegs- / Young Professional-Ebene handeln, aber auch um Freelance-Jobs, Volontariate oder Festanstellungen, was im Edge-Attribut "relationship" entsprechend codiert wurde.

Im Folgenden wird die Verteilung der verschiedenen Beschäftigungsarten betrachtet.

1. Praktikums- und Werkstudiumsstellen
```{r Journ. Stationen: Praktika/Werkstudiums-Jobs}
workinmedia
workinmedia_intern <- delete.edges(workinmedia, E(workinmedia)[relationship != " 7"]) # selektiert nur die Praktikums-/Werkstudiumsstellen

workinmedia_intern1 <- delete.vertices(workinmedia_intern, V(workinmedia_intern)[degree(workinmedia_intern, V(workinmedia_intern), mode="all")==0]) # löscht Isolates

workinmedia_intern1

# Generiert einen Plot
plot(workinmedia_intern1,
     layout=layout_with_kk,
     edge.curved=curve_multiple(workinmedia_intern1),
     vertex.frame.color="white",
     #edge.color="grey",
     edge.curved=.2,
     edge.arrow.size=.00001,
     edge.arrow.color="white",
     vertex.label.cex = .2,
     vertex.label.color="black",
     vertex.label.family="sans",
     vertex.size=3,
     asp=0, 
     main="Praktikums- und Werkstudiumsstellen in Medienunternehmen
     der #30u30-Mitglieder der Jahrgänge 2017 bis 2021")

# Löscht mehrfache Kanten
workinmedia_intern2 <- simplify(workinmedia_intern1, remove.multiple = TRUE)

# Zählt die Anzahl der verschiedenen Knotentypen im Teilnetzwerk: Wie viele Personen sind im Praktikums-/Werkstudiumsnetzwerk enthalten?
mediainterns <- V(workinmedia_intern2)$type == 1
sum(mediainterns) # 48 Praktis / Werkstudis

# Wie viele Organisationen / Unternehmen?
mediainterns_org <- V(workinmedia_intern2)$type == 2
sum(mediainterns_org)  # 57 Org.
```
2. Freiberufliche Tätigkeiten 
```{r Journ. Stationen: Freelance-Jobs}
workinmedia
workinmedia_freelance <- delete.edges(workinmedia, E(workinmedia)[relationship != "10"]) # selektiert nur die Beziehungen, bei denen es sich um freiberufl. Tätigkeiten handelt

workinmedia_freelance <- delete.vertices(workinmedia_freelance, V(workinmedia_freelance)[degree(workinmedia_freelance, V(workinmedia_freelance), mode="all")==0]) # löscht isolierte Knoten

workinmedia_freelance

# Generiert den Plot des Teilnetzwerks
plot(workinmedia_freelance,
     layout=layout_with_kk,
     edge.curved=curve_multiple(workinmedia_freelance),
     vertex.frame.color="white",
     #edge.color="grey",
     edge.curved=.2,
     edge.arrow.size=.00001,
     edge.arrow.color="white",
     vertex.label.cex = .2,
     vertex.label.color="black",
     vertex.label.family="sans",
     vertex.size=3,
     asp=0,
     main="Freelance-Stellen in Medienunternehmen
     der #30u30-Mitglieder der Jahrgänge 2017 bis 2021")

# Löscht multiple Kanten
workinmedia_freelance1 <- simplify(workinmedia_freelance, remove.multiple = TRUE)

# Berechnung der Anzahl der einzelnen Knotentypen
# Wie viele Medien-Freelancer im Netzwerk?
mediafreelancer <- V(workinmedia_freelance1)$type == 1
sum(mediafreelancer)   # 28 Freelancer

# Wie viele Organisationen, bei denen diese beschäftigt waren?
mediafreelancer_org <- V(workinmedia_freelance1)$type == 2
sum(mediafreelancer_org)  # 33 Org.
```

3. Volontariate
```{r Journ. Stationen: Volontariate}
workinmedia
workinmedia_volo <- delete.edges(workinmedia, E(workinmedia)[relationship != " 8"]) # Löscht alle Kanten, bei denen es sich nicht um Volontariats-Kanten handelt

workinmedia_volo <- delete.vertices(workinmedia_volo, V(workinmedia_volo)[degree(workinmedia_volo, V(workinmedia_volo), mode="all")==0]) # Löscht isolierte Knoten

workinmedia_volo

# Generiert den Plot des Teilnetzwerks
plot(workinmedia_volo,
     layout=layout_with_kk,
     edge.curved=curve_multiple(workinmedia_volo),
     vertex.frame.color="white",
     #edge.color="grey",
     edge.curved=.2,
     edge.arrow.size=.00001,
     edge.arrow.color="white",
     vertex.label.cex = .2,
     vertex.label.color="black",
     vertex.label.family="sans",
     vertex.size=3,
     asp=0,
     main="Volontariatsstellen in Medienunternehmen
     der #30u30-Mitglieder der Jahrgänge 2017 bis 2021")

# Löscht multiple Kanten
workinmedia_volo1 <- simplify(workinmedia_volo, remove.multiple = TRUE)

# Zählt Anzahl der einzelnen Knotentypen
# Wie viele Personen haben ein Volontariat absolviert?
mediavolo <- V(workinmedia_volo1)$type == 1
sum(mediavolo)  # 4 Volos

# Bei wie vielen Organisationen wurden Volontariate absolviert?
mediavolo_org <- V(workinmedia_volo1)$type == 2
sum(mediavolo_org)  # 4 Org.
```
4. Festanstellungen
```{r Journ. Stationen: Festangestellte}
workinmedia
workinmedia_fulltime <- delete.edges(workinmedia, E(workinmedia)[relationship != " 9"]) # Löscht alle Knoten, bei denen es sich nicht um Vollzeit-Beschäftigungen bzw. Festanstellungen handelt

workinmedia_fulltime <- delete.vertices(workinmedia_fulltime, V(workinmedia_fulltime)[degree(workinmedia_fulltime, V(workinmedia_fulltime), mode="all")==0]) # löscht isolierte Knoten

workinmedia_fulltime

# Generiert den Plot des Teilnetzwerks
plot(workinmedia_fulltime,
     layout=layout_with_kk,
     edge.curved=curve_multiple(workinmedia_fulltime),
     vertex.frame.color="white",
     edge.curved=.2,
     edge.arrow.size=.00001,
     edge.arrow.color="white",
     vertex.label.cex = .2,
     vertex.label.color="black",
     vertex.label.family="sans",
     vertex.size=3,
     asp=0,
     main="Festanstellungen in Medienunternehmen
     der #30u30-Mitglieder der Jahrgänge 2017 bis 2021")

workinmedia_fulltime1 <- simplify(workinmedia_fulltime, remove.multiple = TRUE)

mediafulltimer <- V(workinmedia_fulltime1)$type == 1
sum(mediafulltimer)  # 8 Festangestellte

mediafulltimer_org <- V(workinmedia_fulltime1)$type == 2
sum(mediafulltimer_org) # 10 Org.
```

### Beschäftigung in Medienunternehmen - Über die Jahre

Welche Rolle spielen (journalistische) Tätigkeiten in Medienunternehmen im Verlauf der Karriere der Talente? Handelt es sich eher um Einstiegsjobs, bevor der Wechsel in die PR vollständig vollzogen wird oder gibt es auch mehrere Personen, die zu einem fortgeschrittenen Punkt in ihrer Karriere noch dort beschäftigt sind?

Hierzu werden die die einzelnen Jahre, von 2006 bis 2022, einzelnen betrachtet und berechnet, wie viele Personen jeweils eine Tätigkeit in Medienunternehmen ausgeübt haben und auf wie viele Medienunternehmen sich diese verteilen.

1. 2006
```{r Journ. Stationen: 2006}
workinmedia

workinmedia_2006 <- subgraph.edges(workinmedia, E(workinmedia)[year == "2006"]) # selektiert Beziehungen des Jahres 2006

workinmedia_06 <- simplify(workinmedia_2006, remove.multiple = TRUE)
# Kantenvereinfachung

# Zählt die Anzahl der verschiedenen Knotentypen
mediaworkers06 <- V(workinmedia_06)$type == 1
sum(mediaworkers06)  # 1 Person

mediaworkers06_org <- V(workinmedia_2006)$type == 2
sum(mediaworkers06_org)# 1 Org.
```

2. 2007
```{r Journ. Stationen: 2007}

workinmedia

workinmedia_2007 <- subgraph.edges(workinmedia, E(workinmedia)[year == "2007"]) # selektiert Beziehungen aus dem Jahr 2007

workinmedia_07 <- simplify(workinmedia_2007, remove.multiple = TRUE)

mediaworkers07 <- V(workinmedia_07)$type == 1
sum(mediaworkers07)  # 0 Personen

mediaworkers07_org <- V(workinmedia_2007)$type == 2
sum(mediaworkers07_org)  # 0 Organisationen
```

3. 2008
```{r Journ. Stationen: 2008}
workinmedia
workinmedia_2008 <- subgraph.edges(workinmedia, E(workinmedia)[year == "2008"]) # selektiert Beziehungen aus dem Jahr 2008

workinmedia_08 <- simplify(workinmedia_2008, remove.multiple = TRUE)
# Kantenvereinfachung

# Anzahl der Knotentypen:
mediaworkers08 <- V(workinmedia_08)$type == 1
sum(mediaworkers08)  # 4 Personen

mediaworkers08_org <- V(workinmedia_2008)$type == 2
sum(mediaworkers08_org)  # 4 Organisationen
```

4. 2009
```{r Journ. Stationen: 2009}
workinmedia
workinmedia_2009 <- subgraph.edges(workinmedia, E(workinmedia)[year == "2009"]) # Selektiert Beziehungen im Jahr 2009

workinmedia_09 <- simplify(workinmedia_2009, remove.multiple = TRUE)
# Kantenvereinfachung

# Anzahl der Knotentypen:
mediaworkers09 <- V(workinmedia_09)$type == 1
sum(mediaworkers09) # 11 Personen

mediaworkers09_org <- V(workinmedia_2009)$type == 2
sum(mediaworkers09_org) # 13 Org.
```

5. 2010
```{r Journ. Stationen: 2010}
workinmedia
workinmedia_2010 <- subgraph.edges(workinmedia, E(workinmedia)[year == "2010"]) # selektiert Beziehungen aus dem Jahr 2010

workinmedia_10 <- simplify(workinmedia_2010, remove.multiple = TRUE)
# Kantenvereinfachung

# Anzahl der Knotentypen
mediaworkers10 <- V(workinmedia_10)$type == 1
sum(mediaworkers10)  # 18 Personen

mediaworkers10_org <- V(workinmedia_2010)$type == 2
sum(mediaworkers10_org)  # 21 Org.
```

6. 2011
```{r Journ. Stationen: 2011}
workinmedia
workinmedia_2011 <- subgraph.edges(workinmedia, E(workinmedia)[year == "2011"]) # selektiert Beziehungen aus dem Jahr 2011

workinmedia_11 <- simplify(workinmedia_2011, remove.multiple = TRUE)
# Kantenvereinfachung

# Anzahl der Knotentypen
mediaworkers11 <- V(workinmedia_11)$type == 1
sum(mediaworkers11) # 18 Personen

mediaworkers11_org <- V(workinmedia_2011)$type == 2
sum(mediaworkers11_org) # 20 Org.
```

7. 2011
```{r Journ. Stationen: 2012}
workinmedia
workinmedia_2012 <- subgraph.edges(workinmedia, E(workinmedia)[year == "2012"]) # selektiert Beziehungen aus dem Jahr 2012

workinmedia_12 <- simplify(workinmedia_2012, remove.multiple = TRUE) # Kantenvereinfachung

# Anzahl der Knotentypen
mediaworkers12 <- V(workinmedia_12)$type == 1
sum(mediaworkers12) # 24 Personen

mediaworkers12_org <- V(workinmedia_2012)$type == 2
sum(mediaworkers12_org) # 25 Org.
```

8. 2013
```{r Journ. Stationen: 2013}
workinmedia
workinmedia_2013 <- subgraph.edges(workinmedia, E(workinmedia)[year == "2013"]) # selektiert Beziehungen aus dem Jahr 2013

workinmedia_13 <- simplify(workinmedia_2013, remove.multiple = TRUE)
# Kantenvereinfachung

# Anzahl der Knotentypen
mediaworkers13 <- V(workinmedia_13)$type == 1
sum(mediaworkers13) # 28 Personen

mediaworkers13_org <- V(workinmedia_2013)$type == 2
sum(mediaworkers13_org) # 34 Org.
```

10. 2014
```{r Journ. Stationen: 2014}
workinmedia
workinmedia_2014 <- subgraph.edges(workinmedia, E(workinmedia)[year == "2014"]) # Selektiert Beziehungen aus dem Jahr 2014

workinmedia_14 <- simplify(workinmedia_2014, remove.multiple = TRUE)
# Kantenvereinfachung

# Anzahl der Knotentypen
mediaworkers14 <- V(workinmedia_14)$type == 1
sum(mediaworkers14) # 29 Personen

mediaworkers14_org <- V(workinmedia_2014)$type == 2
sum(mediaworkers14_org) # 35 Org.
```

11. 2015
```{r Journ. Stationen: 2015}
workinmedia
workinmedia_2015 <- subgraph.edges(workinmedia, E(workinmedia)[year == "2015"]) # selektiert Beziehungen aus dem Jahr 2015

workinmedia_15 <- simplify(workinmedia_2015, remove.multiple = TRUE)
# Kantenvereinfachung

# Anzahl der Knotentypen
mediaworkers15 <- V(workinmedia_15)$type == 1
sum(mediaworkers15)  # 27 Personen

mediaworkers15_org <- V(workinmedia_2015)$type == 2
sum(mediaworkers15_org)  # 31 Org.
```

12. 2016
```{r Journ. Stationen: 2016}
workinmedia
workinmedia_2016 <- subgraph.edges(workinmedia, E(workinmedia)[year == "2016"]) # selektiert Beziehungen aus dem Jahr 2016

workinmedia_16 <- simplify(workinmedia_2016, remove.multiple = TRUE)
# Kantenvereinfachung

# Anzahl der Knotentypen
mediaworkers16 <- V(workinmedia_16)$type == 1
sum(mediaworkers16)  # 17 Personen

mediaworkers16_org <- V(workinmedia_2016)$type == 2
sum(mediaworkers16_org) # 21 Org.
```

13. 2017
```{r Journ. Stationen: 2017}
workinmedia
workinmedia_2017 <- subgraph.edges(workinmedia, E(workinmedia)[year == "2017"]) # Selektiert Beziehungen aus dem Jahr 2017

workinmedia_17 <- simplify(workinmedia_2017, remove.multiple = TRUE)
# Kantenvereinfachung

# Anzahl der Knotentypen
mediaworkers17 <- V(workinmedia_17)$type == 1
sum(mediaworkers17) # 15 Personen

mediaworkers17_org <- V(workinmedia_2017)$type == 2
sum(mediaworkers17_org) # 17 Org.
```

14. 2018
```{r Journ. Stationen: 2018}
workinmedia
workinmedia_2018 <- subgraph.edges(workinmedia, E(workinmedia)[year == "2018"]) # selektiert Beziehungen aus dem Jahr 2018

workinmedia_18 <- simplify(workinmedia_2018, remove.multiple = TRUE)
# Kantenvereinfachung

# Anzahl der Knotentypen
mediaworkers18 <- V(workinmedia_18)$type == 1
sum(mediaworkers18) # 15 Personen

mediaworkers18_org <- V(workinmedia_2018)$type == 2
sum(mediaworkers18_org) # 14 Org.
```

15. 2019
```{r Journ. Stationen: 2019}
workinmedia
workinmedia_2019 <- subgraph.edges(workinmedia, E(workinmedia)[year == "2019"]) # Selektiert Beziehungen aus dem Jahr 2019

workinmedia_19 <- simplify(workinmedia_2019, remove.multiple = TRUE)
# Kantenvereinfachung

# Anzahl der Knotentypen
mediaworkers19 <- V(workinmedia_19)$type == 1
sum(mediaworkers19) # 11 Personen

mediaworkers19_org <- V(workinmedia_2019)$type == 2
sum(mediaworkers19_org) # 10 Org.
```

16. 2020
```{r Journ. Stationen: 2020}
workinmedia
workinmedia_2020 <- subgraph.edges(workinmedia, E(workinmedia)[year == "2020"]) # selektiert Beziehungen aus dem Jahr 2020

workinmedia_20 <- simplify(workinmedia_2020, remove.multiple = TRUE)
# Kantenvereinfachung

# Anzahl der Knotentypen:
mediaworkers20 <- V(workinmedia_20)$type == 1
sum(mediaworkers20) # 7 Personen

mediaworkers20_org <- V(workinmedia_2020)$type == 2
sum(mediaworkers20_org) # 6 Org.
```

17. 2021
```{r Journ. Stationen: 2021}
workinmedia
workinmedia_2021 <- subgraph.edges(workinmedia, E(workinmedia)[year == "2021"]) # selektiert Beziehungen aus dem Jahr 2021

workinmedia_21 <- simplify(workinmedia_2021, remove.multiple = TRUE)
# Kantenvereinfachung

# Anzahl der Knotentypen
mediaworkers21 <- V(workinmedia_21)$type == 1
sum(mediaworkers21) # 6 Personen

mediaworkers21_org <- V(workinmedia_21)$type == 2
sum(mediaworkers21_org) # 7 Org.
```

18. 2022
```{r Journ. Stationen: 2022}
workinmedia
workinmedia_2022 <- subgraph.edges(workinmedia, E(workinmedia)[year == "2022"]) # selektiert Beziehungen aus dem Jahr 2018

workinmedia_22 <- simplify(workinmedia_2022, remove.multiple = TRUE)
# Kantenvereinfachung

# Anzahl der Knotentypen
mediaworkers22 <- V(workinmedia_22)$type == 1
sum(mediaworkers22) # 5 Personen

mediaworkers22_org <- V(workinmedia_22)$type == 2
sum(mediaworkers22_org) # 6 Org.
```

### Sponsoren als Arbeitgeber
Im Kontext der beruflichen Verbindungen der #30u30 kann auch die Rolle der Sponsoren von #30u30 betrachtet werden: Werden besonders viele Talente aufgenommen, die zum Aufnahmezeitpunkt bei einem Partnerunternehmen arbeiten? Steigt der Anteil derer, die bei Partnerunternehmen arbeiten, nach deren Aufnahme?

Dies wird im Folgenden betrachtet.
Zunächst: Wie viele Sponsoren/Partnerunternehmen enthält das Arbeitsnetzwerk heute und wie viele in den jew. Aufnahmejahren zusammen?

```{r Sponsoren bei Aufnahme und heute}
spon_work22 <- V(work_simple)$sponsor == 1
sum(spon_work22, na.rm=T) # heute: 15 Partnerorg. / Sponsoren

spon_pe <- V(members_preentry2)$sponsor == 1
sum(spon_pe, na.rm = T) # in Aufnahmejahren insg.: 13 Sponsoren
```

#### Rolle der Sponsoren als Arbeitgeber in einzelnen Jahrgängen

1. Jahrgang 2017

a) Wie viele Sponsoren waren im Arbeitsnetzwerk der Mitglieder des Jahrgangs 2017 bis zum Aufnahmejahr enthalten?
```{r Sponsoren - Jahrgang 2017 - Bis Aufnahme}
jg17_preentry

jg17_pe_spon <- V(jg17_preentry)$sponsor == 1
sum(jg17_pe_spon) # 4 Sponsoren
```

b) Wie viele Sponsoren sind im Arbeitsnetzwerk der Mitglieder des Jahrgangs 2017 heute insgesamt enthalten?

```{r Sponsoren - Jahrgang 2017 - Gesamt}
jg17_2

jg17_spon <- V(jg17_2)$sponsor == 1
sum(jg17_spon) # 7 Sponsoren
```

2. Jahrgang 2018
a) Wie viele Sponsoren waren im Arbeitsnetzwerk der Mitglieder des Jahrgangs 2018 bis zum Aufnahmejahr enthalten?
```{r Sponsoren - Jahrgang 2018 - Bis Aufnahme}
jg18_preentry

jg18_pe_spon <- V(jg18_preentry1)$sponsor == 1
sum(jg18_pe_spon, na.rm=T) # 4 Sponsoren
```

b) Wie viele Sponsoren sind im Arbeitsnetzwerk der Mitglieder des Jahrgangs 2018 bis heute enthalten?

```{r Sponsoren - Jahrgang 2018 - Gesamt}

jg18_3

jg18_spon <- V(jg18_3)$sponsor == 1
sum(jg18_spon, na.rm=T)
# 5 Sponsoren
```

3. Jahrgang 2019
a) Wie viele Sponsoren waren im Arbeitsnetzwerk der Mitglieder des Jahrgangs 2019 bis zum Aufnahmejahr enthalten?
```{r Sponsoren - Jahrgang 2019 - Bis Aufnahme}
jg19_preentry

jg19_pe_spon <- V(jg19_preentry)$sponsor == 1
sum(jg19_pe_spon) # 5 Sponsoren
```

b) Wie viele Sponsoren sind insgesamt im Arbeitsnetzwerk der Mitglieder des Jahrgangs 2019 bis heute enthalten?

```{r Sponsoren - Jahrgang 2019 - Gesamt}
jg19_3

jg19_spon <- V(jg19_3)$sponsor == 1
sum(jg19_spon)# 7 Sponsoren
```
4. Jahrgang 2020
a) Wie viele Sponsoren waren im Arbeitsnetzwerk der Mitglieder des Jahrgangs 2020 bis zum Aufnahmejahr enthalten?

```{r Sponsoren - Jahrgang 2020 - Bis Aufnahme}
jg20_preentry

jg20_pe_spon <- V(jg20_preentry)$sponsor == 1
sum(jg20_pe_spon) # 4 Sponsoren
```

a) Wie viele Sponsoren sind insgesamt im Arbeitsnetzwerk der Mitglieder des Jahrgangs 2020 bis heute enthalten?

```{r Sponsoren - Jahrgang 2020 - Gesamt}
jg20_spon <- V(jg20_2)$sponsor == 1
sum(jg20_spon) # 4 Sponsoren
```
5. Jahrgang 2021
a) Wie viele Sponsoren waren im Arbeitsnetzwerk der Mitglieder des Jahrgangs 2021 bis zum Aufnahmejahr enthalten?

```{r Sponsoren - Jahrgang 2021 - Bis Aufnahme}
jg21_preentry
jg21_pe_spon <- V(jg21_preentry)$sponsor == 1
sum(jg21_pe_spon, na.rm=T) # 4 Sponsoren
```

b) Wie viele Sponsoren sind insgesamt im Arbeitsnetzwerk der Mitglieder des Jahrgangs 2021 bis heute enthalten?

```{r Sponsoren - Jahrgang 2021 - Gesamt}
jg21_2
jg21_spon <- V(jg21_2)$sponsor == 1
sum(jg21_spon, na.rm=T) # 4 Sponsoren
```

## Akademische Stationen: Studiumsnetzwerk der #30u30-Jahrgänge 2017 - 2021

Im Folgenden werden die Ausbildungsstätten der #30u30-Mitglieder analysiert, sowohl vergleichend zwischen den einzelnen Jahrgängen als auch mit Blick auf Bachelor- und Masterstudien.

Zunächst wird das Teilnetzwerk der Studiumsverbindungen erstellt.
```{r Akademische Stationen: Teilnetzwerk Studium erstellen}
members # ruft Mitgliedernetzwerk ab

unis <- delete_vertices(members, V(members)[(type == 2) & (category != 5)]) # löscht alle Organisationen außer Hochschulen

unis <- delete.vertices(unis, V(unis)[degree(unis, V(unis), mode="all")==0]) # löscht isolierte Knoten

study <- delete.edges(unis, E(unis)[relationship > " 6"]) # Löscht Arbeitsbeziehungen (da auch einige Personen bei Unis gearbeitet haben)
                      
study <- delete.vertices(study, V(study)[degree(study, V(study), mode="all")==0]) # löscht isolierte Knoten

study_simple <- simplify(study, remove.multiple = T) # Kantenvereinfachung
```

Wie vielen Verbindungen gehen auf die einzelnen Hochschulen ein? Wo haben mehrere Personen studiert? Welche populären Hochschulen stechen hervor?
```{r Akademische Stationen: Indegree-Verteilung}
ind_hs <- degree(study_simple, mode="in")
sort(ind_hs, decreasing = T)
```

Wie viele der 150 Talente haben studiert, wie viele Hochschulen enthält das Teilnetzwerk und welche Hochschultypen?
```{r Knoten- / Hochschultypen}
# Wie viele Personen haben studiert?
study_pers <- V(study)$type == 1
sum(study_pers)  # 144 Personen

# Wie viele Hochschulen im Netzwerk?
study_hs <- V(study)$category == 5
sum(study_hs)  # 189 Hochschulen insgesamt

# Verteilung der einzelnen Hochschultypen:

# Wie viele staatl. Unis?
study_uni_staat <- V(study)$ownership == 1
sum(study_uni_staat)  # 121 staatliche Unis

# wie viele Privatunis?
study_uni_priv <- V(study)$ownership == 2
sum(study_uni_priv)  # 15 private Unis

# Wie viele Staatl. FHs / HAWs?
study_fh_staat <- V(study)$ownership == 3
sum(study_fh_staat)  # 23 staatliche FHs

# wie viele private FHs?
study_fh_priv <- V(study)$ownership == 4
sum(study_fh_priv)  # 14 private FHs

# wie viele Business Schools?
study_bs <- V(study)$ownership == 5
sum(study_bs)  # 5 Business Schools

# wie viele Weiterbildungsakademien?
study_wb <- V(study)$ownership == 6
sum(study_wb)  # 6 Weiterbildungsakademien

# wie viele Berufsschulen?
study_berufsschule <- V(study)$ownership == 7
sum(study_berufsschule)  # 1 Berufsschule

# wie viele duale HS?
study_dual <- V(study)$ownership == 8
sum(study_dual)  # 1 duale HS

# Sonstige:
study_sonst <- V(study)$ownership == 9
sum(study_sonst)  # 3 Sonstige HS
```

Zur Unterscheidung werden die Knoten der Hochschultypen unterschiedlich eingefärbt:

```{r Akademische Stationen: Visualisierungsanpassung - Knoten einfärben}
V(study_simple)[mentor==1]$color <- "firebrick1" # färbt Knoten der #30u30 rot

# Einfärben der versch. Hochschultypen
V(study_simple)[ownership==1]$color <- "palegreen3" # Staatliche Unis hellgrün
V(study_simple)[ownership==2]$color <- "olivedrab3" # Staatliche FHs hell-oliv
V(study_simple)[ownership==3]$color <- "palegreen4" # Private Unis dunkelgrün
V(study_simple)[ownership==4]$color <- "olivedrab4" # Private FHs dunkeloliv
V(study_simple)[ownership==5]$color <- "darkseagreen" # Business Schools dunkel-pastellgrün
V(study_simple)[ownership==6]$color <- "darkseagreen1" # Weiterbildungsakademie pastellgrün
V(study_simple)[ownership==7]$color <- "green" # Berufsschule knallgrün
V(study_simple)[ownership==8]$color <- "grey" # Sonstige HS grau
```

Generiert den Plot der Studiumsbeziehungen in den fünf Jahrgängen.
```{r Akademische Stationen: Hochschulnetzwerk - Plot}
plot(study_simple,
     layout=layout_with_kk,
     edge.curved=curve_multiple(study_simple), # verhindert, dass sich Kanten überlagern
     vertex.frame.color="white",
     edge.arrow.size=.0001,
     edge.arrow.color="white",
     vertex.size=ind_hs*1.2,
     edge.color="grey",
     edge.curved=.2,
     vertex.label=ifelse(ind_hs>4, V(study_simple)$name, NA), # Label wird nur bei Knoten mit einem indegree-Wert von über 4 angezeigt
     vertex.label.degree=0, 
     vertex.label.cex=.25,
     vertex.label.family="sans",
     vertex.label.font=2,
     vertex.label.degree=.8,
     vertex.label.color="black",
     asp=0,
     main="Studiumsnetzwerk der #30u30-Jahrgänge 2017 bis 2021")
```

### Bachelorstudium
Nun werden ausschließlich die Verbindungen zwischen den Personen und den Hochschulen betrachtet, wo diese ein Bachelorstudium absolviert haben.

Erstellt das Teilnetzwerk der Bachelorstudien.
```{r Teilnetzwerk Bachelorstudium}
study
bachelor <- subgraph.edges(study, E(study)[relationship == " 2"]) # selektiert Kanten, bei denen es sich um ein Bachelorstudium handelt

bachelor <- delete.vertices(bachelor, V(bachelor)[degree(bachelor, V(bachelor), mode ="all")==0]) # löscht isolierte Knoten

bachelor_simple <- simplify(bachelor, remove.multiple = T) # Kantenvereinfachung
```

Welche Hochschulen waren im Bachelorstudium besonders populär? Wo haben jahrgangsübergreifend mehrere Personen studiert?

```{r Bachelorstudium: Indegree-Verteilung}
ind_bachelor <- degree(bachelor_simple, mode = "in")
sort(ind_bachelor, decreasing = T)
```

Wie viele Personen haben ein Bachelorstudium absolviert, auf wie viele Hochschulen verteilen sich diese und auf welche Hochschultypen?
```{r Bachelorstudium: Knoten- / Hochschultypen}
# Wie viele Personen haben studiert?
study_ba <- V(bachelor)$type == 1
sum(study_ba) # 144 Personen

# Wie viele Hochschulen im Bachelor-Netzwerk?
study_ba_hs <- V(bachelor)$category == 5
sum(study_ba_hs) # 84 Hochschulen

# Verteilung auf Hochschultypen
  # Wie viele staatl. Unis?
ba_uni_staat <- V(bachelor)$ownership == 1
sum(ba_uni_staat) # 47 staatliche Unis

# wie viele Privatunis?
ba_uni_priv <- V(bachelor)$ownership == 2
sum(ba_uni_priv)  # 4 private Unis

# Wie viele Staatl. FHs / HAWs?
ba_fh_staat <- V(bachelor)$ownership == 3
sum(ba_fh_staat)  # 18 staatliche FHs

# wie viele private FHs?
ba_fh_priv <- V(bachelor)$ownership == 4
sum(ba_fh_priv)  # 10 private FHs

# wie viele Business Schools?
ba_bs <- V(bachelor)$ownership == 5
sum(ba_bs)  # 0 Business Schools

# wie viele Weiterbildungsakademien?
ba_wb <- V(bachelor)$ownership == 6
sum(ba_wb)  # 2 Weiterbildungsakademien

# wie viele Berufsschulen?
ba_berufsschule <- V(bachelor)$ownership == 7
sum(ba_berufsschule)  # 0 Berufsschulen

# wie viele duale HS?
ba_dual <- V(bachelor)$ownership == 8
sum(ba_dual)  # 1 duale HS

# Sonstige:
ba_sonst <- V(bachelor)$ownership == 9
sum(ba_sonst)  # 2 Sonstige HS
```
Zur Unterscheidung werden die Knoten je nach Hochschultyp unterschiedlich eingefärbt.

```{r Bachelorstudium: Visualisierungsanpassung}
V(bachelor_simple)[mentor==1]$color <- "firebrick1" # färbt Knoten der #30u30 rot

# Einfärben der versch. Hochschultypen
V(bachelor_simple)[ownership==1]$color <- "palegreen3" # Staatliche Unis hellgrün
V(bachelor_simple)[ownership==2]$color <- "olivedrab3" # Staatliche FHs hell-oliv
V(bachelor_simple)[ownership==3]$color <- "palegreen4" # Private Unis dunkelgrün
V(bachelor_simple)[ownership==4]$color <- "olivedrab4" # Private FHs dunkeloliv
V(bachelor_simple)[ownership==5]$color <- "darkseagreen" # Business Schools dunkel-pastellgrün
V(bachelor_simple)[ownership==6]$color <- "darkseagreen1" # Weiterbildungsakademie pastellgrün
V(bachelor_simple)[ownership==7]$color <- "green" # Berufsschule knallgrün
V(bachelor_simple)[ownership==8]$color <- "grey" # Sonstige HS grau
```

Erzeugt den Plot der Personen und der Hochschulen, an denen diese ein Bachelorstudium absolviert haben.
```{r Bachelorstudium: Plot}
plot(bachelor_simple,
     layout=layout_with_kk,
     edge.curved=curve_multiple(bachelor_simple), # verhindert, dass sich Kanten überlagern
     vertex.frame.color="white",
     edge.arrow.size=.0001,
     edge.arrow.color="white",
     vertex.size=ind_bachelor*1.2,
     edge.color="grey",
     edge.curved=.2,
     vertex.label=ifelse(ind_bachelor>4, V(bachelor_simple)$name, NA), # Label wird nur bei Knoten mit einem indegree-Wert von über 4 angezeigt
     vertex.label.degree=0, 
     vertex.label.cex=.4,
     vertex.label.family="sans",
     vertex.label.font=2,
     vertex.label.color="black",
     main="Bachelorstudiums-Netzwerk 
     der #30u30-Jahrgänge 2017 bis 2021",
     sub="Knotengröße entspr. Indegree-Wert und mit Faktor 1,2 skaliert",
     asp=0)

# Mit Labels bei allen Hochschulen (mit indegree > 1)
plot(bachelor_simple,
     layout=layout_with_fr,
     edge.curved=curve_multiple(bachelor_simple), # verhindert, dass sich Kanten überlagern
     vertex.frame.color="white",
     edge.arrow.size=.0001,
     edge.arrow.color="white",
     vertex.size=ind_bachelor*1.2,
     edge.color="grey",
     edge.curved=.2,
     vertex.label=ifelse(ind_bachelor>1, V(bachelor_simple)$name, NA), # Label wird nur bei Knoten mit einem indegree-Wert von über 1 angezeigt
     vertex.label.degree=0, 
     vertex.label.cex=.4,
     vertex.label.family="sans",
     vertex.label.font=2,
     vertex.label.color="black",
     main="Bachelor-Hochschulen der Jahrgänge 2017 bis 2021",
     asp=0)
```

### Masterstudium
Nun werden nur die Verbindungen zwischen den Personen und den Hochschulen betrachtet, wo diese ein Masterstudium absolviert haben.

```{r Teilnetzwerk Masterstudium erstellen}
study
master <- subgraph.edges(study, E(study)[relationship == " 3"]) # selektiert die Kanten, bei denen es sich um ein Masterstudium handelt

master <- delete.vertices(master, V(master)[degree(master, V(master), mode ="all")==0]) # löscht isolierte Knoten

master_simple <- simplify(master, remove.multiple = T) # Kantenvereinfachung
```

Welche Hochschulen waren als Masterstudiumsstätte besonders populär und verbinden mehrere Talente?
```{r Masterstudium: Indegree-Verteilung}
ind_master <- degree(master_simple, mode = "in")
sort(ind_master, decreasing = T)
```

Wie viele Personen haben ein Masterstudium absolviert und auf wie viele und welche Art von Hochschule verteilen sich diese?
```{r Masterstudium: Knoten- / Hochschultypen}
# Wie viele Personen haben ein Masterstudium?
study_ma <- V(master)$type == 1
sum(study_ma)  # 107 Personen

# Wie viele Hochschulen im Netzwerk?
study_ma_hs <- V(master)$category == 5
sum(study_ma_hs)  # 69 Hochschulen

# Anzahl der Hochschultypen
  # Wie viele staatl. Unis?
ma_uni_staat <- V(master)$ownership == 1
sum(ma_uni_staat) # 46 staatliche Unis

# wie viele Privatunis?
ma_uni_priv <- V(master)$ownership == 2
sum(ma_uni_priv)  # 2 private Unis

# Wie viele Staatl. FHs / HAWs?
ma_fh_staat <- V(master)$ownership == 3
sum(ma_fh_staat)  # 10 staatliche FHs

# wie viele private FHs?
ma_fh_priv <- V(master)$ownership == 4
sum(ma_fh_priv)  # 5 private FHs

# wie viele Business Schools?
ma_bs <- V(master)$ownership == 5
sum(ma_bs)  # 2 Business Schools

# wie viele Weiterbildungsakademien?
ma_wb <- V(master)$ownership == 6
sum(ma_wb)  # 2 Weiterbildungsakademien

# wie viele Berufsschulen?
ma_berufsschule <- V(master)$ownership == 7
sum(ma_berufsschule)  # 0 Berufsschulen

# wie viele duale HS?
ma_dual <- V(master)$ownership == 8
sum(ma_dual)  # 0 duale HS

# Sonstige:
ma_sonst <- V(master)$ownership == 9
sum(ma_sonst)  # 1 Sonstige HS
```

Zur Unterscheidung werden die Knoten der verschiedenen Hochschultypen unterschiedlich eingefärbt.

```{r Masterstudium: Visualisierungsanpassung}
V(master_simple)[mentor==1]$color <- "firebrick1" # färbt Knoten der #30u30 rot

# Einfärben der versch. Hochschultypen
V(master_simple)[ownership==1]$color <- "palegreen3" # Staatliche Unis hellgrün
V(master_simple)[ownership==2]$color <- "olivedrab3" # Staatliche FHs hell-oliv
V(master_simple)[ownership==3]$color <- "palegreen4" # Private Unis dunkelgrün
V(master_simple)[ownership==4]$color <- "olivedrab4" # Private FHs dunkeloliv
V(master_simple)[ownership==5]$color <- "darkseagreen" # Business Schools dunkel-pastellgrün
V(master_simple)[ownership==6]$color <- "darkseagreen1" # Weiterbildungsakademie pastellgrün
V(master_simple)[ownership==7]$color <- "green" # Berufsschule knallgrün
V(master_simple)[ownership==8]$color <- "grey" # Sonstige HS grau
```

Erstellt den Plot der Personen und der Hochschulen, wo diese im Master studiert haben.
```{r Masterstudium: Plot}
plot(master_simple,
     layout=layout_with_kk,
     edge.curved=curve_multiple(master_simple), # verhindert, dass sich Kanten überlagern
     vertex.frame.color="white",
     edge.arrow.size=.0001,
     edge.arrow.color="white",
     vertex.size=ind_master*1.2,
     edge.color="grey",
     edge.curved=.2,
     vertex.label=ifelse(ind_master>4, V(master_simple)$name, NA), # Label wird nur bei Knoten mit einem indegree-Wert von über 4 angezeigt
     vertex.label.degree=0, 
     vertex.label.cex=.4,
     vertex.label.family="sans",
     vertex.label.font=2,
     vertex.label.color="black",
     main="Master-Hochschulen der #30u30-Jahrgänge 2017 bis 2021",
     sub="Knotengröße entspr. Indegree-Wert und mit Faktor 1,2 skaliert"
     asp=0)

# Mit Labels bei allen HS mit indegree > 1
plot(master_simple,
     layout=layout_with_fr,
     edge.curved=curve_multiple(master_simple), # verhindert, dass sich Kanten überlagern
     vertex.frame.color="white",
     edge.arrow.size=.0001,
     edge.arrow.color="white",
     vertex.size=ind_master*1.2,
     edge.color="grey",
     edge.curved=.2,
     vertex.label=ifelse(ind_master>1, V(master_simple)$name, NA), # Label wird nur bei Knoten mit einem indegree-Wert von über 1 angezeigt
     vertex.label.degree=0, 
     vertex.label.cex=.4,
     vertex.label.family="sans",
     vertex.label.font=2,
     vertex.label.color="black",
     main="Master-Hochschulen der #30u30-Jahrgänge 2017 bis 2021",
     sub="Knotengröße entspr. Indegree-Wert und mit Faktor 1,2 skaliert"
     asp=0)
```

### Einzelne Jahrgänge: Ausbildungsstätten

Nun werden die Ausbildungsstätten der einzelnen Jahrgänge und deren Mitglieder betrachtet.

#### Jahrgang 2017
Erstellt das Studiumsnetzwerk des Jahrgangs 2017:
```{r Studiumsnetzwerk - Jahrgang 2017}
study17 <- delete.vertices(study, V(study)[(type == 1) & (entry != 2017)]) # selektiert aus dem Studiumsnetzwerk von den #30u30 nur den Jahrgang 2017

study17 <- delete.vertices(study17, V(study17)[degree(study17, V(study17), mode="all")==0]) # löscht isolierte Kanten
```

Löscht mehrfache Kanten, damit bei der Indegree-Abfrage nur die einfache Anzahl der Verbindungen zwischen Person und Hochschule geliefert wird (nicht für jedes Jahr).

```{r Studiumsnetzwerk - Jahrgang 2017 - Kanten-Vereinfachung}
study17_simple <- simplify(study17, remove.multiple = TRUE)
```

Welche Hochschulen verbinden mehrere Talente des Jahrgangs 2017?
```{r Studiumsnetzwerk - Jahrgangs 2017 - Indegree-Verteilung}
ind_study17 <- degree(study17_simple, V(study17_simple), mode="in")
sort(ind_study17,decreasing=TRUE)
```

##### Jahrgang 2017 - Bachelorstudium
Erstellt Netzwerk des Jahrgangs 2017 nur mit Verbindungen zwischen den Mitgliedern und Hochschulen, wo diese im Bachelor studiert haben.
```{r Studiumsnetzwerk - Jahrgang 2017 - Bachelor}
study17_ba <- subgraph.edges(study17, E(study17)[relationship == " 2"]) # selektiert Kanten, bei denen es sich um BA-Studium handelt

study17_ba <- delete.vertices(study17_ba, V(study17_ba)[degree(study17_ba, V(study17_ba), mode="all")]==0) # löscht isolierte Knoten

study17_ba1 <- simplify(study17_ba, remove.multiple = T) # Kantenvereinfachung
```

Bei welchen Hochschulen haben mehrere Jahrgangsmitglieder ein Bachelorstudium absolviert?
```{r Studiumsnetzwerk - Jahrgang 2017 - Bachelor - Indegree-Verteilung}
ind_ba17 <- degree(study17_ba1, mode="in")
sort(ind_ba17, decreasing = T)
```

Plottet das Netzwerk der Mitglieder des Jahrgangs 2017 und deren Bachelorhochschulen:
```{r Studiumsnetzwerk - Jahrgang 2017 - Bachelor - Plot}
plot(study17_ba1,
     layout=layout_with_fr,
     edge.curved=curve_multiple(study17_ba1),
     vertex.size=3,
     vertex.frame.color="white",
     edge.color="grey",
     edge.curved=.2,
     edge.arrow.size=.00001,
     edge.arrow.color="white",
     vertex.label.degree=0, 
     vertex.label.cex=.25,
     vertex.label.family="sans",
     vertex.label.font=2,
     vertex.label.color="black", 
     asp=0,
     main="Bachelorstudiums-Hochschulen 
     der #30u30-Mitglieder des Jahrgangs 2017")
```

##### Jahrgang 2017 - Masterstudium
Erstellt Netzwerk des Jahrgangs 2017 nur mit Verbindungen zwischen den Mitgliedern und Hochschulen, wo diese im Master studiert haben.
```{r Studiumsnetzwerk - Jahrgang 2017 - Master}
study17_ma <- subgraph.edges(study17, E(study17)[relationship == " 3"]) # selektiert nur Kanten, bei denen es sich um Master-Studium handelt

study17_ma <- delete.vertices(study17_ma, V(study17_ma)[degree(study17_ma, V(study17_ma), mode="all")]==0) # löscht isolierte Knoten

study17_ma1 <- simplify(study17_ma, remove.multiple = T) # Kantenvereinfachung
```

Welche Hochschulen waren bei Masterstudierenden des Jahrgangs populär? Wie viele Verbindungen gehen auf die einzelnen Hochschulen ein?

```{r Studiumsnetzwerk - Jahrgang 2017 - Master - Indegree-Verteilung}
ind_ma17 <- degree(study17_ma1, mode="in")
sort(ind_ma17, decreasing = T)
```

Erstellt den Plot der Beziehungen zwischen den Jahrgangsmitgliedern und Master-Hochschulen:

```{r Studiumsnetzwerk - Jahrgang 2017 - Master - Plot}
plot(study17_ma1,
     layout=layout_with_fr,
     edge.curved=curve_multiple(study17_ma1),
     vertex.size=3,
     vertex.frame.color="white",
     edge.color="grey",
     edge.curved=.2,
     edge.arrow.size=.00001,
     edge.arrow.color="white",
     vertex.label.degree=0, 
     vertex.label.cex=.25,
     vertex.label.family="sans",
     vertex.label.font=2,
     vertex.label.color="black", 
     asp=0,
     main="Masterstudiums-Hochschulen 
     der #30u30-Mitglieder des JG 2017")
```

#### Jahrgang 2018

Erstellt das Studiumsnetzwerk des Jahrgangs 2018:
```{r Studiumsnetzwerk - Jahrgang 2018}
study18 <- delete.vertices(study, V(study)[(type == 1) & (entry != 2018)]) # selektiert aus dem Studiumsnetzwerk unter den #30u30 nur den Jahrgang 2018

study18 <- delete.vertices(study18, V(study18)[degree(study18, V(study18), mode="all")==0]) # löscht isolierte Knoten

study18
```

Löscht mehrfache Kanten, damit bei der Indegree-Abfrage nur die einfache Anzahl der Verbindungen geliefert wird (nicht für jedes Jahr).

```{r Studiumsnetzwerk - Jahrgang 2018 - Kanten-Vereinfachung}
study18_simple <- simplify(study18, remove.multiple = TRUE)
```

Welche Hochschulen sind im Jahrgang 2018 besonders populär und verbinden mehrere Personen?
```{r Studiumsnetzwerk - Jahrgang 2018 - Indegree-Verteilung}
ind_study18 <- degree(study18_simple, V(study18_simple), mode="in")
sort(ind_study18,decreasing=TRUE)
```

##### Jahrgang 2018 - Bachelorstudium
Erstellt Netzwerk des Jahrgangs 2018 nur mit Verbindungen zwischen den Mitgliedern und Hochschulen, wo diese im Bachelor studiert haben.

```{r Studiumsnetzwerk - Jahrgang 2018 - Bachelor}
study18_ba <- subgraph.edges(study18, E(study18)[relationship == " 2"]) # selektiert Kanten, bei denen es sich um ein Bachelor-Studium handelt

study18_ba <- delete.vertices(study18_ba, V(study18_ba)[degree(study18_ba, V(study18_ba), mode="all")]==0) # löscht isolierte Knoten

study18_ba1 <- simplify(study18_ba, remove.multiple = T) # Kantenvereinfachung
```

Welche Hochschulen waren im Bachelorstudium besonders populär, wo haben mehrere Personen im Bachelor studiert?

```{r Studiumsnetzwerk Jahrgang 2018 - Indgree-Verteilung}
ind_ba18 <- degree(study18_ba1, mode="in")
sort(ind_ba18, decreasing = T)
```

Erzeugt den Plot der Beziehungen zwischen den Jahrgangsmitgliedern und Bachelor-Hochschulen:

```{r Studiumsnetzwerk - Jahrgang 2018 - Bachelor - Plot}
plot(study18_ba1,
     layout=layout_with_fr,
     edge.curved=curve_multiple(study18_ba1),
     vertex.size=3,
     vertex.frame.color="white",
     edge.color="grey",
     edge.curved=.2,
     edge.arrow.size=.00001,
     edge.arrow.color="white",
     vertex.label.degree=0, 
     vertex.label.cex=.25,
     vertex.label.family="sans",
     vertex.label.font=2,
     vertex.label.color="black", 
     asp=0,
     main="Bachelorstudiums-Hochschulen 
     der #30u30-Mitglieder des Jahrgangs 2018")
```

##### Jahrgang 2018: Masterstudium
Erstellt Netzwerk des Jahrgangs 2018 nur mit Verbindungen zwischen den Mitgliedern und Hochschulen, wo diese im Master studiert haben.

```{r Studiumsnetzwerk - Jahrgang 2018 - Master}
study18_ma <- subgraph.edges(study18, E(study18)[relationship == " 3"]) # selektiert Kanten, bei denen es sich um Master-Studium handelt

study18_ma <- delete.vertices(study18_ma, V(study18_ma)[degree(study18_ma, V(study18_ma), mode="all")]==0) # löscht isolierte Knoten

study18_ma1 <- simplify(study18_ma, remove.multiple = T) # Kantenvereinfachung
```

Bei welchen Hochschulen haben mehrere Personen ein Masterstudium absolviert?
```{r Studiumsnetzwerk - Jahrgang 2018 - Indegree-Verteilung}
ind_ma18 <- degree(study18_ma1, mode="in")
sort(ind_ma18, decreasing = T)
```

Erstellt den Plot der Beziehungen zwischen den Jahrgangsmitgliedern und Master-Hochschulen:

```{r Studiumsnetzwerk - Jahrgang 2018 - Master - Plot}
plot(study18_ma1,
     layout=layout_with_fr,
     edge.curved=curve_multiple(study18_ma1),
     vertex.size=3,
     vertex.frame.color="white",
     edge.color="grey",
     edge.curved=.2,
     edge.arrow.size=.00001,
     edge.arrow.color="white",
     vertex.label.degree=0, 
     vertex.label.cex=.25,
     vertex.label.family="sans",
     vertex.label.font=2,
     vertex.label.color="black", 
     asp=0,
     main="Masterstudiums-Hochschulen 
     der #30u30-Mitglieder des Jahrgangs 2018")
```

#### Jahrgang 2019

Erstellt das Studiumsnetzwerk des Jahrgangs 2019:

```{r Studiumsnetzwerk - Jahrgang 2019}
study19 <- delete.vertices(study, V(study)[(type == 1) & (entry != 2019)]) # selektiert aus dem Studiumsnetzwerk unter den #30u30 nur den Jahrgang 2019

study19 <- delete.vertices(study19, V(study19)[degree(study19, V(study19), mode="all")==0]) # löscht isolierte Knoten
```

Löscht mehrfache Kanten, damit bei der Indegree-Abfrage nur die einfache Anzahl der Verbindungen geliefert wird (nicht für jedes Jahr).

```{r Studiumsnetzwerk - Jahrgang 2019 - Kanten-Vereinfachung}
study19_simple <- simplify(study19, remove.multiple = TRUE)
```

Welche Hochschulen stechen insgesamt im Studiumsnetzwerk des Jahrgangs hervor, wo haben mehrere Personen studiert?

```{r Studiumsnetzwerk - Jahrgang 2019 - Indegree-Verteilung}
ind_study19 <- degree(study19_simple, V(study19_simple), mode="in")
sort(ind_study19,decreasing=TRUE)
```

##### Jahrgang 2019: Bachelorstudium

Erstellt Netzwerk des Jahrgangs 2019 nur mit Verbindungen zwischen den Mitgliedern und Hochschulen, wo diese im Bachelor studiert haben.

```{r Studiumsnetzwerk - Jahrgang 2019 - Bachelor}
study19_ba <- subgraph.edges(study19, E(study19)[relationship == " 2"]) # selektiert Kanten, bei denen es sich um Bachelor-Studium handelt

study19_ba <- delete.vertices(study19_ba, V(study19_ba)[degree(study19_ba, V(study19_ba), mode="all")]==0) # löscht isolierte Knoten

study19_ba1 <- simplify(study19_ba, remove.multiple = T) # Kantenvereinfachung
```

Bei welchen Hochschulen haben besonders viele Personen des Jahrgangs im Bachelor studiert?

```{r Studiumsnetzwerk - Jahrgang 2019 - Bachelor - Indegree-Verteilung}
ind_ba19 <- degree(study19_ba1, mode="in")
sort(ind_ba19, decreasing = T)
```

Erstellt den Plot der Beziehungen zwischen den Jahrgangsmitgliedern und Bachelor-Hochschulen:

```{r Studiumsnetzwerk - Jahrgang 2019 - Bachelor - Plot}
plot(study19_ba1,
     layout=layout_with_fr,
     edge.curved=curve_multiple(study19_ba1),
     vertex.size=3,
     vertex.frame.color="white",
     edge.color="grey",
     edge.curved=.2,
     edge.arrow.size=.00001,
     edge.arrow.color="white",
     vertex.label.degree=0, 
     vertex.label.cex=.25,
     vertex.label.family="sans",
     vertex.label.font=2,
     vertex.label.color="black", 
     asp=0,
     main="Bachelorstudiums-Hochschulen 
     der #30u30-Mitglieder des Jahrgangs 2019")
```
##### Jahrgang 2019 - Masterstudium
Erstellt Netzwerk des Jahrgangs 2019 nur mit Verbindungen zwischen den Mitgliedern und Hochschulen, wo diese im Master studiert haben.

```{r Studiumsnetzwerk - Jahrgang 2019 - Master}
study19_ma <- subgraph.edges(study19, E(study19)[relationship == " 3"]) # selektiert Kanten, bei denen es sich um Master-Studium handelt

study19_ma <- delete.vertices(study19_ma, V(study19_ma)[degree(study19_ma, V(study19_ma), mode="all")]==0) # löscht isolierte Knoten

study19_ma1 <- simplify(study19_ma, remove.multiple = T) # Kantenvereinfachung
```

Bei welchen Hochschulen haben besonders viele Personen des Jahrgangs im Master studiert?

```{r Studiumsnetzwerk - Jahrgang 2019 - Master - Indegree-Verteilung}

ind_ma19 <- degree(study19_ma1, mode="in")
sort(ind_ma19, decreasing = T)

```

Erstellt den Plot der Beziehungen zwischen den Jahrgangsmitgliedern und Master-Hochschulen:

```{r Studiumsnetzwerk - Jahrgang 2019 - Master - Plot}
plot(study19_ma1,
     layout=layout_with_fr,
     edge.curved=curve_multiple(study19_ma1),
     vertex.size=3,
     vertex.frame.color="white",
     edge.color="grey",
     edge.curved=.2,
     edge.arrow.size=.00001,
     edge.arrow.color="white",
     vertex.label.degree=0, 
     vertex.label.cex=.25,
     vertex.label.family="sans",
     vertex.label.font=2,
     vertex.label.color="black", 
     asp=0,
     main="Masterstudiums-Hochschulen 
     der #30u30-Mitglieder des Jahrgangs 2019")
```

#### Jahrgang 2020
Erstellt das Studiumsnetzwerk des Jahrgangs 2020:

```{r Studiumsnetzwerk - Jahrgang 2020}
study20<- delete.vertices(study, V(study)[(type == 1) & (entry != 2020)]) # selektiert aus dem Studiumsnetzwerk unter den #30u30 nur den Jahrgang 2020

study20 <- delete.vertices(study20, V(study20)[degree(study20, V(study20), mode="all")==0]) # löscht isolierte Knoten
```

Löscht mehrfache Kanten, damit bei der Indegree-Abfrage nur die einfache Anzahl der Verbindungen geliefert wird (nicht für jedes Jahr).

```{r Studiumsnetzwerk - Jahrgang 2020 - Kanten-Vereinfachung}
study20_simple <- simplify(study20, remove.multiple = TRUE)
```

An welchen Hochschulen haben mehrere Talente des Jahrgangs studiert?

```{r Studiumsnetzwerk - Jahrgang 2020 - Indegree-Verteilung}
ind_study20 <- degree(study20_simple, V(study20_simple), mode="in")
sort(ind_study20,decreasing=TRUE)
```

##### Jahrgang 2020: Bachelorstudium 

```{r Studiumsnetzwerk - Jahrgang 2020 - Bachelor}
study20_ba <- subgraph.edges(study20, E(study20)[relationship == " 2"]) # selektiert Kanten, bei denen es sich um BA-Studium handelt

study20_ba <- delete.vertices(study20_ba, V(study20_ba)[degree(study20_ba, V(study20_ba), mode="all")]==0) # löscht isolierte Knoten

study20_ba1 <- simplify(study20_ba, remove.multiple = T) # Kantenvereinfachung
```

An welchen Hochschulen haben mehrere Talente des Jahrgangs ein Bachelorstudium absolviert?

```{r Studiumsnetzwerk - Jahrgang 2020 - Indegree-Verteilung}
ind_ba20 <- degree(study20_ba1, mode="in")
sort(ind_ba20, decreasing = T)
```

Erzeugt den Plot der Beziehungen zwischen den Jahrgangsmitgliedern und Bachelor-Hochschulen:

```{r Studiumsnetzwerk - Jahrgang 2020 - Bachelor - Plot}
plot(study20_ba1,
     layout=layout_with_fr,
     edge.curved=curve_multiple(study20_ba1),
     vertex.size=3,
     vertex.frame.color="white",
     edge.color="grey",
     edge.curved=.2,
     edge.arrow.size=.00001,
     edge.arrow.color="white",
     vertex.label.degree=0, 
     vertex.label.cex=.25,
     vertex.label.family="sans",
     vertex.label.font=2,
     vertex.label.color="black", 
     asp=0,
     main="Bachelorstudiums-Hochschulen 
     der #30u30-Mitglieder des Jahrgangs 2020")
```

##### Jahrgang 2020: Masterstudium
Erstellt Netzwerk des Jahrgangs 2020 nur mit Verbindungen zwischen den Mitgliedern und Hochschulen, wo diese im Master studiert haben.

```{r Studiumsnetzwerk - Jahrgang 2020 - Master}
study20_ma <- subgraph.edges(study20, E(study20)[relationship == " 3"]) # selektiert Kanten, bei denen es sich um Master-Studium handelt

study20_ma <- delete.vertices(study20_ma, V(study20_ma)[degree(study20_ma, V(study20_ma), mode="all")]==0) # löscht isolierte Knoten

study20_ma1 <- simplify(study20_ma, remove.multiple = T) # Kantenvereinfachung
```

An welchen Hochschulen haben mehrere Talente des Jahrgangs ein Masterstudium absolviert?

```{r Studiumsnetzwerk - Jahrgang 2020 - Indegree-Verteilung}
ind_ma20 <- degree(study20_ma1, mode="in")
sort(ind_ma20, decreasing = T)
```

Erstellt den Plot der Beziehungen zwischen den Jahrgangsmitgliedern und Master-Hochschulen:

```{r Studiumsnetzwerk - Jahrgang 2020 - Master - Plot}
plot(study20_ma1,
     layout=layout_with_fr,
     edge.curved=curve_multiple(study20_ma1),
     vertex.size=3,
     vertex.frame.color="white",
     edge.color="grey",
     edge.curved=.2,
     edge.arrow.size=.00001,
     edge.arrow.color="white",
     vertex.label.degree=0, 
     vertex.label.cex=.25,
     vertex.label.family="sans",
     vertex.label.font=2,
     vertex.label.color="black", 
     asp=0,
     main="Masterstudiums-Hochschulen 
     der #30u30-Mitglieder des Jahrgangs 2020")
```

#### Jahrgang 2021

Erstellt das Studiumsnetzwerk des Jahrgangs 2021:

```{r Studiumsnetzwerk - Jahrgang 2021}
study21 <- delete.vertices(study, V(study)[(type == 1) & (entry != 2021)]) # selektiert aus dem Studiumsnetzwerk unter den #30u30 nur den Jahrgang 2021

study21 <- delete.vertices(study21, V(study21)[degree(study21, V(study21), mode="all")==0]) # löscht isolierte Knoten
```

Löscht bzw. vereinfacht mehrfache Kanten, damit bei der Indegree-Abfrage nur die einfache Anzahl der Verbindungen geliefert wird (nicht für jedes Jahr)

```{r Studiumsnetzwerk - Jahrgang 2021 - Kanten-Vereinfachung}
study21_simple <- simplify(study21, remove.multiple = TRUE)
```

Welchen Hochschulen stechen heraus, weil dort mehrere Talente des Jahrgangs ein Studium absolviert haben?

```{r Studiumsnetzwerk - Jahrgang 2021 - Indegree-Verteilung}
ind_study21 <- degree(study21_simple, V(study21_simple), mode="in")
sort(ind_study21,decreasing=TRUE)
```

##### Jahrgang 2021: Bachelorstudium
Erstellt Netzwerk des Jahrgangs 2021 nur mit Verbindungen zwischen den Mitgliedern und Hochschulen, wo diese im Bachelor studiert haben.

```{r Studiumsnetzwerk - Jahrgang 2021 - Bachelor}
study21_ba <- subgraph.edges(study21, E(study21)[relationship == " 2"]) # selektiert Kanten, bei denen es sich um Bachelor-Studium handelt

study21_ba <- delete.vertices(study21_ba, V(study21_ba)[degree(study21_ba, V(study21_ba), mode="all")]==0) # löscht isolierte Knoten

study21_ba1 <- simplify(study21_ba, remove.multiple = T) # Kantenvereinfachung
```

An welchen Hochschulen haben mehrere Talente des Jahrgangs ein Bachelorstudium absolviert?

```{r Studiumsnetzwerk - Jahrgang 2021 - Bachelor - Indegree-Verteilung}
ind_ba21 <- degree(study21_ba1, mode="in")
sort(ind_ba21, decreasing = T)
```

Erstellt den Plot der Beziehungen zwischen den Jahrgangsmitgliedern und Bachelor-Hochschulen:

```{r Studiumsnetzwerk - Jahrgang 2021 - Bachelor - Plot}
plot(study21_ba1,
     layout=layout_with_fr,
     edge.curved=curve_multiple(study21_ba1),
     vertex.size=3,
     vertex.frame.color="white",
     edge.color="grey",
     edge.curved=.2,
     edge.arrow.size=.00001,
     edge.arrow.color="white",
     vertex.label.degree=0, 
     vertex.label.cex=.25,
     vertex.label.family="sans",
     vertex.label.font=2,
     vertex.label.color="black", 
     asp=0,
     main="Bachelorstudiums-Hochschulen 
     der #30u30-Mitglieder des Jahrgangs 2021")
```

##### Jahrgang 2021: Masterstudium
Erstellt Netzwerk des Jahrgangs 2021 nur mit Verbindungen zwischen den Mitgliedern und Hochschulen, wo diese im Master studiert haben.

```{r Studiumsnetzwerk - Jahrgang 2021 - Master}
study21_ma <- subgraph.edges(study21, E(study21)[relationship == " 3"]) # selektiert Kanten, bei denen es sich um Master-Studium handelt

study21_ma <- delete.vertices(study21_ma, V(study21_ma)[degree(study21_ma, V(study21_ma), mode="all")]==0) # löscht isolierte Knoten

study21_ma1 <- simplify(study21_ma, remove.multiple = T) # Kantenvereinfachung
```

Welche Hochschulen stechen hervor, wenn es darum geht, wo die Jahrgangsmitglieder ein Masterstudium gemacht haben? Wo überschneiden sich deren Wege?

```{r Studiumsnetzwerk - Jahrgang 2021 - Indegree-Verteilung}
ind_ma21 <- degree(study21_ma1, mode="in")
sort(ind_ma21, decreasing = T)
```

Erstellt den Plot der Beziehungen zwischen den Jahrgangsmitgliedern und Master-Hochschulen:

```{r Studiumsnetzwerk - Jahrgang 2021 - Master - Plot}
plot(study21_ma1,
     layout=layout_with_fr,
     edge.curved=curve_multiple(study21_ma1),
     vertex.size=3,
     vertex.frame.color="white",
     edge.color="grey",
     edge.curved=.2,
     edge.arrow.size=.00001,
     edge.arrow.color="white",
     vertex.label.degree=0, 
     vertex.label.cex=.25,
     vertex.label.family="sans",
     vertex.label.font=2,
     vertex.label.color="black", 
     asp=0,
     main="Masterstudiums-Hochschulen 
     der #30u30-Mitglieder des Jahrgangs 2021")
```

### Populärste Hochschulen
Abschließend werden Ego-Netzwerke der drei populärsten Hochschulen erstellt, an denen die meisten der 150 #30u30-Mitglieder studiert haben. Die erstellten Netzwerke werden die Personen beinhalten, die an der jew. Hochschule studiert haben, sowie deren weitere Verbindungen - um Überschneidungen zwischen Hochschulabsolvent:innen zu analysieren.

#### Universität Leipzig
```{r Egonetzwerk Universität Leipzig}
members

# selektiert aus dem Mitgliedernetzwerk alle Personen, die mit der Uni Leipzig über zwei Schritte verbunden sind.
unileipzig <- make_ego_graph(members, order = 2, nodes = V(members)$name == "Universität Leipzig", mode ="all")
unileipzig

V(unileipzig[[1]])[name=="Universität Leipzig"]$color <- "orange" # färbt Knoten der Uni Leipzig orange

unileipzig_simple <- simplify(unileipzig[[1]], remove.multiple=T) # Kantenvereinfachung
```

Listet die Indegree-Werte im Ego-Netzwerk auf, um zu prüfen, welche Organisationen bei den Leipziger Absolvent:innen populär sind.

```{r Ego-Netzwerk Universität Leipzig - Indegree-Verteilung}
ind_leipzig <- degree(unileipzig_simple, V(unileipzig_simple), mode="in")
sort(ind_leipzig,decreasing=TRUE)
```

Erzeugt den Plot des Ego-Netzwerks:
```{r Ego-Netzwerk Universität Leipzig - Plot}
plot(unileipzig[[1]], 
     main="Ego-Netzwerk Universität Leipzig",
     layout=layout_with_fr,
     edge.arrow.size=.000001,
     edge.arrow.color="white",
     vertex.frame.color="white",
     edge.curved=.2,
     vertex.size=ind_leipzig*.8,
     vertex.label.cex=.2,
     vertex.label.family="sans",
     vertex.label.color="black",
     sub="Beziehungen zweiten Grads 
     - Studierende der Universität Leipzig und deren Verbindungen",
     asp=0)

# Plot ohne Labels
plot(unileipzig[[1]], 
     main="Ego-Netzwerk Universität Leipzig",
     vertex.frame.color="white",
     edge.arrow.size=.00001, 
     edge.curved=.2,
     vertex.size=3,
     vertex.label=NA,
     vertex.label.family="sans",
     vertex.label.color="black",
     sub="Beziehungen des zweiten Grades",
     asp=0)
```


#### Universität Mannheim 
```{r Egonetzwerk Universität Mannheim erstellen}
members

# selektiert aus dem Mitgliedernetzwerk alle Personen, die mit der Uni Mannheim über zwei Schritte verbunden sind.
unimannheim <- make_ego_graph(members, order = 2, nodes = V(members)$name == "Universität Mannheim", mode ="all")
unimannheim

V(unimannheim[[1]])[name=="Universität Mannheim"]$color <- "orange" # färbt Knoten der Uni Mannheim orange

unimannheim_simple <- simplify(unimannheim[[1]], remove.multiple=T) # Kantenvereinfachung
```

Listet die Indegree-Werte im Ego-Netzwerk auf, um zu prüfen, welche Organisationen bei den Mannheimer Absolvent:innen populär sind.

```{r Ego-Netzwerk Universität Mannheim - Indegree-Verteilung}
ind_mannheim <- degree(unimannheim_simple, V(unimannheim_simple), mode="in")
sort(ind_mannheim,decreasing=TRUE)
```

Erzeugt den Plot des Ego-Netzwerks:
```{r Ego-Netzwerk Universität Mannheim - Plot}
plot(unimannheim_simple, 
     main="Ego-Netzwerk Universität Mannheim",
     layout=layout_with_fr,
     edge.arrow.size=.000001,
     edge.arrow.color="white",
     vertex.frame.color="white",
     edge.curved=.2,
     vertex.size=ind_mannheim*.8,
     vertex.label.cex=.2,
     vertex.label.family="sans",
     vertex.label.color="black",
     sub="Beziehungen zweiten Grads 
     - Studierende der Universität Mannheim und deren Verbindungen",
     asp=0)
```

#### Universität Hohenheim

```{r Egonetzwerk Universität Hohenheim erstellen}
members

# selektiert aus dem Mitgliedernetzwerk alle Knoten, die mit der Uni Hohenheim über zwei Schritte verbunden sind.
unihohenheim <- make_ego_graph(members, order = 2, nodes = V(members)$name == "Universität Hohenheim", mode ="all")

unihohenheim[[1]]

V(unihohenheim[[1]])[name=="Universität Hohenheim"]$color <- "orange" # färbt Knoten der Uni Hohenheim orange

unihohenheim_simple <- simplify(unihohenheim[[1]], remove.multiple=T) # Kantenvereinfachung
```

Listet die Indegree-Werte im Ego-Netzwerk auf, um zu prüfen, welche Organisationen bei den Hohenheimer Absolvent:innen populär sind.

```{r Ego-Netzwerk Universität Hohenheim - Indegree-Verteilung}
ind_hohenheim <- degree(unihohenheim_simple, V(unihohenheim_simple), mode="in")
sort(ind_hohenheim,decreasing=TRUE)
```

Erzeugt den Plot des Ego-Netzwerks:

```{r Ego-Netzwerk Universität Hohenheim - Plot}
plot(unihohenheim_simple, 
     main="Ego-Netzwerk Universität Hohenheim",
     layout=layout_with_fr,
     edge.arrow.size=.000001,
     edge.arrow.color="white",
     vertex.frame.color="white",
     edge.curved=.2,
     vertex.size=ind_hohenheim*.8,
     vertex.label.cex=.25,
     vertex.label.family="sans",
     vertex.label.color="black",
     sub="Beziehungen zweiten Grads 
     - Studierende der Universität Hohenheim und deren Verbindungen",
     asp=0)
```

#### Vergleichende Berechnungen: Populärste Universitäten

Abschließend werden verschiedene Berechnungen zu den drei populärsten Universitäten angestellt, um mögliche Vergleiche zu ziehen.
1. Vergleich der Dichtewerte in den Netzwerken der drei Universitäten und deren Absolvent:innen:

```{r Dichte}
# Uni Leipzig
  edge_density(unileipzig_simple) # 0,01343785

# Uni Mannheim
  edge_density(unimannheim_simple) # 0,01208791

  # Uni Hohenheim 
  edge_density(unihohenheim_simple) # 0,01212537
```

2. Vergleich der mittleren Pfaddistanz und des Netzwerkdurchmessers in den Netzwerken der drei Universitäten und deren Absolvent:innen:

```{r Mittlere Pfaddistanz & Diameter}
# Mittlere Pfaddistanz
  #Uni Leipzig
  mean_distance(unileipzig_simple, directed=F) # 3,449496
  # Uni Mannheim
  mean_distance(unimannheim_simple, directed=F) # 3,507204
  # Uni Hohenheim
  mean_distance(unihohenheim_simple, directed=F) #3,49531

# Durchmesser / Diameter / Farthest Vertices
  # Uni Leipzig
  diameter(unileipzig_simple, directed=F) # Ergebnis: 4
  farthest_vertices(unileipzig_simple, directed=F) # AIESEC; Flair Magazine
  # Uni Mannheim
  diameter(unimannheim_simple, directed=F) # Ergebnis: 4
  farthest_vertices(unimannheim_simple, directed=F) 
    # Academy for Excellence, Communication Consultants
  # Uni Hohenheim
  diameter(unihohenheim_simple, directed=F) # Ergebnis: 4
  farthest_vertices(unihohenheim_simple, directed=F) 
    # Academy for Excellence, Deutscher Bundestag
```

3. Vergleich der Cluster, Modularität etc. in den Netzwerken der drei Universitäten und deren Absolvent:innen:

```{r Clustering}

# Uni Leipzig
  cl_leipzig <- cluster_walktrap(unileipzig_simple)
  membership(cl_leipzig)
  modularity(cl_leipzig) # 0,5970486
  communities(cl_leipzig) # 11
  components(unileipzig_simple) # 1 Komponente = alle 95 Knoten verbunden
  
# Uni Mannheim
  cl_mannheim <- cluster_walktrap(unimannheim_simple)
  membership(cl_mannheim)
  modularity(cl_mannheim) # 0,7174268
  communities(cl_mannheim) # 8
  components(unimannheim_simple) # 1 Komponente = alle 91 Knoten verbunden

# Uni Hohenheim
  cl_hohenheim <- cluster_walktrap(unihohenheim_simple)
  membership(cl_hohenheim)
  modularity(cl_hohenheim) # 0,6786668
  communities(cl_hohenheim) # 9
  components(unihohenheim_simple) # 1 Komponente = alle 94 Knoten verbunden
```
  
## Mitglieder ohne akademischen Abschluss

Die Mehrheit der 150 Personen in den #30u30-Jahrgängen 2017 bis 2021 hat studiert, nur vier Personen sind mit Ausbildung oder Abitur in den Job gestartet. Was unterscheidet sie von den akademisierten Mitgliedern?

```{r Teilnetzwerk der Mitglieder ohne Studiumsabschluss}

members
nonacademic <- delete.vertices(members, V(members)[(type == 1) & (education > 3)]) # löscht alle #30u30-Mitglieder mit mind. BA-Abschluss

nonacademic <- delete.vertices(nonacademic, V(nonacademic)[degree(nonacademic, V(nonacademic), mode = "all") == 0]) # löscht isolierte Knoten

nonacademics <- simplify(nonacademic, remove.multiple = TRUE) # Kantenvereinfachung

# Passt die Visualisierung der Knoten an (Jahrgänge verschieden eingefärbt)
V(nonacademics)[V(nonacademics)$entry == 2018]$color <- "gold1" # JG 18 wird gelb
V(nonacademics)[V(nonacademics)$entry == 2020]$color <- "firebrick" # JG 20 wird rot
V(nonacademics)[V(nonacademics)$type == 1]$shape <- "circle" # Personen werden als Kreis angezeigt

# Plot
plot(nonacademics,
     layout=layout_with_kk,
     edge.curved=curve_multiple(nonacademics), # verhindert, dass sich Kanten überlagern
     vertex.frame.color=NA,
     edge.curved=.2,
     edge.color="grey80",
     vertex.size=8,
     vertex.label.cex=.5,
     vertex.label.family="sans",
     vertex.label.font=2,
     vertex.label.color="black",
     main="Netzwerk der #30u30-Mitglieder ohne Studium",
     sub="Gelber Kreis = Jahrgangsmitglied 2018, Roter Kreis = Jahrgangsmitglied 2020",
     edge.arrow.size=.000001,
     edge.arrow.color="white",
     asp=0)
```

Wie viele Verbindungen haben die vier Personen jeweils?

```{r Teilnetzwerk der Mitglieder ohne Studiumsabschluss - Outdegrees}
outd_nonacad <- degree(nonacademics, V(nonacademics), mode="out")
outd_nonacad # 1,2,4,8

# Durchschnittliche Anzahl an Verbindungen
(1+2+4+8)/4 # Ergebnis: Im Durchschnitt hat jede Person ohne akadem. Abschluss  3,75 Verbindungen

```

## Lehrtätigkeiten an Hochschulen

Einige der #30u30 waren oder sind aktuell als Dozent:innen an Hochschulen tätig. An welchen Hochschulen sammeln sich diese, gibt es Überschneidungen und handelt es sich primär um die ehemaligen Ausbildungsstätten der Personen?

Hierzu werden diejenigen, die als Dozent:innen tätig sind oder waren selektiert und deren Netzwerk betrachtet.

```{r Teilnetzwerk Dozent:innen erstellen}
members # ruft #30u30-Netzwerk auf

teachers <- subgraph.edges(members, E(members)[relationship == 15]) # selektiert Kanten, bei denen es sich um Lehrtätigkeiten handelt

teachers <- delete.vertices(teachers, V(teachers)[degree(teachers, V(teachers), mode="all")==0]) # löscht Isolates
```

Färbt Jahrgänge und Hochschultypen unterschiedlich ein, um diese zu unterscheiden.

```{r Teilnetzwerk Dozent:innen: Visualisierungsanpassung}
# Knoten der Mitglieder der einzelnen Jahrgänge verschieden einfärben:
V(teachers)[V(teachers)$entry == 2017]$color <- "firebrick1" 
V(teachers)[V(teachers)$entry == 2018]$color <- "firebrick2" 
V(teachers)[V(teachers)$entry == 2019]$color <- "firebrick3" 
V(teachers)[V(teachers)$entry == 2020]$color <- "firebrick" 
V(teachers)[V(teachers)$entry == 2021]$color <- "firebrick4"  

# Einfärben der versch. Hochschultypen:
V(teachers)[ownership==1]$color <- "palegreen3" # Staatliche Unis hellgrün
V(teachers)[ownership==2]$color <- "olivedrab3" # Staatliche FHs hell-oliv
V(teachers)[ownership==3]$color <- "palegreen4" # Private Unis dunkelgrün
V(teachers)[ownership==4]$color <- "olivedrab4" # Private FHs dunkeloliv
V(teachers)[ownership==5]$color <- "darkseagreen" # Business Schools dunkel-pastellgrün
V(teachers)[ownership==6]$color <- "darkseagreen1" # Weiterbildungsakademie pastellgrün
V(teachers)[ownership==7]$color <- "green" # Berufsschule knallgrün
V(teachers)[ownership==8]$color <- "grey" # Sonstige HS grau
```

Kanten werden vereinfacht: Mehrfache Kanten nur einmal anzeigen:

```{r Teilnetzwerk Dozent:innen: Kantenvereinfachung}
teachers1 <- simplify(teachers, remove.multiple = T)
```

Wie viele Personen sind als Dozent:in tätig? Auf wie viele Hochschulen verteilen sie sich und um welche Hochschultypen handelt es sich?
Berechnung der Anzahl der verschiedenen Knotentypen:

```{r Teilnetzwerk Dozent:innen: Knotentypen}
# Wie viele #30u30-Mitglieder haben oder hatten einen Lehrauftrag?
teachers_ppl <- V(teachers)$type == 1
sum(teachers_ppl) # 12 Personen

# Wie viele Hochschulen?
teachers_hs <- V(teachers)$category == 5
sum(teachers_hs)# 13 Hochschulen

# Welcher Hochschultyp?
teachers_uni_staat <- V(teachers)$ownership == "1"
sum(teachers_uni_staat)  # 3 staatliche Unis

teachers_uni_priv <- V(teachers)$ownership == "2"
sum(teachers_uni_priv) # 0 Privat-Unis

teachers_fh_staat <- V(teachers)$ownership == "3"
sum(teachers_fh_staat) # 4 Staatliche FHs

teachers_fh_priv <- V(teachers)$ownership == "4"
sum(teachers_fh_priv) # 4 private FHs

teachers_bs <- V(teachers)$ownership == "5"
sum(teachers_bs, na.rm=T)
# 0 Business Schools

teachers_wb <- V(teachers)$ownership == "6"
sum(teachers_wb, na.rm=T)
# 0 Weiterbildungsakademien

```
Wie viele Personen haben / hatten in den jew. Jahrgängen einen Lehrauftrag?
```{r Teilnetzwerk Dozent:innen: Anzahl in jew. Jahrgängen}
# Wie viele #30u30-Mitglieder aus JG 2017?
teachers_17 <- V(teachers)$entry == "2017"
sum(teachers_17) # 5 Personen

# Wie viele aus JG 2018?
teachers_18 <- V(teachers)$entry == "2018"
sum(teachers_18) # 3 Personen

# Wie viele aus JG 2019?
teachers_19 <- V(teachers)$entry == "2019"
sum(teachers_19) # 1 Person

# Wie viele aus JG 2020?
teachers_20 <- V(teachers)$entry == "2020"
sum(teachers_20) # 1 Person

# Wie viele aus Jahrgang 2021?
teachers_21 <- V(teachers)$entry == "2021"
sum(teachers_21) # 2 Personen
```

An welchen Hochschulen lehren mehrere #30u30-Mitglieder?
```{r Teilnetzwerk Dozent:innen: Indegree-Verteilung}
ind_teacher <- degree(teachers1, mode="in")
sort(ind_teacher, decreasing = T)
```

Wer hat mehrere Lehraufträge?
```{r Teilnetzwerk Dozent:innen: Outdegree-Verteilung}
outd_teacher <- degree(teachers1, mode="out")
sort(outd_teacher, decreasing = T)

```

Generiert den Plot
```{r Teilnetzwerk Dozent:innen: Plot}
plot(teachers1,
     layout=layout_with_kk,
     edge.curved=curve_multiple(teachers1),
     vertex.size=3,
     vertex.frame.color="white",
     edge.color="grey",
     edge.curved=.2,
     edge.arrow.size=.00001,
     edge.arrow.color="white",
     vertex.label.degree=0, 
     vertex.label.cex=.4,
     vertex.label.family="sans",
     vertex.label.font=2,
     vertex.label.color="black", 
     asp=0,
     main="Netzwerk der #30u30-Mitglieder der Jahrgänge 2017 bis 2021 
     mit Lehrauftrag und den entspr. Hochschulen")
```
Lehren die Personen überwiegend an Hochschulen, an denen sie selbst studiert haben? Hierzu werden zu allen Dozent:innen Ego-Netzwerke erstellt, zusammengefügt und die Studiumsverbindungen selektiert, um Studiums- und Lehrstätte abzugleichen.

```{r Teilnetzwerk Dozent:innen: Ego-Netzwerke aller Lehrpersonen}
bisswanger <- subgraph <- make_ego_graph(members, order = 1,  c("Luisa Bisswanger"))
bohn <- subgraph <- make_ego_graph(members, order = 1,  c("Ricarda Bohn"))
bolzenius <- subgraph <- make_ego_graph(members, order = 1,  c("Jule Bolzenius"))
erdmeier <- subgraph <- make_ego_graph(members, order = 1,  c("Kristine Erdmeier"))
klarholz <- subgraph <- make_ego_graph(members, order = 1,  c("Lisa Klarholz"))
ludwig <- subgraph <- make_ego_graph(members, order = 1,  c("Alina Ludwig"))
lutermann <- subgraph <- make_ego_graph(members, order = 1,  c("Katharina Lutermann"))
nebe <- subgraph <- make_ego_graph(members, order = 1,  c("Tina Nebe"))
sievers <- subgraph <- make_ego_graph(members, order = 1,  c("Felix Sievers"))
friese <- subgraph <- make_ego_graph(members, order = 1,  c("Eva-Maria Friese"))
kirchenbauer <- subgraph <- make_ego_graph(members, order = 1,  c("Alena Kirchenbauer"))
reidinger <- subgraph <- make_ego_graph(members, order = 1,  c("Felix Reidinger-Tomschin"))

# Zu einem Graph zusammenfügen, indem die Ego-Netzwerke doppelt vom Gesamtnetzwerk subtrahiert werden
tn_teachers <- members - (members - bisswanger[[1]] - bohn[[1]] - bolzenius[[1]] - erdmeier[[1]] - klarholz[[1]] - ludwig[[1]] - lutermann[[1]] - nebe[[1]] - sievers[[1]] - friese[[1]] - kirchenbauer[[1]]- reidinger[[1]]) 
# Isolierte Knoten löschen
tn_teachers <- delete_vertices (tn_teachers, V(tn_teachers)[degree(tn_teachers, mode="all")=="0"])
```

Um zu vergleichen, wo die Personen studiert hat und wo sie oder er lehrt, werden alle anderen Beziehungen gelöscht (Arbeitsbeziehungen etc.):
```{r Teilnetzwerk Dozent:innen: Subgraph - Kanten selektieren}
tn_teachers1 <- delete.edges(tn_teachers, E(tn_teachers)[relationship == " 8"])
tn_teachers2 <- delete.edges(tn_teachers1, E(tn_teachers1)[relationship == " 9"])
tn_teachers3 <- delete.edges(tn_teachers2, E(tn_teachers2)[relationship == " 7"])
tn_teachers4 <- delete.edges(tn_teachers3, E(tn_teachers3)[relationship == "10"])
tn_teachers5 <- delete.edges(tn_teachers4, E(tn_teachers4)[relationship == "11"])
tn_teachers6 <- delete.edges(tn_teachers5, E(tn_teachers5)[relationship == "12"])
tn_teachers7 <- delete.edges(tn_teachers6, E(tn_teachers6)[relationship == "13"])
tn_teachers8 <- delete.edges(tn_teachers7, E(tn_teachers7)[relationship == "14"])
```

Isolierte Knoten löschen und Kanten, die eine Lehrtätigkeit darstellen, zur Hervorhebung orange einfärben:

```{r Teilnetzwerk Dozent:innen: Subgraph - Isolates löschen, Kanten färben}
tn_teachers8 <- delete_vertices (tn_teachers8, V(tn_teachers8)[degree(tn_teachers8, mode="all")=="0"]) # isolierte Knoten löschen

E(tn_teachers8)[E(tn_teachers8)$relationship == 15]$color <- "orange" # Kanten, die eine Lehrtätigkeit darstellen, werden orange gefärbt
```

Erstellt den Plot des Subgraphs der Dozent:innen und all deren Verbindungen:

```{r Teilnetzwerk Dozent:innen: Subgraph inkl. weitere Verbindungen - Plot}
plot(tn_teachers8,
     layout=layout_with_kk,
     edge.curved=curve_multiple(tn_teachers8),
     vertex.size=3,
     vertex.frame.color="white",
     edge.curved=.2,
     edge.arrow.size=.00001,
     edge.arrow.color="white",
     vertex.label.degree=0, 
     vertex.label.cex=.25,
     vertex.label.family="sans",
     vertex.label.font=2,
     vertex.label.color="black", 
     asp=0,
     main="Netzwerk der #30u30-Mitglieder der JG 2017 bis 2021 
     mit Lehrauftrag")
```


## PR-Initiativen, Branchenverbände & Vereine
Nach der Betrachtung der Arbeits- und Hochschulnetzwerke werden nun andere Aspekte beleuchtet: Zunächst werden die Mitglieder in den Fokus genommen, die sich in Initiativen oder Vereinen mit Kommunikations- / PR-Bezug engagierten oder engagieren.

```{r Mitgliedschaft in Initiativen: Erstellen des Teilnetzwerks}
members # ruft Mitgliedernetzwerk ab

initiative <- delete.vertices(members, V(members)[(type==2) & (category!=6)])
# löscht alle Organisationen, bei denen es sich nicht um Vereine, Verbände oder Initiativen handelt

# Löschen der Edges, bei denen es sich nicht um eine Mitgliedschaft (relationship == 11) oder eine leitende Vereins-Tätigkeit (relationship == 13) handelt, um Arbeitsbeziehungen bei Verbänden auszuklammern
initiative_mem <- delete.edges(initiative, E(initiative)[relationship>13]) 

initiative_mem1 <- delete.edges(initiative_mem, E(initiative_mem)[relationship<11]) 
```

Zur Unterscheidung werden die Knoten je nach Jahrgangszugehörigkeit unterschiedlich eingefärbt. 

```{r Mitgliedschaft in Initiativen: Einfärben der Jahrgänge}
V(initiative_mem1)[V(initiative_mem1)$entry == 2017]$color <- "firebrick1" # JG 17 wird gelb
V(initiative_mem1)[V(initiative_mem1)$entry == 2018]$color <- "firebrick2" # JG 18 wird orange
V(initiative_mem1)[V(initiative_mem1)$entry == 2019]$color <- "firebrick3" # JG 19 wird dunkelorange
V(initiative_mem1)[V(initiative_mem1)$entry == 2020]$color <- "firebrick" # JG 20 wird rot
V(initiative_mem1)[V(initiative_mem1)$entry == 2021]$color <- "firebrick4" # JG 21 wird dunkelrot
```

Da bei der reinen Selektion von Knoten, bei denen es sich um Vereine o. ä. handelt (category == 6), auch noch Vereine ohne PR-/Komm.-Bezug im Netzwerk enthalten sind, werden diese nun manuell entfernt.

```{r Mitgliedschaft in Initiativen: Löschen von unpassenden Nodes}
initiative_mem1 <- delete_vertices (initiative_mem1, V(initiative_mem1)[degree(initiative_mem1, mode="all")=="0"]) # löscht isolierte Knoten

# Löschen von Vereinen, bei denen es sich nicht um PR-/Kommunikations-bezogene (Branchen-)Vereine handelt
initiative_mem1 <- delete_vertices (initiative_mem1, V(initiative_mem1)[name=="30xFriends"]) 
initiative_mem1 <- delete_vertices (initiative_mem1, V(initiative_mem1)[name=="Young Königswinter Alumni Association"])
initiative_mem1 <- delete_vertices (initiative_mem1, V(initiative_mem1)[name=="Debattierclub JGU Mainz"])
initiative_mem1 <- delete_vertices (initiative_mem1, V(initiative_mem1)[name=="Deutsche Debattiergesellschaft"])
initiative_mem1 <- delete_vertices (initiative_mem1, V(initiative_mem1)[name=="AIESEC"])
initiative_mem1 <- delete_vertices (initiative_mem1, V(initiative_mem1)[name=="enactus München"])

# Isolates löschen
initiative_mem1 <- delete_vertices (initiative_mem1, V(initiative_mem1)[degree(initiative_mem1, mode="all")=="0"]) 

# Doppelte / Mehrfache Kanten löschen
initiative_mem2 <- simplify(initiative_mem1, remove.multiple = TRUE,edge.attr.comb = igraph_opt("edge.attr.comb"))
```

Nun wird das Netzwerk der #30u30-Mitglieder geplottet, die in PR-/Kommunikations-Vereinen (o. ä.) aktiv waren oder sind.

```{r Mitgliedschaft in Initiativen: Plot}
plot(initiative_mem2,
     edge.curved=curve_multiple(initiative_mem2),
     layout=layout_with_fr,
     vertex.size=3,
     vertex.label.dist=.2,
     vertex.label.cex=.2,
     vertex.label.family="sans",
     vertex.label.color="black",
     edge.arrow.size=.000001,
     edge.arrow.color="white",
     asp=0,
     main="Mitgliedschaft in PR-Initiativen oder -Vereinen",
     sub="Netzwerk der entspr. #30u30-Mitglieder der Jahrgänge 2017 - 2021")
```
Wie viele der 150 Personen haben sich engagiert und auf wie viele Verbände / Vereine konzentriert sich deren Engagement?

```{r Mitgliedschaft in Initiativen: Anzahl der Knotentypen}
# Wie viele Verbände und Vereine sind im Netzwerk enthalten?
vereine <- V(initiative_mem1)$category == 6
sum(vereine, na.rm=T) # Ergebnis: 20

# Wie viele der #30u30-Mitglieder sind im Netzwerk enthalten / waren bzw. sind Mitglied eines Vereins o. einer Initiative?
vereine_mem <- V(initiative_mem1)$type == 1
sum(vereine_mem, na.rm=T) # Ergebnis: 39

# Wie hoch ist der prozentuale Anteil der Vereinsmitglieder an allen 150 #30u30-Alumni?
100*(39/150) # 26 %
```
Bei der Mitgliedschaft in Vereinen o. ä. wird zwischen einfacher Mitgliedschaft und einer leitenden Rolle unterschieden, etwa wenn jemand die Initiative selbst gegründet oder eine Vorstandsrolle inne hat(te). Wie viele Personen die beiden Rollen ausgefüllt haben, wird im folgenden berechnet.

```{r Mitgliedschaft in Initiativen: Art des Engagements}
# Wie viele der Personen hatten eine leitende Position / Gründerrolle=
verein_leadership <- subgraph.edges(initiative_mem1, E(initiative_mem1)[relationship == 13])
verein_leaders <- V(verein_leadership)$type == 1
sum(verein_leaders, na.rm=T) # Ergebnis: 23 

# Wie viele davon waren "nur" Mitglied ?
verein_membership <- subgraph.edges(initiative_mem1, E(initiative_mem1)[relationship == 11])
verein_members <- V(verein_membership)$type == 1
sum(verein_members, na.rm=T) # Ergebnis: 21 Personen
```
Welche Verbände, Vereine oder Initiativen verbinden mehrere Personen?

```{r Mitgliedschaft in Initiativen: Indegree-Verteilung}
ind_initiative <- degree(initiative_mem2, mode="in")
ind_initiative
sort(ind_initiative, decreasing = T)
```
Das Netzwerk wird geplottet, um die Verbindungen und insbesondere die zentralen Vereine (o. ä.) sichtbar zu machen.

```{r Mitgliedschaft in Initiativen: Plot - indegree-skaliert}
plot(initiative_mem2,
     layout=layout_with_fr,
     edge.curved=curve_multiple(initiative_mem2),
     edge.curved=.8,
     vertex.size=ind_initiative*.8,
     vertex.label.cex=.3,
     vertex.label.family="sans",
     vertex.label.color="black",
     edge.arrow.size=.000001,
     edge.arrow.color="white",
     asp=0,
     main="Mitgliedschaft in PR-Initiativen oder Vereinen",
     sub="Netzwerk der entspr. #30u30-Mitglieder der Jahrgänge 2017 - 2021,
     Verbindungen bis März 2022, 
     Knotengröße entspr. Indegree-Wert und mit Faktor 0,8 skaliert")
```
### Veränderungen von Aufnahmejahr bis heute

Hat sich das Engagement der #30u30-Mitglieder vom jeweiligen Aufnahmejahr bis heute verändert, etwa weil die Aufnahme bei #30u30 die Awareness für Netzwerke und Vereine geschärft hat? Dazu wird betrachtet, wie viele Personen sich bis zum jew. Aufnahmejahr schon engagiert haben.

```{r Mitgliedschaft in Initiativen - Bis Aufnahme - Netzwerk erstellen}
members_preentry

initiative_preentry <- delete.vertices(members_preentry, V(members_preentry)[(type==2) & (category!=6)])

# Löschen der Edges, bei denen es sich nicht um eine Mitgliedschaft (relationship == 11) oder eine leitende Tätigkeit (relationship == 13) handelt, sodass Arbeitsbeziehungen bei Verbänden gelöscht werden
initiative_preentry1 <- delete.edges(initiative_preentry, E(initiative_preentry)[relationship>13]) 

initiative_preentry2 <- delete.edges(initiative_preentry1, E(initiative_preentry1)[relationship<11]) 
```

Zur Unterscheidung werden die Knoten je nach Jahrgangszugehörigkeit unterschiedlich eingefärbt. 

```{r Mitgliedschaft in Initiativen - Bis Aufnahme - Visualisierungsanpassung}

# Zur besseren Unterscheidung werden die Mitglieder der einzelnen Jahrgänge verschieden eingefärbt
V(initiative_preentry2)[V(initiative_preentry2)$entry == 2017]$color <- "firebrick1" # JG 17 wird gelb
V(initiative_preentry2)[V(initiative_preentry2)$entry == 2018]$color <- "firebrick2" # JG 18 wird orange
V(initiative_preentry2)[V(initiative_preentry2)$entry == 2019]$color <- "firebrick3" # JG 19 wird dunkelorange
V(initiative_preentry2)[V(initiative_preentry2)$entry == 2020]$color <- "firebrick" # JG 20 wird rot
V(initiative_preentry2)[V(initiative_preentry2)$entry == 2021]$color <- "firebrick4" # JG 21 wird dunkelrot

```

Löschen der Knoten ohne Verbindungen und die keine PR-Initiativen sind.
```{r Mitgliedschaft in Initiativen - Bis Aufnahme - Löschen unpassender Nodes}

# Isolates löschen
initiative_preentry2 <- delete_vertices (initiative_preentry2, V(initiative_preentry2)[degree(initiative_preentry2, mode="all")=="0"]) 

# Löschen von Vereinen, bei denen es sich nicht um Kommunikations-bezogene (Branchen)vereine handelt
initiative_preentry2 <- delete_vertices (initiative_preentry2, V(initiative_preentry2)[name=="30xFriends"]) 
initiative_preentry2 <- delete_vertices (initiative_preentry2, V(initiative_preentry2)[name=="Young Königswinter Alumni Association"])
initiative_preentry2 <- delete_vertices (initiative_preentry2, V(initiative_preentry2)[name=="Debattierclub JGU Mainz"])
initiative_preentry2 <- delete_vertices (initiative_preentry2, V(initiative_preentry2)[name=="Deutsche Debattiergesellschaft"])
initiative_preentry2 <- delete_vertices (initiative_preentry2, V(initiative_preentry2)[name=="AIESEC"])
initiative_preentry2 <- delete_vertices (initiative_preentry2, V(initiative_preentry2)[name=="enactus München"])

# Doppelte / Mehrfache Kanten löschen
initiative_preentry2 <- simplify(initiative_preentry2, remove.multiple = TRUE,edge.attr.comb = igraph_opt("edge.attr.comb"))

# Isolates löschen
initiative_preentry2 <- delete_vertices (initiative_preentry2, V(initiative_preentry2)[degree(initiative_preentry2, mode="all")=="0"]) 

```

Generiert den Plot der Initiativen-Mitglieder:
```{r Mitgliedschaft in Initiativen - Bis Aufnahme - Plot}
plot(initiative_preentry2,
     edge.curved=curve_multiple(initiative_preentry2),
     layout=layout_with_fr,
     vertex.size=3,
     vertex.label.dist=.2,
     vertex.label.cex=.2,
     vertex.label.family="sans",
     vertex.label.color="black",
     edge.arrow.size=.000001,
     edge.arrow.color="white",
     asp=0)
```

Zählt wie viele der jew. Knotentypen im Netzwerk enthalten sind: Wie viele Personen und wie viele Vereine o. ä. ?

```{r Mitgliedschaft in Initiativen - Bis Aufnahme - Anzahl der Knotentypen}

# Wie viele Verbände und Vereine sind im Netzwerk enthalten?
vereine_bisaufnahme <- V(initiative_preentry2)$category == 6
sum(vereine_bisaufnahme, na.rm=T)
# Ergebnis: 19

# Wie viele der #30u30-Mitglieder sind im Netzwerk enthalten / waren bzw. sind Mitglied eines Vereins o. einer Initiative?
vereine_bisaufn_mem <- V(initiative_preentry2)$type == 1
sum(vereine_bisaufn_mem, na.rm=T)
# Ergebnis: 37 

# Wie hoch ist der prozentuale Anteil der Vereinsmitglieder an allen 150 #30u30-Alumni?
100*(37/150) # gerundet: 24,67 %
```

Art des Engagements: Wie viele Kanten stellen eine leitende Funktion, wie viele eine reine Mitgliedschaft dar? 

```{r Mitgliedschaft in Initiativen - Bis Aufnahme - Art des Engagements}

# Bei wie vielen Edges handelt es sich um eine leitende Funktion?
verein_bisaufn_leadership <- E(initiative_preentry1)$relationship == 13
sum(verein_bisaufn_leadership, na.rm=T) # 55

# Bei wie vielen Edges handelt es sich um eine "bloße" Mitgliedschaft?
verein_bisaufn_membership <- E(initiative_preentry1)$relationship == 11
sum(verein_bisaufn_membership, na.rm=T) # Ergebnis: 97 
```
Welche Initiativen verbinden mehrere #30u30-Mitglieder bis zum jew. Jahr deren Aufnahme - also schon vor #30u30?

```{r Mitgliedschaft in Initiativen - Bis Aufnahme - Indegree-Verteilung}
ind_initiative_pe <- degree(initiative_preentry2, mode="in")
ind_initiative_pe
sort(ind_initiative_pe)
```

Generiert den Plot der Initiativen-Mitglieder vor Aufnahme bei #30u30:
```{r Mitgliedschaft in Initiativen - Bis Aufnahme - Plot indegree-skaliert}
plot(initiative_preentry2,
     layout=layout_with_fr,
     edge.curved=curve_multiple(initiative_preentry2),
     edge.curved=.8,
     vertex.size=ind_initiative_pe*.8,
     vertex.label.cex=.3,
     vertex.label.family="sans",
     vertex.label.color="black",
     edge.arrow.size=.000001,
     edge.arrow.color="white",
     asp=0)
```

### Teilnetzwerk studentische Initiativen

Um zu betrachten, ob die Mitglieder der Initiativen darüber hinaus - bspw. mit Blick auf Arbeitgeber - Überschneidungen aufweisen, wird ein Graph erstellt, in dem die Initiativen, deren Mitglieder und deren Verbindungen enthalten sind. Genauer gesagt werden für die jeweiligen Initiativen Ego-Netzwerke zweiten Grades erstellt, sodass die direkt damit verbundenen Personen, aber auch die Kanten der Personen inkludiert sind.

```{r Mitgliedschaft in Initiativen: Ego-Netzwerke der Initiativen erstellen}
prsh_hannover <- subgraph <- make_ego_graph(members, order = 2,  c("PRSH (Hannover)"))
goldene20 <- subgraph <- make_ego_graph(members, order = 2,  c("Goldene Zwanziger (Jena)"))
kommunikos <- subgraph <- make_ego_graph(members, order = 2,  c("Kommunikos (Osnabrück)"))
kommon <- subgraph <- make_ego_graph(members, order = 2,  c("KOMMON (Darmstadt)"))
kommunity <- subgraph <- make_ego_graph(members, order = 2,  c("KOMMunity"))
lprs_ev <- subgraph <- make_ego_graph(members, order = 2,  c("LPRS (Leipzig)"))
campusrel <- subgraph <- make_ego_graph(members, order = 2,  c("Campus Relations (Muenster)"))
kommoguntia <- subgraph <- make_ego_graph(members, order = 2,  c("Kommoguntia (Mainz)"))
priho_ev <- subgraph <- make_ego_graph(members, order = 2,  c("PRIHO (Hohenheim)"))
marketing_tp <- subgraph <- make_ego_graph(members, order = 2,  c("Marketing zwischen Theorie und Praxis"))

# Von Gesamtnetzwerk der #30u30-Mitglieder doppelt subtrahieren (= addieren):
studentinitiatives <- members - (members - prsh_hannover[[1]] - goldene20[[1]] - kommunikos[[1]] - kommon[[1]] - kommunity[[1]] - lprs_ev[[1]] - campusrel[[1]] - kommoguntia[[1]] - priho_ev[[1]] - marketing_tp[[1]]) 
studentinitiatives
```

Bevor geplottet wird werden isolierte Knoten sowie solche, die in dem zu erstellenden Netzwerk nicht relevant sind, gelöscht, sowie Kanten vereinfacht.

```{r Mitgliedschaft in Initiativen: Selektion von Knoten und Kantenvereinfachung}
studentinitiatives <- delete_vertices (studentinitiatives, V(studentinitiatives)[category == 5])
studentinitiatives <- delete_vertices (studentinitiatives, V(studentinitiatives)[category == 7])

studentinitiatives <- delete_vertices (studentinitiatives, V(studentinitiatives)[degree(studentinitiatives, mode="all")=="0"])

studentinitiatives <- simplify(studentinitiatives, remove.multiple = TRUE)
```

Zur Unterscheidung werden die Knoten je nach Jahrgangszugehörigkeit verschieden eingefärbt.

```{r Mitgliedschaft in Initiativen: Visualisierungsanpassung}
V(studentinitiatives)[V(studentinitiatives)$entry == 2017]$color <- "firebrick1" 
V(studentinitiatives)[V(studentinitiatives)$entry == 2018]$color <- "firebrick2"
V(studentinitiatives)[V(studentinitiatives)$entry == 2019]$color <- "firebrick3" 
V(studentinitiatives)[V(studentinitiatives)$entry == 2020]$color <- "firebrick"
V(studentinitiatives)[V(studentinitiatives)$entry == 2021]$color <- "firebrick4" 
```

Nun wird geplottet. 
```{r Plot der student. Initiativen, deren Mitglieder und deren Verbindungen}
plot(studentinitiatives,
     layout=layout_with_fr,
     edge.curved=curve_multiple(studentinitiatives), # verhindert, dass sich Kanten überlagern
     vertex.frame.color="white",
     edge.arrow.size=.0001,
     edge.arrow.color="white",
     vertex.size=1.5,
     #vertex.size=ind_studinit*.4,
     edge.color="grey",
     edge.curved=.2,
     vertex.label.degree=0, 
     vertex.label.cex=.15,
     vertex.label.family="sans",
     vertex.label.font=2,
     vertex.label.color="black",
     asp=0,
     main="Netzwerk der PR-Initiativen und deren Mitglieder 
     in den #30u30-Jahrgängen 2017 bis 2021",
     sub="Inkl. der Verbindungen der einzelnen Mitglieder")
```
Welche Organisationen haben mehrere eingehende Verbindungen? Was verbindet die Initiativenmitglieder über die Mitgliedschaft hinaus, wo treffen sich ihre Pfade?
```{r Mitgliedschaft in Initiativen: Indegree-Verteilung}
ind_studinit <- degree(studentinitiatives, mode="in")
sort(ind_studinit, decreasing = T)
```

## Stipendiennetzwerk

Wie bereits zuvor berechnet, wurde die Mehrheit der 150 Talente nicht durch ein Stipendium gefördert. Nun werden jedoch diejenigen, die eine Förderung erhalten haben, gesondert in den Blick genommen, um mögliche Gemeinsamkeiten zu erkennen.

```{r Erstellen des Teilnetzwerks der Stipendiat:innen}
members

# Selektieren der Kanten, bei denen es sich um Förderung durch ein Stipendium handelt
scholarship <- subgraph.edges(members, E(members)[relationship == 14])
scholarship

# Doppelte Kanten werden nur einfach dargestellt
scholarship1 <- simplify(scholarship, remove.multiple = TRUE)
```

Knoten werden je nach Jahrgangs-Zugehörigkeit zur Unterscheidung verschieden eingefärbt.

```{r Teilnetzwerk Stipendium: Visualisierungsanpassung}
V(scholarship1)[V(scholarship1)$entry == 2017]$color <- "firebrick1"
V(scholarship1)[V(scholarship1)$entry == 2018]$color <- "firebrick2" 
V(scholarship1)[V(scholarship1)$entry == 2019]$color <- "firebrick3" 
V(scholarship1)[V(scholarship1)$entry == 2020]$color <- "firebrick"
V(scholarship1)[V(scholarship1)$entry == 2021]$color <- "firebrick4" 
```

Stipendiumsnetzwerk wird geplottet:

```{r Teilnetzwerk Stipendium: Plot}
#Plot - ohne Degree-Anpassung der Knotengröße
plot(scholarship1,
     layout=layout_with_kk,
     edge.curved=curve_multiple(scholarship1),
     vertex.size=5,
     vertex.frame.color="white",
     edge.color="grey",
     edge.curved=.2,
     edge.arrow.size=.00001,
     edge.arrow.color="white",
     vertex.size=5,
     vertex.label.degree=0, 
     vertex.label.cex=.2,
     vertex.label.family="sans",
     vertex.label.font=2,
     vertex.label.color="black", 
     asp=0,
     main="Netzwerk der Stipendiat:innen 
     in den #30u30-Jahrgängen 2017 bis 2021")
```
Welche Stipendien kommen mehrfach vor und verbinden mehrere Talente?

```{r Teilnetzwerk Stipendium: Indegree-Verteilung}
ind_scholarship <- degree(scholarship1, mode="in")
sort(ind_scholarship, decreasing = T)
```

Welche Personen wurden durch mehrere Stipendien gefördert?

```{r Teilnetzwerk Stipendium: Outdegree-Verteilung}
outd_scholarship <- degree(scholarship1, V(scholarship1), mode="out")
outd_scholarship
sort(outd_scholarship, decreasing = T)
```

Zwischenfazit: Die am häufigsten vertretenen Stipendien sind das Stipendium des Dt. Akadem. Auslandsdiensts und das Deutschlandstipendium, gefolgt von der Studienstiftung d. dt. Volkes und dem Promos-Stipendium (je degree=4). Die Person mit den meisten Stipendien ist Gertrud Kohl, sie wurde von drei Begabtenförderungswerken unterstützt. 10 der Talente wurden durch mindestens zwei Stipendien gefördert. Während es viele isolierte Personen gibt, die durch das Stipendium keine Überschneidung mit anderen Talenten haben, sind die Stipendiat*innen von Promos-, DAAD,Deutschland-, E-fellows- und Konrad Adenauer-Stipendium miteinander vernetzt, die genannten Insitutionen fungieren hier als Brücken.

In einem Plot werden nun zentrale Knoten entspr. ihres Indegree-Werts größer dargestellt:

```{r Teilnetzwerk Stipendium: Plot mit Indegree-Skalierung}
ind_scholarship <- degree(scholarship1, V(scholarship1), mode="in")

# Generiert den Plot
plot(scholarship1,
     layout=layout_with_kk,
     edge.curved=curve_multiple(scholarship1),
     vertex.size=ind_scholarship*1.2, # Knotengröße entspr. des indegree-Werts skaliert
     vertex.frame.color="white",
     edge.color="grey",
     edge.curved=.2,
     edge.arrow.size=.00001,
     edge.arrow.color="white",
     vertex.size=7,
     vertex.label.degree=0, 
     vertex.label.cex=.2,
     vertex.label.family="sans",
     vertex.label.font=2,
     vertex.label.color="black", 
     asp=0,
     main="Netzwerk der Stipendiat:innen 
     in den #30u30-Jahrgängen 2017 bis 2021",
     sub="Knotengröße entspr. Indegree-Wert und mit Faktor 1,2 skaliert")
```

```{r Teilnetzwerk Stipendium: Clusteranalyse}
scholarship1
gc_schol <- cluster_walktrap(scholarship1)

# Berechne Modularität
modularity(gc_schol) # 0,8104914
membership(gc_schol)
par(mfrow=c(1,1), mar=c(0,0,1,2))

#Visualisierung der Cluster
plot(gc_schol, 
     scholarship1, 
     main="Clusteranalyse der Stipendiat:innen 
     in den #30u30-Jahrgängen 2017 - 2021",
     edge.curved=curve_multiple(scholarship1),
     edge.arrow.size=.000001, 
     edge.color="grey",
     edge.curved=.2,
     edge.arrow.color="white",
     vertex.size=5,
     vertex.label.degree=0, 
     vertex.label.cex=.15,
     vertex.label.family="sans",
     vertex.label.font=2,
     vertex.label.color="black")

```
```{r Komponenten im Stipendiat:innen-Netzwerk}
components(scholarship1)
# 16 Komponenten, $csize [1] 24  3  2  2  2  3  3  2  2  2  3  2  2  2  5  2
```

Um wie viele und welche Art von Organisation handelt es sich bei den Stipendiengebern?

```{r Teilnetzwerk Stipendium: Anzahl & Zusammensetzung der Stipendiengeber}
stipendiumsgeber <- V(scholarship1)$type == 2
sum(stipendiumsgeber, na.rm=T) # Zählt, wie viele Förderungsinstitutionen das Netzwerk enthält - insg. 26

# Wie viele davon sind Begabtenförderungswerke?
stiftungen <- V(scholarship1)$category == 7
sum(stiftungen, na.rm=T) # 21

# Wie viele sind Hochschulen?
hs_foerderung <- V(scholarship1)$category == 5
sum(hs_foerderung, na.rm = T) # 4 Hochschulen 

# Wie viele davon sind Forschungsinstitute?
inst <- V(scholarship1)$category == 10
sum(inst, na.rm=T) # 1

# Wie viele Stipendiat*innen sind im Netzwerk?
stips <- V(scholarship1)$type == 1
sum(stips, na.rm = T) # 35 Personen
```

Zwischenfazit: Von den 26 stipendienvergebenden Institutionen handelt es sich bei 21 - der Mehrheit - um klassische Begabtenförderungswerke, bei 4 um Hochschulen, die Leistungen mit Stipendien honorieren.

Wie dicht ist das Netzwerk?

```{r Teilnetzwerk Stipendium: Dichte}
edge_density(scholarship1) # 0,01256831
```
### Vergleich: Stipendiat:innen und Nicht-Stipendiat:innen

Um zu betrachten, ob sich die Netzwerke bei Stipendiat:innen und Nicht-Stipendiat:innen unterscheiden, werden entsprechende Teilnetzwerke erstellt, die alle Verbindungen der (Nicht-)Stipendiat:innen beinhalten.

#### Teilnetzwerk der Stipendiat:innen

```{r Teilnetzwerk der Stipendiat:innen und deren Verbindungen}
members

scholars <- delete.vertices(members, V(members)[(type == 1) & (scholarship != 1)]) # selektiert nur die #30u30-Mitglieder, die durch Stipendien gefördert wurden

scholars <- delete.vertices(scholars, V(scholars)[degree(scholars, V(scholars), mode = "all") == 0]) # löscht isolierte Knoten

scholars_simple <- simplify(scholars, remove.multiple = TRUE) # löscht mehrfache Kanten / Vereinfachung
```
Welche Organisationen verbinden mehrere der Stipendiat:innen?

```{r Teilnetzwerk der Stipendiat:innen: Indegree-Verteilung}
ind_scholars <- degree(scholars_simple, mode="in")
sort(ind_scholars, decreasing = T)
```

Generiert den Plot des Teilnetzwerks der Stipendiat:innen mit all deren organisat. Verbindungen.

```{r Teilnetzwerk der Stipendiat:innen: Plots}
plot(scholars_simple,
     layout=layout_with_fr,
     edge.curved=curve_multiple(scholars_simple),
     vertex.frame.color="white",
     edge.color="grey",
     edge.curved=.2,
     edge.arrow.size=.00001,
     edge.arrow.color="white",
     vertex.size=2,
     vertex.label.degree=0, 
     vertex.label.cex=.2,
     vertex.label.family="sans",
     vertex.label.font=2,
     vertex.label.color="black", 
     asp=0,
     main="Netzwerk der Stipendiat:innen 
     in den #30u30-Jahrgängen 2017 - 2021")

# Plot - mit Indegreeskalierung
plot(scholars_simple,
     layout=layout_with_fr,
     edge.curved=curve_multiple(scholars_simple),
     vertex.frame.color="white",
     edge.color="grey",
     edge.curved=.2,
     edge.arrow.size=.00001,
     edge.arrow.color="white",
     vertex.size=ind_scholars*1.2,
     vertex.label.degree=0, 
     vertex.label.cex=.2,
     vertex.label.family="sans",
     vertex.label.font=2,
     vertex.label.color="black", 
     asp=0,
     main="Netzwerk der Stipendiat:innen 
     in den #30u30-Jahrgängen 2017 - 2021",
     sub="Knotengröße entspr. Indegree-Wert und mit Faktor 1,2 skaliert")

# Mit Indegreeskalierung - OHNE LABEL
plot(scholars_simple,
     layout=layout_with_fr,
     edge.curved=curve_multiple(scholars_simple),
     vertex.frame.color="white",
     edge.color="grey",
     edge.curved=.2,
     edge.arrow.size=.00001,
     edge.arrow.color="white",
     vertex.size=ind_scholars*1.2,
     vertex.label=NA, 
     main="Netzwerk der Stipendiat:innen 
     in den #30u30-Jahrgängen 2017 - 2021",
     sub="Knotengröße entspr. Indegree-Wert und mit Faktor 1,2 skaliert",
     asp=0)
```

Betweenness-Werte im Stipendiat:innen-Netzwerk: Wer sind die Broker?

```{r Teilnetzwerk der Stipendiat:innen: Betweenness}
betweenness(scholars_simple)
centr_betw(scholars_simple, directed=TRUE)
betw_scholars <- betweenness(scholars_simple)
sort(betw_scholars, decreasing = T)

V(scholars_simple)[mentor==1]$color <- "firebrick1" # färbt #30u30 rot

# Wir können die Ausgabe von betweenness()  einer Variablen im Netz zuordnen und die Knoten entsprechend dimensionieren.
V(scholars_simple)$betweenness <- betweenness(scholars_simple)

plot(scholars_simple, 
     vertex.label.cex = .6, 
     vertex.label.color = "black", 
     vertex.size = V(scholars_simple)$betweenness) # Größe angepasst an betweenness

# Da die  Betweenness-Zentralität sehr groß sein kann, wird sie im folgenden normalisiert, indem man sie durch das Maximum teilt und mit einem Skalar multipliziert, bevor man sie plottet. So erkennen wir die Broker besser.
plot(scholars_simple,
     layout=layout_with_fr,
     edge.curved=curve_multiple(scholars_simple),
     vertex.label=ifelse(betw_scholars>7000, V(scholars_simple)$name, NA), # Labels nur bei Knoten mit Betweenness > 7000 anzeigen
     vertex.label.cex = .2, 
     vertex.label.frame=NA,
     vertex.label.color = "black", 
     vertex.label.family="sans",
     edge.arrow.size=.00001,
     edge.arrow.color="white",
     asp=0,
     vertex.size = V(scholars_simple)$betweenness/max(V(scholars_simple)$betweenness) * 10,
     main="Broker im Teilnetzwerk der Stipendiat:innen 
     in den #30u30-Jahrgängen 2017 - 2021", 
     sub="Knotengröße entspr. Betweennness-Wert und mit Faktor 10 skaliert")
```
#### Zum Vergleich: Nicht-Stipendiat:innen
```{r Teilnetzwerk der Nicht-Stipendiat:innen}
members
non_scholars <- delete.vertices(members, V(members)[(type == 1) & (scholarship == 1)]) # löscht die Stipendiat*innen aus dem Mitgliedernetzwerk

non_scholars <- delete.vertices(non_scholars, V(non_scholars)[degree(non_scholars, V(non_scholars), mode = "all") == 0]) # löscht isolierte Knoten

nonscholars_simple <- simplify(non_scholars, remove.multiple = TRUE) # Kantenvereinfachung
```

Welche Organisationen verbinden mehrere Nicht-Stipendiat:innen?

```{r Teilnetzwerk der Nicht-Stipendiat:innen: Indegree-Verteilung}
ind_nonschol <- degree(nonscholars_simple, mode="in")
sort(ind_nonschol, decreasing = T)
```
Generiert den Plot des Netzwerks der Nicht-Stipendiat:innen in den 5 #30u03-Jahrgängen:

```{r Teilnetzwerk der Nicht-Stipendiat:innen: Plot}
plot(nonscholars_simple,
     layout=layout_with_fr,
     edge.curved=curve_multiple(nonscholars_simple),
     vertex.frame.color="white",
     edge.color="grey",
     edge.curved=.2,
     edge.arrow.size=.00001,
     edge.arrow.color="white",
     vertex.size=ind_nonschol*1.2,
     vertex.label=NA, 
     asp=0,
     main="Netzwerk der Nicht-Stipendiat:innen
     in den #30u30-Jahrgängen 2017 - 2021",
     sub="Knotengröße entspr. Indegree-Wert und mit Faktor 1,2 skaliert")
```
Betweenness-Werte im Netzwerk der Nicht-Stipendiat:innen: Wer sind hier die Broker?

```{r Teilnetzwerk der Nicht-Stipendiat:innen: Betweenness}
betweenness(nonscholars_simple)
centr_betw(nonscholars_simple, directed=TRUE)
betw_nonscholars <- betweenness(nonscholars_simple)
sort(betw_nonscholars, decreasing = T)

V(nonscholars_simple)[mentor==1]$color <- "firebrick1" # färbt #30u30 rot

# Wir können die Ausgabe von betweenness()  einer Variablen im Netz zuordnen und die Knoten entsprechend dimensionieren.
V(nonscholars_simple)$betweenness <- betweenness(nonscholars_simple)

plot(nonscholars_simple, 
     vertex.label.cex = .6, 
     vertex.label.color = "black", 
     vertex.size = V(nonscholars_simple)$betweenness) # Größe angepasst an betweenness

# Da die  Betweenness-Zentralität sehr groß sein kann, wird sie im folgenden normalisiert, indem man sie durch das Maximum teilt und mit einem Skalar multipliziert, bevor man sie plottet. So erkennen wir die Broker besser.
plot(nonscholars_simple,
     layout=layout_with_fr,
     edge.curved=curve_multiple(nonscholars_simple),
     vertex.label=ifelse(betw_nonscholars>30000,V(nonscholars_simple)$name, NA),
     vertex.label.cex = .2, 
     vertex.frame.color=NA,
     vertex.label.color = "black", 
     vertex.label.family="sans",
     edge.arrow.size=.00001,
     edge.arrow.color="white",
     asp=0,
     vertex.size = V(nonscholars_simple)$betweenness/max(V(nonscholars_simple)$betweenness) * 5,
     main="Broker im Netzwerk der Nicht-Stipendiat:innen 
     in den #30u30-Jahrgängen 2017 - 2021",
     sub="Knotengröße entspr. Betweenness-Wert und mit Faktor 5 skaliert")
```
Wie viele Cluster / Komponenten umfassen beide Teilnetzwerke und wie hoch ist die Modularität jeweils?

```{r Stipendiat:innen & Nicht-Stipendiat:innen: Clusteranalyse}
# Stipendiat*innen
scholars_simple
gc_scholars <- cluster_walktrap(scholars_simple)

# Berechne Modularität
modularity(gc_scholars) # 0,7693319
# Zeigt Komponenten an
components(scholars_simple) # 2 Komponenten (317 Knoten, 8 Knoten)

membership(gc_scholars)

# Nicht-Stipendiat*innen
nonscholars_simple
gc_nonschol <- cluster_walktrap(nonscholars_simple)

# Berechne Modularität
modularity(gc_nonschol) # 0,7418976

# Zeige Komponenten
components(nonscholars_simple) # 7 Komponenten (Größe: 738, 10, 6, 5, 4, 2, 9 Knoten)

membership(gc_nonschol)
```
Zum Vergleich werden jeweils auch Dichte und Pfaddistanz berechnet.

```{r Stipendiat:innen und Nicht-Stipendiat:innen: Dichte}
# Dichte
100*edge_density(nonscholars_simple) # 0,15 %
100*edge_density(scholars_simple) #0,35 %
```

```{r Stipendiat:innen und Nicht-Stipendiat:innen: Pfaddistanz & Diameter}
# Mittlere Pfaddistanz:
mean_distance(scholars_simple, directed=FALSE) # 5,893543
mean_distance(nonscholars_simple, directed=FALSE) # 6,563518

# Durchmesser / Diameter
diameter(scholars_simple, directed=FALSE) # 12 Schritte
diameter(nonscholars_simple, directed=FALSE) # 14 Schritte
```

## Mentoring und Mentor:innen

### Mentor:innen
Final werden die Mentor:innen der #30u30-Mitglieder der Jahrgänge 2017 bis 2021 in den Blick genommen.

Zunächst wird nur ein Netzwerk der Mentor:innen und deren Verbindungen erstellt.

```{r Teilnetzwerk der Mentor:innen}
g

mentors <- delete_vertices (g, V(g)[mentor == 1]) #Löscht die #30u30, sodass nur die Mentor*innen und ihre Verbindungen angezeigt werden
mentors

mentors1 <- delete_vertices(mentors, V(mentors)[degree(mentors, mode="all")==0]) # löscht isolierte Knoten
```

```{r Teilnetzwerk Mentor:innen: Plot}
ind_mentors <- degree(mentors1, mode="in")

# Generiert den Plot des Netzwerks der Mentor:innen, Knotengröße entspr. dem Indegree-Wert skaliert:
plot(mentors1,
     layout=layout_with_fr,
     edge.curved=curve_multiple(mentors1), # verhindert, dass sich Kanten überlagern
     vertex.size=ind_mentors*1.2,
     vertex.frame.color="white",
     edge.color="grey",
     edge.curved=.2,
     vertex.label.degree=0, 
     vertex.label.cex=.15,
     vertex.label.family="sans",
     vertex.label.font=2,
     vertex.label.color="black",
     main="Netzwerk der Mentor*innen der #30u30-Jahrgänge 2017 - 2021",
     sub="Knotengröße entspr. Indegree-Wert und mit Faktor 1,2 skaliert",
     asp=0)
```

Da die Visualisierung nicht alles auf den ersten Blick zeigt, werden die Indegree-Werte noch einmal abgerufen, um zu prüfen, ob es Organisationen gibt, mit denen mehrere Mentor:innen in Verbindung stehen.

```{r Indegree-Verteilung: Zentrale Organisationen}
sort(ind_mentors, decreasing = T)
```
Wie dicht ist das Netzwerk der Mentor:innen?

```{r Teilnetzwerk Mentor:innen: Dichte}
edge_density(mentors1) # 0,002905303 (= 0,29 %)
```
Bei welchen Branchenverbänden waren die Mentor*innen tätig?

```{r Teilnetzwerk Mentor:innen: Branchenverbände}
mentors
mentor_mem <- subgraph.edges(mentors, E(mentors)[relationship == 11]) # selektiert Kanten, bei denen es sich um Mitgliedschaft handelt
mentor_mem2 <- add.edges(mentor_mem, E(mentor_mem)[relationship == 13]) # selektiert Kanten, bei denen es sich um leitende Funktionen in Vereinen handelt

# Visualisierung
V(mentor_mem2)[V(mentor_mem2)$mentor == 2]$color <- "orange" # Mentor*innen werden orange gefärbt
V(mentor_mem2)[V(mentor_mem2)$type == 1]$shape <- "circle" # Personen werden als Kreis angezeigt

# Plot
plot(mentor_mem2,
     layout=layout_with_fr,
     edge.curved=curve_multiple(mentor_mem2), # verhindert, dass sich Kanten überlagern
     vertex.size=5,
     vertex.frame.color="white",
     edge.curved=.2,
     vertex.label.degree=0, 
     vertex.label.cex=.3,
     vertex.label.family="sans",
     vertex.label.font=2,
     vertex.label.color="black",
     main="Netzwerk der Mentor:innen der #30u30-Jahrgänge 2017 - 2021",
     sub="Mitgliedschaft in Branchenverbänden / Parteien",
     asp=0)
```

### Mentor:innen und Mentees

Nachdem die Mentor:innen zunächst einzeln betrachtet wurden, wird nun das Netzwerk der Mentor:innen und Mentees erstellt und analysiert.

```{r Teilnetzwerk Mentorship}
g 

mentoring <- subgraph.edges(g, E(g)[relationship == 17]) # Selektieren der Mentoring-Beziehungen
mentoring

# Mentor*innen und Mentees unterschiedlich einfärben
V(mentoring)[V(mentoring)$mentor == 1]$color <- "firebrick1"
V(mentoring)[V(mentoring)$mentor == 2]$color <- "orange2"

mentoring

ind_mentoring <- degree(mentoring, mode="in")

# Generiert den Plot
plot(mentoring,
     layout=layout_with_fr,
     edge.curved=curve_multiple(mentoring), # verhindert, dass sich Kanten überlagern
     edge.curved=.2,
     vertex.size=ind_mentoring*1.2,
     vertex.label.cex=.15,
     vertex.label.family="sans",
     vertex.label.font=2,
     vertex.label.color="black",
     vertex.frame.color=NA,
     main="Mentoring-Netzwerk der #30u30-Jahrgänge 2017 bis 2021",
     sub="rot = #30u30-Mitglied, orange = Mentor*innen,
     Knotengröße entspr. Indegree-Wert und mit Faktor 1,2 skaliert",
     asp=0)
```
Wie viele Komponenten, Cluster, Communities umfasst das Mentoring-Netzwerk?
```{r Teilnetzwerk Mentorship - Cluster}
mentoring
gc_mentoring <- cluster_walktrap(mentoring)

# Berechne Modularität
modularity(gc_mentoring) # 0,9772102

membership(gc_mentoring)
par(mfrow=c(1,1), mar=c(0,0,1,2))

# Liefert Komponenten (Anzahl und Bestandteile)
components(mentoring) # Anzahl: 66
```
Gibt es Mentor:innen, die mit mehreren #30u30-Mitgliedern in Verbindung stehen? Welche Mentees haben besonders viele Mentor:innen?

```{r Teilnetzwerk Mentoring: Indegree- und Outdegree-Verteilung}
# Welche Mentor*innen kommen am häufigsten vor?
ind_mentoring
sort(ind_mentoring, decreasing = T) 
  # Eugenia Lagemann: indegree = 3, Andreas Winiarski: indegree = 2

# Wie viele Mentor:innen haben die einzelnen #30u30-Mitglieder?
outd_mentoring <- degree(mentoring, mode="out")
sort(outd_mentoring, decreasing = T)

# Person mit den meisten Mentor:innen?
V(mentoring)$name[degree(mentoring)==max(degree(mentoring))]
    # Judith Götter, outdegree = 11

```

Andreas Winiarski ist der männliche Mentor, der am meisten Talente verbindet. Nun wird ein Teilnetzwerk von Winiarski und seinen Mentees generiert, um zu betrachten, wo sich deren Wege überschneiden.

```{r Teilnetzwerk Andreas Winiarski und Mentees}
# Erstellen des Teilnetzwerks
# Einlesen der Edgelist
edges <- read.csv("https://raw.githubusercontent.com/hb062/BA_30u30/main/Teilnetzwerk%3A%20Mentoring-Netzwerk%20A.%20Winiarski/el_winiarski_mentees", header=T, as.is=T, sep = ",", fileEncoding="UTF-8")
head(edges) # zeigt die ersten Zeilen der Edgelist an

# Einlesen der Nodelist
nl <- read.csv("https://raw.githubusercontent.com/hb062/BA_30u30/main/Teilnetzwerk%3A%20Mentoring-Netzwerk%20A.%20Winiarski/nl_winiarski_mentees", header=T, as.is=T, sep = ",", fileEncoding="UTF-8")

head(nl) # zeigt die ersten Zeilen der Nodelist an

# Verknüpfen von Edge- und Nodelist  zu einer Matrix
edgematrix <-as.matrix(edges)
mentor_winiarski <- graph_from_data_frame(d=edgematrix, vertices=nl, directed=TRUE)

mentor_winiarski
mentor_winiarski <- simplify(mentor_winiarski, remove.multiple = TRUE) # Kantenvereinfachung

# Visualisierungsanpassung

V(mentor_winiarski)[V(mentor_winiarski)$mentor == 2]$color <- "gold1" # mentors gelb färben
V(mentor_winiarski)[V(mentor_winiarski)$name == "Andreas Winiarski"]$color <- "firebrick1" # Winiarski wird rot gefärbt
V(mentor_winiarski)[V(mentor_winiarski)$mentor == 1]$color <- "orange1" # Mentees orange färben
V(mentor_winiarski)[V(mentor_winiarski)$type == 2]$shape <- "square" # Organisationen werden als Quadrat angezeigt
V(mentor_winiarski)[V(mentor_winiarski)$category == 1]$color <- "dodgerblue4" # Unternehmen dunkelblau
V(mentor_winiarski)[V(mentor_winiarski)$category == 2]$color <- "cadetblue3" # Agenturen hellblau
V(mentor_winiarski)[V(mentor_winiarski)$category == 5]$color <- "seagreen" #Hochschulen grün
V(mentor_winiarski)[V(mentor_winiarski)$category == 6]$color <- "violetred" # Vereine pink
V(mentor_winiarski)[V(mentor_winiarski)$category == 7]$color <- "pink1" #Stipendien rosa

par(mar=c(1,1,1,1))

ind_winiarski <- degree(mentor_winiarski, mode="in")

# Visualisierung des neu generierten Netzwerks (indegree-skaliert)
plot(mentor_winiarski,
     vertex.shape="circle",
     layout=layout_with_kk,
     edge.curved=curve_multiple(mentor_winiarski), # verhindert, dass sich Kanten überlagern
     edge.arrow.size=.000001,
     edge.arrow.colour="white",
     vertex.label.size=.2, 
     vertex.size=ind_winiarski*2,
     vertex.frame.color=NA,
     vertex.label.color="black",
     vertex.label.cex=.3,
     asp=0,
     vertex.label.family="sans",
     main="Mentoring-Netzwerk von Andreas Winiarski",
     sub="Verflechtungen mit den Mentees, 
     Knotengröße entspr. Indegree-Wert und mit Faktor 1,2 skaliert")

# wie hoch ist die Dichte?
edge_density(mentor_winiarski) # 0.04558405
```
Wer unter den #30u30 heraussticht ist Judith Götter, da sie die meisten Mentoring-Beziehungen besitzt. In einem Ego-Netzwerk wird sie daher separat betrachtet, um die Orte der Überschneidungen mit ihren Mentor:innen zu erkennen. Dabei wird deutlich: Diese zentrieren sich um die Lufthansa und die Agentur Montua Partner.

```{r Mentoring - Ego-Netzwerk Judith Götter und Mentor:innen}
g
V(g)[V(g)$name == "Judith Götter"]$color <- "firebrick" # Götter wird rot hervorgehoben
V(g)[V(g)$member == 2]$color <- "orange" # Mentor*innen orange
g

# selektiert aus dem Gesamtnetzwerk alle Knoten, die mit Judith Götter über einen Schritt verbunden sind.
goetter <- make_ego_graph(g, order = 1, nodes = V(g)$name == "Judith Götter", mode ="all")
goetter

# Generiert den Plot des Ego-Netzwerks "goetter"
plot(goetter[[1]], 
     main="Ego-Netzwerk Judith Götter",
     vertex.frame.color="white",
     edge.arrow.size=.000001,
     edge.arrow.color="white",
     edge.curved=.2,
     vertex.size=10,
     vertex.label.cex=.3,
     vertex.label.family="sans",
     vertex.label.color="black",
     sub="nur direkte Beziehungen des ersten Grads",
     asp=0)
```
