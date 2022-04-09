Datensatz 30u30
Codebuch Stand 2022-04-09 
erstellt von Hannah Bauer (hb062@hdm-stuttgart.de)

## Inhalt
- Edges.csv (Edgelist)
- Nodes.csv (Nodelist)
- Codebuch.md (Codierung der Datensätze)

## Forschungsinteresse, Ursprung und Datenerhebung
Im Rahmen meiner Bachelorarbeit werde ich die letzten fünf Jahrgänge des #30u30-Netzwerks für junge, aufstrebende Talente der Kommunikationsbranche analysieren. Konkret werden die Mitglieder der Jahrgänge 2017 bis 2021 in den Blick genommen. Da die Auswahl der 30 Talente eines Jahrgangs von einer zwei Mann-Jury getroffen wird, stellt sich die Frage, welche Kriterien dahinterstehen und ob in der Zusammenstellung generelle Tendenzen erkennbar sind. Insbesondere soll untersucht werden: Was verbindet die #30u30 über ihre Mitgliedschaft im Nachwuchsnetzwerk hinaus? – gibt es gemeinsame Stationen in Ausbildung oder Beruf?
Die Daten der Mitglieder wurden auf Basis von deren LinkedIn-Profilen erfasst und entsprechend der nachfolgenden Kritierien codiert. Ergänzend wurden teilweise auch die XING-Profile der Personen sowie die entsprechenden Porträts im PR-Report herangezogen.
Das Netzwerk ist ein ungerichtetes two-mode Gesamtnetzwerk, welches zur weiteren Analyse in einzelne Teilnetzwerke aufgeteilt werden kann.

# Umgang mit fehlenden Werten 
Fehlende Werte oder falls sich Werte nur auf einen bestimmten Typ an Nodes bezieht, werden diese nicht erfasst. Das zugehörige Feld in der Edge-List bleibt in diesem Fall frei.

## Edgelist und Edge-Attribute
Grundregel Die Edgelist darf pro Spalte immer nur einen Wert enthalten. Bis auf die ID und den Namen ist dieser idealerweise numerisch codiert (als Zahl).

- from: Die Werte in der Spalte “from” definieren einen Ausgangspunkt einer Beziehung, da es sich um ein ungerichtetes Netzwerk handelt, ist die Richtung der Beziehung nicht relevant, es gibt keinen Sender oder Empfänger. Der eingetragene Wert entspricht einer ID in der Nodelist und enthält keine Sonderzeichen, sondern nur ein Wort.
- to: Die Werte in der Spalte “to” definieren den Empfänger in ungerichteten Netzwerken. Der eingetragene Wert entspricht einer ID in der Nodelist und enthält keine Sonderzeichen, sondern nur ein Wort.
- relationship: Das Edge-Attribut “relationship” definiert die Art der Beziehung zwischen den Knoten, da es sich um ein multiplexes Netzwerk mit verschiedenen Beziehungsarten handelt. Hierbei werden vor allem die Beziehungen zwischen Knoten des Typ 1 und Typ 2 definiert, da wenig Daten zu den Beziehungen zwischen den einzelnen natürlichen Personen vorliegen. Primär wird die Verbindung zwischen natürlicher Person und der Organisation näher definiert, etwa ob diese durch ein Studium, Praktikum o.ä. besteht. Darüber hinaus werden Verbindungen zwischen den #30u30 und ihren Mentorinnen / Mentoren erfasst.
1 = Ausbildung 2 = Bachelorstudium 3 = Masterstudium 4 = Auslandssemester 5 = PhD 6 = Weiterbildung 7 = Praktikum/Werkstudium 8 = Traineeship / Volontariat 9 = Anstellung *beschreibt ein festes Arbeitsverhältnis zwischen einer natürlichen Person und einer Organisation, keine Unterscheidung zwischen Hierarchiestufen* 
10 = Freelance *beschreibt eine freiberufliche Tätigkeit* 
11 = Mitgliedschaft *beschreibt die Mitgliedschaft in einer Partei oder einem Verein, z.B. einer studentischen PR-Initiative oder einer Hochschulgruppe.* 
12 = Ehrenamt *beschreibt das ehrenamtliche Engagement einer natürlichen Person bei einer Organisation.* 
13 = Founder *bezieht sich auf Personen, die in einer Organisation / einem Verein eine Führungsrolle hatten / Gründerin o. Gründer waren*
14 = Förderung *Förderung meint, dass die natürliche Person durch ein Begabtenförderungswerk mit einem Stipendium gefördert wurde.* 
15 = Lehrtätigkeit *beschreibt die Lehrtätigkeit von Personen an Hochschulen, entweder in Vollzeit oder als Gastdozierende*
16 = family *beschreibt familiäre Verbindungen, etwa zwischen Eltern - Kind oder Geschwistern* 
17 = mentorship *beschreibt eine Verbindung zwischen einem Mentor und einem #30u30-Mitglied.*
18 = Freundschaft / Beziehung

