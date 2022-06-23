# Datensatz zur Netzwerkanalyse von #30u30
## Codebuch 
Stand 23.06.2022 \
erstellt von Hannah Bauer (hb062@hdm-stuttgart.de)

## Inhalt des Datensatzes
- Edges.csv (Edgelist) https://github.com/hb062/BA_30u30/blob/main/el.csv
- Nodes.csv (Nodelist) https://github.com/hb062/BA_30u30/blob/main/nl.csv
- Codebuch.md (Codierung der Datensätze)

## Umgang mit fehlenden Werten 
Fehlen Werte oder falls sich Werte nur auf einen bestimmten Typ an Nodes bezieht, werden diese mit dem Wert "99" codiert. Dies ist beispielsweise der Fall bei Organisationen, für die beim Vertex-Attribut "sex" logischerweise kein Wert vergeben wird. Gleiches gilt für Daten der Edge-List, etwa wenn zu einer Beziehung kein Jahr bekannt ist. Das vorherige Vorgehen, bei dem das zugehörige Feld in der Edge- oder Node-List frei gelassen oder mit NA versehen wurde, hatte zu Problemen beim Plotten der Daten geführt, konkret bei der Arbeit mit und Selektion nach Vertex-/Node-Attributen. Eine Umcodierung auf "99" konnte zur Beseitigung des Problems beitragen.
Fehlen Werte oder beziehen sich Attribute nur auf einen bestimmten Typ an Nodes des two-mode-Networks, wird entsprechend mit dem Wert "99" codiert.

# Edgelist und Edge-Attribute
Grundregel: Die Edgelist enthält pro Spalte immer nur einen Wert. Bis auf die ID und den Namen ist dieser numerisch codiert (als Zahl).

- from: Die Werte in der Spalte “from” definieren den Ausgangspunkt einer Beziehung. 
Da es sich um ein gerichtetes Netzwerk handelt, geht die Beziehung von einem Sender aus, bspw. einer Person, die bei einem Unternehmen gearbeitet hat. Der eingetragene Wert entspricht einer ID in der Nodelist und enthält keine Sonderzeichen, sondern nur ein Wort. Zur Unterscheidung ähnlicher Namen wird dieses ggf. mit einer Zahl oder einem Unterstrich ergänzt.

- to: Die Werte in der Spalte “to” definieren den Empfänger in ungerichteten Netzwerken.  Da es sich um ein gerichtetes Netzwerk handelt, sind beispielsweise Arbeitsstellen die Empfänger einer Beziehung, die Verbindung läuft von Arbeitnehmer:innen hin zur Arbeitsstelle.
 Der eingetragene Wert entspricht einer ID in der Nodelist und enthält keine Sonderzeichen, sondern nur ein Wort, zur Unterscheidung ähnlicher Namen wird dieses ggf. mit einer Zahl oder einem Unterstrich ergänzt.
 
 - year: Das Edge-Attribut „year“ definiert das Jahr, in dem die jeweilige Beziehung bestand, beziehungsweise wann eine Person bei einer Organisation beschäftigt war oder an einer Hochschule studiert hat. Die Zeiträume wurden auf Basis der Angaben in den jeweiligen LinkedIn-Profilen der Personen erfasst. Bei mehrjährigen Verbindungen wurde entsprechend für jedes separate Jahr eine Beziehung angelegt. Die Jahreszahlen reichen von 2006 bis 2022 und umfassen damit den Berufs- und Studienweg der Akteur:innen vor ihrer Aufnahme in das #30u30-Netzwerk bis hin zum aktuellen Zeitpunkt zur Datenerhebung im Rahmen dieser Arbeit (März 2022). Bei Verbindungen, bei denen das Jahr nicht bekannt war, wurde statt einer Jahreszahl der Wert 99 vergeben. 

