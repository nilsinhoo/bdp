# OLS-Benchmarking mit Apache Spark

Dieses Projekt untersucht die **Performanz von OLS-Regressionen in Spark MLlib** in verschiedenen Cluster-Konfigurationen.  
Die Case Study wurde am Standort Iserlohn der Fachhochschule Südwestfalen im Rahmen des Studienganges **Angewandte Informatik** (M.Sc.) als Teil des Moduls **Big Data Processing** (Prof. Dr. Benjamin Buchwitz) im Sommersemester 2025 von Nils Kranefeld (30418372) angefertigt.


## Quellen

Die theoretischen Grundlagen und Spark-spezifischen Informationen sind im Poster fortlaufend mit `[n]` referenziert:

[1] Apache Spark, *Spark Release 3.5.0*. Apache Software Foundation, 2025. [Online]. Available: https://spark.apache.org/releases/spark-release-3-5-0.html

[2] A. Ng, *CS229: Supervised Learning Lecture Notes*. Stanford Univ., 2003. [Online]. Available: https://see.stanford.edu/materials/aimlcs229/cs229-notes1.pdf

[3] A.-L. Cholesky (publ. by A. Benoît), “Sur la résolution numérique des systèmes d’équations linéaires,” *Bulletin Géodésique*, vol. 2, no. 7, pp. 67–77, 1924.

[4] G. H. Golub and C. F. Van Loan, *Matrix Computations*, 4th ed. Baltimore, MD, USA: Johns Hopkins Univ. Press, 2013, pp. 159–164.

[5] C.-H. Hsieh, “Linear Regression,” Lecture Notes, Univ. of California, Los Angeles (UCLA), 2019. [Online]. Available: https://web.cs.ucla.edu/~chohsieh/teaching/CS260_Winter2019/notes_linearregression.pdf

[6] Apache Spark, “Advanced topics: Optimization of linear methods (developer),” *Spark 3.5.0 Documentation*, 2023. [Online]. Available: https://spark.apache.org/docs/3.5.0/ml-advanced.html

[7] J. Dowling, “Large-Scale Machine Learning and Deep Learning (ID2223): Linear Regression,” Lecture Slides, KTH, Nov. 2, 2017, pp. 67–71.


---

## Projektstruktur

Innerhalb des Jupyter-Arbeitsverzeichnisses (`work/`) ist die Struktur wie folgt:

```
work/
├── notebooks/
│   ├── ols-spark.ipynb   # Generierung synthetischer Daten + OLS-Benchmarks in Spark
│   └── plots.ipynb       # Auswertung + Matplotlib-Plots für das Poster
│
├── stats/                # CSV-Dateien mit Trainingsmetriken (Sparkmeasure + OLS Summary)
│   ├── local-var_fts.csv
│   ├── multi-var_fts.csv
│   ├── local-var_dpts.csv
│   └── multi-var_dpts.csv
│
└── img/                  # Visualisierungen für das Poster (generiert mit matplotlib)
```

---

## Start der Docker-Compose-Anwendung

Die Benchmarks können in drei Modi durchgeführt werden.  
Jeder Modus erfordert eine angepasste Einstellung der Umgebungsvariable `MODE` im Notebook (`ols-spark.ipynb`):

1. **Local Mode**
   - Nutzung eines lokalen Spark-Contexts (`local[N]`).
   - Kein Cluster, alle Kerne laufen im Jupyter-Container.

2. **Single Mode**
   - Start eines einzelnen Spark-Workers über den Spark Master.
   - Simulation einer minimalen Cluster-Umgebung.

3. **Multi Mode**
   - Start von zwei Spark-Workern (jeweils mit mehreren Cores).
   - Untersuchung der Skalierung und parallelen Datenverarbeitung.

Im Laufe der Case Study stellte sich heraus, dass sich die im Single Mode nicht signifikant vom Local Mode unterschieden, sodass im weiteren Verlauf lediglich die Modi **Local Mode** und **Multi Mode** genauer betrachtet wurden. 

Durch die Verwendung der ENV-Files `.env.local`, `.env.cluster-1w` und `.env.cluster-2w` können die verschiedenen Modi angegeben und gestartet werden.

### Schritte zum Starten

1. Repository klonen und in das Projektverzeichnis wechseln.  
2. Docker Compose mit gewünschtem Modus starten:  
   ```bash
   docker compose --env-file .env.local up -d
   docker compose --env-file .env.cluster-1w up -d --scale spark-worker=1
   docker compose --env-file .env.cluster-2w up -d --scale spark-worker=2
   ```
3. JupyterLab im Browser öffnen (typischerweise `http://localhost:18888`).  
4. Im Notebook (`ols-spark.ipynb`) den gewünschten **MODE** setzen (`"local"`, `"single"`, `"multi"`).  
5. Notebook ausführen, um Datensätze zu generieren, Benchmarks zu fahren und CSV-Dateien in `work/stats/` abzulegen.  

---

## Beschreibung der Notebooks

### `notebooks/ols-spark.ipynb`
- Kernstück des Projekts.  
- Enthält den Code zur **Generierung synthetischer Daten** ($y = X\beta + \varepsilon$).  
- Führt OLS-Regressionen in Spark MLlib durch (`LinearRegression` mit `solver="normal"`).  
- Misst Trainingszeit und Ressourcennutzung mit **sparkmeasure**.  
- Exportiert Ergebnisse als CSV in den Ordner `stats/`.  

### `notebooks/plots.ipynb`
- Lädt CSV-Dateien aus `stats/`.  
- Erstellt **Matplotlib-Grafiken** (z. B. Trainingszeit vs. Anzahl Features oder Datensätze).  
- Exportiert Plots ins Verzeichnis `img/` für die Poster-Erstellung.  

---

## Poster und Ergebnisse

- Alle Benchmarks (CSV-Dateien in `stats/`) bilden die Grundlage für die Diagramme im Poster.  
- Poster referenziert theoretische Grundlagen mit `[n]`.  
- Plots stammen direkt aus `plots.ipynb`.  

---

## Einschränkungen der Case Study

Die vorliegende Fallstudie unterliegt mehreren Einschränkungen, die bei der Interpretation der Ergebnisse berücksichtigt werden müssen:

- **Betriebssystem**: Die Experimente wurden nicht unter einem Echtzeitbetriebssystem (RTOS), sondern unter Windows 11 durchgeführt. Dies kann zu schwankenden Laufzeiten durch Hintergrundprozesse führen.  
- **Netzwerkmodellierung**: Es fand keine echte Netzwerkkommunikation zwischen verteilten Knoten statt, da alle Prozesse auf demselben Rechner ausgeführt wurden. Dadurch fehlen potenzielle Latenzen und Kommunikationskosten, die in einem realen Cluster auftreten.  
- **Virtualisierungsschicht**: Die gesamte Ausführung erfolgte in Docker-Containern. Dies kann zu geringfügigen Unregelmäßigkeiten in der Ressourcennutzung (CPU-, Speicher- oder I/O-Overhead) führen.  
- **Clustergröße**: Die Fallstudie beschränkte sich auf ein Single-Node-Setup mit simulierten Partitionen. Effekte, die sich erst bei einer größeren Anzahl physisch getrennter Knoten zeigen, konnten nicht erfasst werden.  

## Fazit
Trotz dieser Einschränkungen verdeutlicht die Fallstudie, dass Apache Spark für große Datenmengen eine geeignete Plattform darstellt. Die Beschleunigung durch die verteilte Berechnung überwiegt ab einem bestimmten Break-Even-Punkt die Paralle­lisierungskosten und führt zu einer messbaren Amortisierung.