- year: Das Edge-Attribut “year” definiert das Jahr, in dem die jeweilige Beziehung bestand, beziehungsweise wann eine Person bei einer Organisation beschäftigt war oder an einer Hochschule studiert hat. Die Zeiträume wurden auf Basis der Angaben in den jeweiligen LinkedIn-Profilen der Personen erfasst. Bei mehrjährigen Verbindungen wurde entsprechend für jedes separate Jahr eine Beziehung angelegt. Die Jahreszahlen reichen von 1961 bis 2022 und umfassen damit den Berufs- und Studienweg der Akteure vor ihrer Aufnahme in das 30u30 Netzwerk bis hin zum aktuellen Zeitpunkt zur Datenerhebung im Rahmen dieser Arbeit (März 2022). Bei Verbindungen, bei denen das Jahr nicht bekannt war, wurde statt einer Jahreszahl kein Wert angegeben.

## Nodelist und Node-Attribute
Da es sich um eine two mode - Akteursnetzwerk handelt, in dem sowohl natürliche Personen (die Mitglieder der #30u30) sowie Organisationen (Unternehmen, Hochschulen, Vereine etc.) erfasst wurden, gibt es Node-Attribute, die sich auf alle Knoten beziehen als auch solche, die nur natürliche Personen oder nur Organisationen näher definieren. Bei diesen Attributen wurde für die Knoten, die davon nicht betroffen sind, kein Wert vergeben.

- id: eindeutige Codierung des Knoten - Jede ID entspricht einer natürlichen Person oder einer Organisation, die mit einer Person in Verbindung steht. ids der #30u30: Die Arbeit mit Initialen würde bei 150 Personen zu vielen Doppelungen führen. Die Knoten der #30u30 werden daher codiert anhand der Nachnamen der Akteurin / des Akteurs (z.B. “Schipp” für Linda Schipp) 
Bei der Codierung der Organisationen (Universitäten, Unternehmen oder Vereinen) wurde eine selbstgewählte Abkürzung gewählt, bei den Arbeitsstellen wurde aufgrund der hohen Zahl an Unternehmen/Agenturen etc.  zur Vermeidung von Doppelungen anhand von Zahlen codiert.

- name:  gibt den Namen oder die Bezeichnung des Knotens an. Bei Personen wurde hier der Vor- und Nachname angegeben, bei Organisationen ein einheitlicher Name, auf Zusätze wie GmbH oder ähnliches wurde der Einfachheit wegen verzichtet.

- type: definiert den Typ der Knoten des two mode-Netzwerks 
1 = natürliche Person 
2 = Organisation (Unternehmen, Agentur, NGO, politische Organisation, Universität etc.)

# Node-Attribute, die Knoten des Typs 1 (natürliche Personen) näher definieren. 
Bei Knoten des Typs 2 (Organisationen) wird für diese Attribute kein Wert vergeben, R Studio erkennt freie Felder automatisch als NA.

- age: definiert das Alter der natürlichen Person zum Zeitpunkt der Aufnahme in das Netzwerk. Wird als numerischer Wert angegeben.

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
6 = Magister, 
7 = PhD (da sich die Personen aufgrund ihres Alters teils noch in der Promotion befinden, wird bereits der Promotionsprozess hier als angestrebter Abschluss erfasst)

- membership: definiert bei natürlichen Personen, ob diese Mitglied in einer studentischen PR-Initiative / PR-Verband sind oder waren.
1 = ja - Mitglied, 2 = Gründungs/Vorstandsrolle, 2 = nein

- scholarship: definiert, ob die Person in ihrem Studium durch ein Stipendium gefördert wurde. 
1 = ja, 2 = nein, 3 = hat selbst kein Stipendium erhalten, ist aber Mitglied der Auswahlkommission

politics: definiert, ob die Person Mitglied einer politischen Partei ist.
1 = ja, 2 = nein

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
2 = Agentur, 
3 = NGO, 
4 = Regierungsorganisation / Partei,
5 = Hochschule,
6 = Verein / Netzwerk, 
7 = Begabtenförderung,
8 = Zeitung / Medienunternehmen,
9 = Sonstiges

- ownership definiert bei Hochschulen, um welche Art der Hochschule es sich handelt.  
1 = Universität staatlich, 2 = Universität privat, 3 = FH / HAW staatlich, 4 = FH / HAW privat, 5 = Business School, 6 = Weiterbildungsakademie, 7 = Berufsschule, 8 = duale Hochschule, 9 = sonstiges

- sponsor definiert, ob die jeweiligen Organisation Partner bzw. Unterstützer des #30u30 Netzwerks ist. 
1 = ja, 
2 = nein