- relationship: Das Edge-Attribut „relationship“ definiert die Art der Beziehung zwischen Knoten, da es sich um ein multiplexes Netzwerk mit verschiedenen Beziehungsarten handelt.  Hierbei wurden vor allem die Beziehungen zwischen Knoten des Typ 1 und Typ 2 (Personen und Organisationen) definiert, da wenig Daten zur Beziehungsqualität zwischen den einzelnen natürlichen Personen vorlagen bzw. mit der gewählten Methode nicht erhoben werden konnten. Primär wird daher die Verbindung zwischen natürlicher Person und der Organisation näher definiert, darüber hinaus werden Verbindungen zwischen den #30u30 und ihren Mentorinnen / Mentoren codiert. \
1 = Ausbildung \
2 = Bachelorstudium \
3 = Masterstudium \
4 = Auslandssemester *(Gesamte Bachelor-/Masterstudiengänge, die im Ausland absolviert wurden, wurden nicht als Auslandssemester, sondern regulär als Bachelor- oder Masterstudium codiert)* \
5 = Promotionsstudium \
6 = Weiterbildung \
7 = Praktikum / Werkstudium / Studentische Tätigkeit \
8 = Traineeship / Volontariat \
9 = Anstellung *(festes Arbeitsverhältnis zwischen einer natürlichen Person und einer Organisation ab Junior-Level, darüber hinaus wird nicht zwischen verschiedenen Karrierestufen differenziert.)* \
10 = Freelance *(freiberufliche / selbstständige Tätigkeit)* \
11 = Mitgliedschaft *(Mitgliedschaft (auch bei Alumni) in einer Partei, oder einem Verein, z.B. einer studentischen PR-Initiative / einer Hochschulgruppe. 12 = Founder (Bezieht sich auf Personen, die in einem Unternehmen arbeiten, das sie selbst gegründet haben)* \
13 = Leader *(Bezieht sich auf Personen, die in einem Verein eine leitende Rolle innehatten oder diesen gegründet haben (z. B. Vorstand bei PRIHO)* \
14 = Förderung *(Beschreibt, dass eine natürliche Person durch ein Begabtenförderungswerk mit einem Stipendium gefördert wurde)* \
15 = Lehrtätigkeit *(Lehrtätigkeit von Personen an Hochschulen, entweder in Vollzeit oder als Gastdozierende)* \
16 = Family *(Familiäre Verbindungen, etwa zwischen Elternteil und Kind oder Geschwistern)* \
17 = Mentorship *(Verbindung zwischen Mentor:in und einem #30u30-Mitglied)* \
18 = Freundschaft / Beziehung \
19 = Magister *(Studium, das mit dem nicht mehr existierenden akademischen Grad Magister beendet wurde)* \
20 = Diplom *(Studium, das mit dem nicht mehr existierenden akademischen Grad Diplom beendet wurde)* \
21 = Staatsexamen 

## Nodelist und Node-Attribute
Grundregel: die IDs der Nodelist müssen mit den IDs der Edgelist komplett übereinstimmen. Ausprägungen der Attribute werden in der Regel numerisch codiert.
Da es sich um ein two mode - Netzwerk handelt, in dem sowohl natürliche Personen (Mitglieder der #30u30) als auch Organisationen (Unternehmen, Hochschulen, Vereine etc.) erfasst wurden, beziehen sich manche Node-Attribute auf alle Knoten beziehen, andere definieren nur natürliche Personen oder nur Organisationen näher. Bei diesen Attributen wurde für die Knoten, die davon nicht betroffen sind, kein Wert vergeben (s. oben – Umfang mit fehlenden Werten).

- id: Eindeutige Codierung des Knoten - jede ID entspricht einer natürlichen Person oder einer Organisation, die mit einer Person in Verbindung steht. 
Da die Arbeit mit Initialen als IDs bei 150 Mitgliedern von #30u30, deren Mentor:innen sowie fast 1.000 Organisationen zu viele Dopplungen und damit eine erhöhte Fehlergefahr entstehen. Bei den Knoten der #30u30 wurde daher anhand der Nachnamen codiert (z.B. “schipp” für Linda Schipp), gleiches gilt für die Mentor:innen. Bei der Codierung der Organisationen (Universitäten, Unternehmen oder Vereinen) wurde eine selbstgewählte Abkürzung vergeben, bei Universitäten wurde überwiegend nach einem bestimmten Muster codiert (*hs_ort* für Hochschulen und *uni_ort* für Universitäten).

- name:  Gibt den Namen oder die Bezeichnung des Knotens an. 
Bei Personen wurde hier der Vor- und Nachname angegeben, bei Organisationen ein einheitlicher Name, auf Zusätze wie GmbH, AG, e. V. oder ähnliches wurde der Einfachheit wegen verzichtet. Sonderfall: Im Falle von Umfirmierungen wird der aktuelle Firmenname verwendet, bei Namensänderungen von Personen (z. B. durch Hochzeit) wird ebenfalls der jetzige Name angegeben.

- type: Definiert den Typ der Knoten des two mode-Netzwerks \
1 = natürliche Person \
2 = Organisation (Unternehmen, Agentur, NGO, Partei, Universität etc.) - Organisationstyp wird näher definiert durch Node-Attribut "category" (s. u.)

# Node-Attribute, die sich auf Knoten des Typs 1 (natürliche Personen) beziehen.

- entry: Definiert das Jahr, in dem die jeweiligen Personen in das #30u30-Netzwerk aufgenommen wurden. Bei dem Wert handelt es sich um eine Jahreszahl.

- sex: Definiert das Geschlecht bei natürlichen Personen. \
1 = weiblich \
2 = männlich \
3 = divers (wurde nicht vergeben)

- mentor: Definiert, ob es sich um ein #30u30-Mitglied oder eine:n Mentor:in eines #30u30-Mitglieds handelt. \
1 = 30u30 Mitglied \
2 = Mentor:in

- education: Definiert den höchsten Bildungsabschluss der natürlichen Personen. \
1 = kein Abschluss \
2 = Gymnasialabschluss / Abitur \
3 = Ausbildung \
4 = Bachelorabschluss \
5 = Masterabschluss / MBA \
6 = Magister / Diplom \
7 = PhD (da sich die Personen aufgrund ihres Alters teils noch in der Promotion befinden, wird der Promotionsprozess bereits als angestrebter Abschluss aufgefasst)

- membership: Ist / war die Person Teil einer studentischen PR-Initiative / PR-Verband? \
1 = Ja - Mitglied \
2 = Ja, in einer Gründungs-/Vorstandsrolle \
3 = Nein

- scholarship: Wurde die Person in ihrem Studium durch ein Stipendium gefördert? \
1 = Ja \
2 = Nein \
3 = Hat selbst kein Stipendium erhalten, ist aber Mitglied der Auswahlkommission

- exchange: Definiert, ob die Person in ihrem Studium akademische Auslandserfahrung gesammelt hat. \
1 = Auslandssemester \
2 = Studium im Ausland \ 
3 = beides \
4 = Nein

- abroad: Definiert, ob die Person in ihrem Berufsleben bislang (zeitweise) im Ausland gearbeitet hat - als Ausland gelten Länder außerhalb des DACH-Raumes gezählt. \
1 = Ja \
2 = Nein

- winner: definiert, ob die Person unter den 30 Mitgliedern ihres Jahrgangs als "Young Professional des Jahres" ausgewählt und vom PR Report ausgezeichnet wurden. \
1 = Gewinner:in des "Young Professional des Jahres"-Awards \
2 = Nein

# Node-Attribute, die sich auf Knoten des Typs 2 (Organisationen) beziehen \
- category: Definiert näher, um welche Art der Organisation es sich handelt.\
1 = Unternehmen \
2 = Agentur / Beratungsunternehmen \
3 = NGO / NPO \
4 = Regierungsorganisation / Partei / Behörde o.ä. \
5 = Hochschule \
6 = Verein / Netzwerk \
7 = Begabtenförderungswerk / Stiftung \
8 = Medienunternehmen / Zeitung / Sender \
9 = Sonstiges \
10 = Forschungs-/Bildungseinrichtung / Institut

- ownership: Definiert bei Hochschulen, um welche Art der Hochschule es sich handelt. \
1 = Universität staatlich \
2 = Universität privat \
3 = FH / HAW staatlich \
4 = FH / HAW privat \
5 = Business School / Wirtschaftshochschule \
6 = Weiterbildungsakademie \
7 = Berufsschule \
8 = duale Hochschule \
9 = sonstiges

- sponsor: Definiert, ob die jeweiligen Organisation Partner bzw. Unterstützer des #30u30 Netzwerks ist oder war. \
1 = Ja \
2 = Nein


# Codiervorgaben

## Codiert wird
- Person hat an einer Hochschule studiert – auch nur für ein Auslandssemester oder an einer Institution eine Weiterbildung absolviert.
- Person hat bei einer Organisation gearbeitet (differenziert wird hier nur nach Praktikum/Werkstudium und Festanstellung, nicht nach weiteren Karrierelevels)
- Person war / ist Mitglied in einer studentischen PR-Initiative oder einem PR-Branchenverband, entweder als einfaches Mitglied oder in leitender Funktion.
- Person wird / wurde durch ein Begabtenförderungswerk mit einem Stipendium gefördert.
- Person wurde im Laufe der Karriere von Mentor*innen betreut
- Bei Mentor:innen: Nur der Arbeitsplatz zur Zeit der Mentoring-Beziehung, der Ausbildungsweg, sowie leitende Positionen in Branchenverbänden, z. B. im DPRG-Vorstand

## Codiert wird nicht
- Soziales / freiwilliges Engagement über Mitgliedschaft in PR-Initiativen hinaus (bspw. Engagement in der Flüchtlingshilfe, freiwilliger Feuerwehr usw.)
- Nebenberufliche / Hobbymäßige Tätigkeiten von Personen, bspw. das Betreiben eines privaten Blogs oder eines Podcast, da daraus mit hoher Wahrscheinlichkeit keine Links zu anderen Personen entstehen. Dies gilt nicht, wenn Personen ein Unternehmen gegründet haben, in dem sie (und ggf. Mitarbeitende) beschäftigt sind.
- Person wird mit einem (branchenrelevanten) Preis – z. B. dem PR Report Award o. ä. –
ausgezeichnet, da Preisvergabeprozesse nicht im Fokus der Arbeit stehen und zudem oftmals weitere Personen bei den prämierten Projekten und Kampagnen beteiligt waren
- Bei Mentor:innen werden Beziehungen nur reduziert codiert, die berufliche Karriere wird auf die Stelle während des Mentoring bzw. den Kontaktpunkt zum Mentee reduziert, alle weiteren Stationen werden nicht erfasst. Bei Angaben zum Studium werden Auslandssemester nicht mit aufgenommen, zudem wird auf die Angabe eines Jahres verzichtet, da zum einen die Überschneidungen vermutlich gering sein dürften, die Jahresangabe für die Fragestellung der Arbeit nicht maßgeblich ist und die Angabe ohnehin in vielen LinkedIn-Profilen der Mentor:innen nicht angegeben wurden.
- Mentor:innen werden nur codiert, wenn diese im entspr. *PR Report*-Porträt zum #30u30-Mitglied von der Person genannt wurden. Eine Nachrecherche, bei den #30u30-Mitgliedern, die keine:n Mentor:in genannt haben, ist nicht erfolgt, hier wurde entsprechend keine Mentoring-Beziehung genannt.

## Sonstige Vorgaben: 
- Firmen unter dem gleichen Dach werden zusammengefasst (z. B. *Porsche* und *Porsche Digital* zu *Porsche*)
- Personen, die bei Aufnahme ins #30u30-Netzwerk einen anderen Namen hatten als heute (z. B. aufgrund von Hochzeit), werden i. d. R. mit ihrem aktuellen Namen codiert.
- Gleiches gilt für Firmenfusionen, etwa bei Agenturen: Wenn Person x damals bei Hering Schuppener ein Praktikum gemacht hat, die Agentur inzwischen aber fusioniert hat und mit anderen Agenturen zu Finsbury Glover Hering zusammengeschlossen wurde, wird die Beziehung zu Finsbury Glover Hering angegeben.

### zu id: 
- Großbuchstaben werden kleingeschrieben
- Leerzeichen und Bindestriche (bei Doppelnamen) werden zu Unterstrichen oder zusammengeschriebenen Doppelnamen verkürzt
- Umlaute werden ersetzt (ä zu ae, ö zu oe und ü zu ue; ß zu ss)
- Doktortitel oder Rechtsformen von Firmen (z. B. GmbH) o. ä. werden in der id nicht angegeben
- Bei Namensdopplung: Da die id bei Personen nur aus dem Nachnamen besteht, ist eine Dopplung bei häufigen Namen wie „mueller“ wahrscheinlich – hier wird bei den Namensdopplern mit einem Unterstrich der Vorname ergänzt (mueller, mueller_max, mueller_michael) – handelt es sich bei einer Person mit dem Namen um eine:n der #303u0, behält diese:r den regulären Namen, die Namen von Mentor:innen werden entsprechend angepasst 
