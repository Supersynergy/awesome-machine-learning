# Welcher Benchmark passt zur Entscheidung?

Ein gutes Ergebnis auf einem öffentlichen Datensatz ist ein Grund, ein Verfahren auszuprobieren. Für die Auswahl zählen auch Suchbudget, Datenverfügbarkeit und die spätere Nutzung. Die folgenden Versuchspläne leiten sich aus veröffentlichten Arbeiten ab. Die berichteten Befunde stammen von deren Autoren.

## Tabellendaten · Was bringt zusätzliches Training?

TabArena startete mit 16 Verfahren auf 51 Datensätzen. In der veröffentlichten Auswertung blieben gradientengeboostete Bäume konkurrenzfähig. Mit größeren Zeitbudgets und Ensembles schlossen Deep-Learning-Verfahren auf. Wer nur den besten Rang übernimmt, übernimmt damit auch Annahmen über verfügbare Trainingszeit und die Anzahl der Modelle pro Vorhersage. [TabArena-Studie, NeurIPS 2025](https://proceedings.nips.cc/paper_files/paper/2025/hash/1697e3fb412da11dc9488249f9e7bbc9-Abstract-Datasets_and_Benchmarks_Track.html) · [Benchmark und Protokoll](https://github.com/autogluon/tabarena)

**Versuch:** Für einen Klassifikator CatBoost und ein AutoGluon-Ensemble auf identischen äußeren Splits vergleichen. Zwei vorab festgelegte Trainingsbudgets einsetzen, etwa 10 und 60 Minuten pro Verfahren und Split. Hyperparameter nur innerhalb des Trainingsanteils auswählen. Neben der Testmetrik die gesamte Suchzeit, Modellgröße und Latenz für eine einzelne Zeile erfassen. So wird sichtbar, ob ein kleiner Gütegewinn zusätzliche Trainings- oder Betriebskosten rechtfertigt. [Beispiel für verschachtelte Kreuzvalidierung](https://scikit-learn.org/stable/auto_examples/model_selection/plot_nested_cross_validation_iris.html)

## Zeitreihen · Welche Daten waren am Prognosestichtag bekannt?

GIFT-Eval umfasst 23 Datensätze aus sieben Domänen mit zehn Abtastfrequenzen. Die Auswertung unterscheidet unter anderem vortrainierte und nachtrainierte Verfahren und dokumentiert Überschneidungen mit Testdaten. Für eine Wochenprognose sind deshalb Frequenz, Horizont und Herkunft der Trainingsdaten entscheidende Filter beim Lesen der Ergebnisse. [Datensatzkarte](https://huggingface.co/datasets/Salesforce/GiftEval) · [Evaluationsprotokoll](https://github.com/SalesforceAIResearch/gift-eval)

**Versuch:** Für tägliche Nachfrage sieben Tage vorhersagen. Als saisonale Baseline dient der Wert desselben Wochentags der Vorwoche. Baseline und Kandidat an mehreren historischen Stichtagen vergleichen und Fehler je Horizonttag ausweisen. Geänderte Preise als Eingabe nur dann zulassen, wenn sie am jeweiligen Stichtag bereits feststanden. Bei einem vortrainierten Modell die bekannte Herkunft der Trainingsdaten neben das Ergebnis schreiben.

## Dokumentensuche · Findet das Modell die richtige Passage?

In BEIR erwies sich die lexikalische Suche mit BM25 über unterschiedliche Domänen hinweg als robuste Baseline. Nachsortieren mit einem weiteren Modell erzielte im Studienmittel starke Ergebnisse und erhöhte den Rechenaufwand. MMTEB ergänzt sprach- und aufgabenspezifische Tests, darunter die Suche in langen Dokumenten. Für ein deutschsprachiges Handbuch ist ein passender Retrieval-Test aussagekräftiger als der Mittelwert über alle Embedding-Aufgaben. [BEIR, NeurIPS 2021](https://arxiv.org/abs/2104.08663) · [MMTEB, ICLR 2025](https://proceedings.iclr.cc/paper_files/paper/2025/hash/fc0e3f908a2116ba529ad0a1530a3675-Abstract-Conference.html)

**Versuch:** Dokumentstand und Passagegrenzen einfrieren. Reale Suchfragen mit relevanten Passagen verknüpfen. Zuerst BM25 und Embeddings vergleichen, danach denselben Kandidatensatz mit einem Reranker nachsortieren. Recall@20 misst, welcher Anteil der relevanten Passagen unter den ersten 20 liegt. nDCG@10 gewichtet relevante Treffer auf frühen Plätzen stärker. Fehlt die richtige Passage bereits im Kandidatensatz, kann Nachsortieren sie nicht zurückholen.

## Inferenz · Wartet ein Mensch oder verarbeitet ein Job einen Stapel?

MLPerf definiert verschiedene Lastszenarien. Im Offline-Szenario stehen alle Eingaben zu Beginn bereit, gemessen wird der Durchsatz. Im Server-Szenario treffen Anfragen über die Zeit ein und der Durchsatz muss ein Latenzziel einhalten. Deshalb kann dasselbe System für nächtliche Batchverarbeitung und für eine interaktive Anwendung unterschiedlich geeignet sein. [MLPerf Inference Rules](https://github.com/mlcommons/inference_policies/blob/master/inference_rules.adoc)

**Versuch:** Für einen Batchjob Datensätze pro Sekunde und Spitzenspeicher messen. Für eine API bei vorgegebener Ankunftsrate die 95. und 99. Perzentile der Antwortzeit einschließlich Warteschlange erfassen. Modellversion, Präzision, Hardware und Eingabelängen festhalten. Modellgüte nach jeder Quantisierung erneut prüfen. Hyperfine kann den kompletten Batchbefehl wiederholt messen. Für API-Perzentile braucht es Zeitstempel je Anfrage. [Hyperfine-Dokumentation](https://github.com/sharkdp/hyperfine#exporting-results)

## Seltene Treffer

Eine Qualitätsprüfung soll fehlerhafte Teile markieren. In diesem **Rechenbeispiel** sind 100 von 10.000 Teilen fehlerhaft. Der gewählte Schwellenwert erkennt 80 davon (80 % Recall) und markiert 5 % der 9.900 einwandfreien Teile irrtümlich. Das ergibt 495 Fehlalarme. Von 575 markierten Teilen sind nur 80 tatsächlich fehlerhaft (13,9 % Präzision). Die Fehlalarmrate allein sagt dem Prüfteam also wenig über seinen Arbeitsaufwand. Eine Precision-Recall-Kurve zeigt, wie viele Treffer bei höherem Recall noch relevant sind. Beim Vergleich zweier Kurven die Häufigkeit der positiven Klasse mit angeben. [Saito und Rehmsmeier, PLOS ONE](https://journals.plos.org/plosone/article?id=10.1371/journal.pone.0118432)

**Auswahl unter einem Tagesbudget:** Kann das Team nur 100 Teile pro Tag nachprüfen, pro Tag die 100 höchsten Scores auswählen und Trefferzahl sowie verpasste Fehler erfassen. Bei einem festen Score-Schwellenwert stattdessen auch die schwankende Zahl der Alarme berichten. Beide Regeln auf früheren Tagen entwickeln und anschließend unverändert auf späteren Tagen auswerten. scikit-learn stellt für die Optimierung eines festen Schwellenwerts anhand einer Zielmetrik eine eigene Kreuzvalidierung bereit. [Threshold-Tuning mit scikit-learn](https://scikit-learn.org/stable/modules/classification_threshold.html)

## Kalibrierung

Eine Qualitätsprüfung kann Defekte gut nach Dringlichkeit sortieren und trotzdem ihr absolutes Risiko überschätzen. Um eine Vorhersage von 80 % zu prüfen, Fälle mit ähnlichem Score zusammenfassen und den beobachteten Defektanteil danebenstellen. Die Anzahl je Gruppe gehört ins Diagramm: 8 Treffer unter 10 Fällen und 800 unter 1.000 ergeben denselben Anteil bei unterschiedlicher Unsicherheit. [Kalibrierungsdiagramme und Metriken](https://scikit-learn.org/stable/modules/calibration.html)

**Versuch:** Modell auf dem Trainingssatz fitten, einen separaten Datensatz zum Kalibrieren verwenden und Rohscores sowie kalibrierte Wahrscheinlichkeiten auf identischen Testfällen vergleichen. Neben dem Zuverlässigkeitsdiagramm Log-Loss oder Brier-Score berichten. Ein besserer Brier-Score allein beweist keine bessere Kalibrierung, weil auch die Trennfähigkeit in ihn eingeht. Guo und Kollegen zeigten, dass bei mehreren untersuchten neuronalen Netzen ein einzelner Temperaturparameter die Wahrscheinlichkeiten verbessern konnte. Beim binären Modell erhält eine positive Temperatur die Rangfolge der Scores. Dadurch kann die Risikoschätzung besser werden, obwohl sich die ersten 100 Prüffälle nicht ändern. [Guo et al., ICML 2017](https://proceedings.mlr.press/v70/guo17a.html)

## Datenverschiebung

WILDS untersucht reale Unterschiede zwischen Trainings- und Einsatzdaten, darunter Aufnahmen verschiedener Kamerafallen und Daten unterschiedlicher Krankenhäuser. In den untersuchten Aufgaben fiel die Modellgüte beim Wechsel der Verteilung oft deutlich ab. Für eine industrielle Sichtprüfung ergibt sich daraus eine konkrete Frage: Wie viele Fehler erkennt das Modell auf einer Kamera, deren Aufnahmen beim Training fehlten? [Koh et al., ICML 2021](https://proceedings.mlr.press/v139/koh21a.html)

**Versuch:** Eine komplette Kamera oder einen Standort für den abschließenden Test zurückhalten. Serienbilder desselben Teils gemeinsam einem Split zuordnen. Baseline und Kandidat auf exakt denselben Teilen bewerten. Je Kamera Zahl der Teile, Zahl der Defekte, verpasste Defekte und Fehlalarme dokumentieren. Eine Kamera mit null Defekten erlaubt eine Schätzung der Fehlalarmrate, ihr Recall ist undefiniert. Für die Freigabe sowohl den über alle Teile gewichteten Wert als auch die Werte der einzelnen Kameras ansehen: Eine stark genutzte Kamera kann im Gesamtwert den Ausfall einer kleineren verdecken.
