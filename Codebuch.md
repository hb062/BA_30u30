Datensatz 30u30
Codebuch Stand 2022-05-27
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
 - 
- relationship: Das Edge-Attribut „relationship“ definiert die Art der Beziehung zwischen Knoten, da es sich um ein multiplexes Netzwerk mit verschiedenen Beziehungsarten handelt.  Hierbei wurden vor allem die Beziehungen zwischen Knoten des Typ 1 und Typ 2 (Personen und Organisationen) definiert, da wenig Daten zur Beziehungsqualität zwischen den einzelnen natürlichen Personen vorlagen bzw. mit der gewählten Methode nicht erhoben werden konnten. Primär wird daher die Verbindung zwischen natürlicher Person und der Organisation näher definiert, darüber hinaus werden Verbindungen zwischen den #30u30 und ihren Mentorinnen / Mentoren codiert. 
1 = Ausbildung
2 = Bachelorstudium
3 = Masterstudium,
4 = Auslandssemester *(Gesamte Bachelor-/Masterstudiengänge, die im Ausland absolviert wurden, wurden nicht als Auslandssemester, sondern regulär als Bachelor- oder Masterstudium codiert)*
5 = Promotionsstudium
6 = Weiterbildung
7 = Praktikum / Werkstudium / Studentische Tätigkeit
8 = Traineeship / Volontariat
9 = Anstellung *(festes Arbeitsverhältnis zwischen einer natürlichen Person und einer Organisation ab Junior-Level, darüber hinaus wird nicht zwischen verschiedenen Karrierestufen differenziert.)*
10 = Freelance *(freiberufliche / selbstständige Tätigkeit)*
11 = Mitgliedschaft *(Mitgliedschaft (auch bei Alumni) in einer Partei, oder einem Verein, z.B. einer studentischen PR-Initiative / einer Hochschulgruppe. 12 = Founder (Bezieht sich auf Personen, die in einem Unternehmen arbeiten, das sie selbst gegründet haben)*
13 = Leader *(Bezieht sich auf Personen, die in einem Verein eine leitende Rolle innehatten oder diesen gegründet haben (z. B. Vorstand bei PRIHO)* 
14 = Förderung *(Beschreibt, dass eine natürliche Person durch ein Begabtenförderungswerk mit einem Stipendium gefördert wurde)*
15 = Lehrtätigkeit *(Lehrtätigkeit von Personen an Hochschulen, entweder in Vollzeit oder als Gastdozierende)*
16 = Family *(Familiäre Verbindungen, etwa zwischen Elternteil und Kind oder Geschwistern)*
17 = Mentorship *(Verbindung zwischen Mentor:in und einem #30u30-Mitglied)*
18 = Freundschaft / Beziehung
19 = Magister *(Beschreibt ein Studium, das mit dem nicht mehr existierenden akademischen Grad Magister beendet wurde)*
20 = Diplom *(Beschreibt ein Studium, das mit dem nicht mehr existierenden akademischen Grad Diplom beendet wurde)*
21 = Staatsexamen 

## Nodelist und Node-Attribute
Da es sich um eine two mode - Akteursnetzwerk handelt, in dem sowohl natürliche Personen (die Mitglieder der #30u30) sowie Organisationen (Unternehmen, Hochschulen, Vereine etc.) erfasst wurden, gibt es Node-Attribute, die sich auf alle Knoten beziehen als auch solche, die nur natürliche Personen oder nur Organisationen näher definieren. Bei diesen Attributen wurde für die Knoten, die davon nicht betroffen sind, kein Wert vergeben.

- id: eindeutige Codierung des Knoten - Jede ID entspricht einer natürlichen Person oder einer Organisation, die mit einer Person in Verbindung steht. ids der #30u30: Die Arbeit mit Initialen würde bei 150 Personen zu vielen Doppelungen führen. Die Knoten der #30u30 werden daher codiert anhand der Nachnamen der Akteurin / des Akteurs (z.B. “Schipp” für Linda Schipp). Bei der Codierung der Organisationen (Universitäten, Unternehmen oder Vereinen) wurde eine selbstgewählte Abkürzung gewählt.

- name:  gibt den Namen oder die Bezeichnung des Knotens an. Bei Personen wurde hier der Vor- und Nachname angegeben, bei Organisationen ein einheitlicher Name, auf Zusätze wie GmbH oder ähnliches wurde der Einfachheit wegen verzichtet. Sonderfall: Im Falle von Umfirmierungen wird der aktuelle Firmenname verwendet. Bei Personen, die z. B. bei der Agentur "Hering Schuppener" gearbeitet haben, die nun in Finsbury Glover Hering umfusioniert wurde, wird die Beziehung dennoch mit Finsbury angegeben, um die Zusammenhänge besser abzubilden. Gleiches gilt für "Burson Marsteller", woraus nach Zusammenschluss mit Cohn-Wolfe "Burson Cohne Wolfe" wurde sowie für die Agentur "CNC Communications", aus der nach einem Zusammenschluss "Kekst CNC" wurde. Auch wenn der Firmenname zur Zeit der Beschäftigung der Personen damals noch anders lautete, wird jeweils der aktuelle Name verwendet.

- type: definiert den Typ der Knoten des two mode-Netzwerks 
1 = natürliche Person 
2 = Organisation (Unternehmen, Agentur, NGO, politische Organisation, Universität etc.)

# Node-Attribute, die Knoten des Typs 1 (natürliche Personen) näher definieren. 
- sex: definiert das Geschlecht der natürlichen Personen
1 = weiblich, 2 = männlich, 3 = divers

- mentor: definiert, ob es sich um ein #30u30-Mitglied oder eine Mentorin / einen Mentor eines #30u30-Mitglieds handelt.
1 = 30u30 Mitglied, 2 = Mentorin / Mentor

- entry: definiert das Jahr, in dem die jeweiligen Personen in das #30u30-Netzwerk aufgenommen wurden. Bei dem Wert handelt es sich um eine Jahreszahl. 

- education definiert den höchsten Bildungsabschluss der natürlichen Personen.
1 = kein Abschluss,
2 = Gymnasialabschluss / Abitur,
3 = Ausbildung, 
4 = Bachelor,
5 = Master / MBA,
6 = Magister / Diplom,
7 = PhD (da sich die Personen aufgrund ihres Alters teils noch in der Promotion befinden, wird bereits der Promotionsprozess hier als angestrebter Abschluss erfasst)

- membership: definiert bei natürlichen Personen, ob diese Mitglied in einer studentischen PR-Initiative / PR-Verband sind oder waren.
1 = ja - Mitglied, 2 = Gründungs/Vorstandsrolle, 3 = nein

- scholarship: definiert, ob die Person in ihrem Studium durch ein Stipendium gefördert wurde. 
1 = ja, 2 = nein, 3 = hat selbst kein Stipendium erhalten, ist aber Mitglied der Auswahlkommission

- exchange: definiert, ob die Person in ihrem Studium ein Auslandssemester absolviert hat oder ganz im Ausland studiert hat. 
1 = Auslandssemester, 
2 = Studium im Ausland, 
3 = beides,
4 = nein

- abroad: definiert, ob die Person in ihrem Berufsleben bislang (zeitweise) im Ausland gearbeitet hat - außerhalb des DACH-Raumes. 
1 = ja, 2 = nein

- winner definiert, ob die Person unter den 30 Mitgliedern ihres Jahrgangs als Young Professional des Jahres ausgewählt und vom PR Report ausgezeichnet wurden. 
1 = Gewinner/in des Young Professional Awards, 
2 = nein

# Node-Attribute, die sich auf Knoten des Typs 2 (Organisationen) beziehen
- category: definiert die Art der Organisation genauer
1 = Unternehmen, 
2 = Agentur / Beratungsunternehmen,
3 = NGO / NPO,
4 = Regierungsorganisation / Partei / Behörde o.ä.,
5 = Hochschule,
6 = Verein / Netzwerk, 
7 = Begabtenförderungswerk / Stiftung
8 = Medienunternehmen / Zeitung / Sender,
9 = Sonstiges, 
10 = Forschungs-/Bildungseinrichtung / Institut

- ownership definiert bei Hochschulen, um welche Art der Hochschule es sich handelt.  
1 = Universität staatlich, 2 = Universität privat, 3 = FH / HAW staatlich, 4 = FH / HAW privat, 5 = Business School, 6 = Weiterbildungsakademie, 7 = Berufsschule, 8 = duale Hochschule, 9 = sonstiges

- sponsor definiert, ob die jeweiligen Organisation Partner bzw. Unterstützer des #30u30 Netzwerks ist. 
1 = ja, 
2 = nein
