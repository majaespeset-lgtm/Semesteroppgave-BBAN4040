# Semesteroppgave høst 2026: Klassifisering av kredittkortklager

**Gruppe:** 0302  
**Gruppemedlemmer:** Maja Tronstad Espeset (10003), Lona Ilebrekke (10076)  
**Emne:** BBAN4040 Språkteknologi med vekt på store språkmodeller  

Prosjektet undersøker hvor godt en stor språkmodell (GPT-6 Luna, zero-shot) klassifiserer kredittkortklager til riktig problemkategori, sammenlignet med en logistisk regresjonsmodell med TF-IDF-vektorer.

#### **Problemstilling:** 
*Hvor godt klarer en stor språkmodell uten treningseksempler å klassifisere kredittkortklager i riktig problemkategori sammenlignet med logistisk regresjon, og hvordan skiller feilene til de to metodene seg fra hverandre?*

---

## Kort om data og metode

#### **Data:** 
Consumer Complaint Database fra CFPB (Kaggle-versjonen: https://www.kaggle.com/datasets/selener/consumer-complaint-database).
- Benytter kun klager i fritekst (*Consumer complaint narrative*), i produktfamilien *Credit card or prepaid card*.
- Avgrenset til de sju største problemkategoriene.
- Etter at duplikater er fjernet, består datasettet av 17 912 klager.
- **Splitt:** 70 % treningssett (12 537), 10 % valideringssett (1 792) og 20 % testsett (3 583). Oppdelingen er stratifisert etter kategori, med `random_state=1`.

#### **Metoder:**
  - Baseline som alltid gjetter den største kategorien. Brukes som referansepunkt.
  - Logistisk regresjon med TF-IDF-vektorer med `class_weight='balanced'`.
  - LLM: GPT-6 Luna via OpenAI API, zero-shot, prompt B, `reasoning={'effort': 'none'}`, standard temperatur (1).

#### **Evaluering:** 
- Macro-F1 er hovedmålet, og accuracy og F1 per kategori brukes som supplement.
- Paired bootstrapping (1000 trekk) for å sjekke om forskjellen mellom metodene er reell.
- Feilene mellom metodene sammenlignes i forvekslingsmatriser, krysstabell, en lengdeanalyse og en manuell feilanalyse.

---

## Organisering

| Fil eller mappe | Innhold |
|---|---|
| `Notebook_1_-_Databehandling_og_logistisk_regresjon.ipynb` | Leser rådata, avgrenser og renser datasettet, lager trenings-, validerings- og testsett, trener logistisk regresjon og baseline. |
| `Notebook_2_-_LLM__zero_shot_.ipynb` | Tester prompt A og B på valideringssettet og klassifiserer testsettet med prompt B via OpenAI API. |
| `Notebook_3_-_Resultater.ipynb` | **Hovednotebook.** Samler resultatene fra notebook 1 og 2 og lager alle tabellene og figurene i rapporten, bootstrapping og utvalget til manuell feilanalyse. |
| `Demo.ipynb` | Liten reproduksjonsdemo (se under). |
| `topp7.csv` | De 17 912 klagene etter avgrensning og fjerning av duplikater (fra notebook 1). |
| `treningssett.csv`, `valideringssett.csv`, `testsett.csv` | Datasplittene fra notebook 1 (Complaint ID, kategori og klagetekst). |
| `resultater_lr.csv` | Prediksjonene fra logistisk regresjon og baseline på testsettet (fra notebook 1). |
| `llm_validering.csv`, `llm_test.csv` | Svarene fra LLM-en på validerings- og testsettet (fra notebook 2). |
| `manuell_feilanalyse.xlsx` | Utvalget på 60 klager med vår manuelle kategorisering av feiltyper. |

**Arbeidsflyt:** Notebook 1 → Notebook 2 → Notebook 3. 
- Hver notebook bygger på filene den forrige lagret.
- Alle CSV-filene ligger i repoet, så notebook 2, notebook 3 og demoen kan kjøres direkte uten å kjøre notebook 1 først.
- Notebook 2 og demoen krever API-nøkkel, mens notebook 1 og 3 kan kjøres uten.

**Ikke inkludert:** Rådatafilen `Kundeklager.csv` (1,28 millioner klager) er ikke med i repoet fordi den er for stor. Den trengs bare for å kjøre notebook 1. Se «Kjøre hele analysen» for hvor den kan lastes ned.

---

## Installasjon

Krever Python 3.10 eller nyere, og disse pakkene: 

| Pakke | Brukes til |
|---|---|
| pandas | Lese, bearbeide og lagre data |
| numpy | Beregninger (IDF-analyse, bootstrap) |
| scikit-learn | TF-IDF, logistisk regresjon, baseline, datasplitt og evaluering |
| matplotlib | Figurer og forvekslingsmatriser |
| openai | Kall til GPT-6 Luna via API |
| python-dotenv | Leser API-nøkkelen fra `.env` |
| openpyxl | Lagrer Excel-filen til feilanalysen |

Demoen alene trenger bare pandas, scikit-learn, openai og python-dotenv. I tillegg brukes `os`, `time`, `json`, `datetime`, `textwrap` og `concurrent.futures`, som følger med Python.

For å kalle OpenAI API trengs en egen API-nøkkel. Lag en fil som heter `.env` i prosjektmappen, med denne linjen:

```
OPENAI_API_KEY=din-nøkkel-her
```

Nøkkelen leses inn med `python-dotenv` og står aldri i koden. `.env` er oppført i `.gitignore` og er ikke med i leveringen.

---

## Kjøre demoen

Demoen viser prosjektets sentrale metode, sammenligningen av logistisk regresjon og LLM, på et lite utvalg av de faktiske dataene:

1. Trener den balanserte logistiske regresjonsmodellen på treningssettet, med samme oppsett som i notebook 1.
2. Trekker et stratifisert utvalg fra testsettet (350 klager, med samme fordeling mellom kategoriene som i testsettet, fast startverdi) og klassifiserer det med baselinen, den logistiske regresjonsmodellen og LLM-en (GPT-6 Luna med prompt B). Prompten og innstillingene er de samme som i notebook 2.
3. Sammenligner LLM-svarene med svarene fra kjøringen 23.10.2026 (lagret i `llm_test.csv`) og viser hvor mange klager som fikk et annet svar. Det illustrerer at svarene kan variere fra kjøring til kjøring med temperatur lik 1.
4. Viser accuracy, macro-F1, F1 per kategori og noen eksempler der modellene er uenige.

##### **Slik kjører du den:**

1. Installer pakkene som beskrevet over, og lag `.env` med API-nøkkelen (se "Installasjon"). Demoen stopper med en feilmelding hvis nøkkelen mangler. 
2. Sjekk at `treningssett.csv`, `testsett.csv` og `llm_test.csv` ligger i samme mappe som `Demo.ipynb`. De ligger i repoet, så notebookene trenger ikke kjøres først.
3. Åpne `Demo.ipynb` i Jupyter Notebook og velg **Run → Run All Cells**.

**Tid og kostnad:**
- Kjøretiden er omtrent ett minutt.
- API-kostnaden er omtrent 0,03 USD: 350 kall og om lag 240 000 input-tokens.
- Demoen beregner kostnaden før API-kallene og stopper hvis anslaget er over 1 USD.
- Til slutt skrives faktisk tokenbruk og kostnad ut.

**Logging:** Hvert API-kall lagres som én linje i `logs/demo_logg.jsonl`. Linjen inneholder tidspunkt, Complaint ID, modell, innstillinger, prompt, fasit, svar, om svaret er en gyldig kategori, og tokenbruk.

**Tolkning av resultatene:** Resultatene i demoen kan avvike fra hele analysen fordi utvalget er lite og fordi LLM-en kan svare annerledes ved hver kjøring (temperatur lik 1). Forskjellen mellom modellene i rapporten er liten, så den er ikke nødvendigvis synlig på et så lite utvalg.

---

## Kjøre hele analysen

1. **Notebook 1:** Last ned datasettet fra https://www.kaggle.com/datasets/selener/consumer-complaint-database, gi filen navnet `Kundeklager.csv` og legg den i prosjektmappen. Notebooken lager `topp7.csv`, de tre datasplittene og `resultater_lr.csv`. Den bruker ingen API.
2. **Notebook 2:** Krever API-nøkkel. Notebooken gjør om lag 7 200 API-kall: valideringssettet med prompt A og B, og testsettet med prompt B. Estimert kostnad er under 0,50 USD.
   - Svarene lagres fortløpende i `llm_validering.csv` og `llm_test.csv`. Klager som allerede har et svar, sendes ikke på nytt, så notebooken kan kjøres flere ganger uten nye kostnader knyttet til klassifisering.
   - Notebooken krever API-nøkkel også når de lagrede svarene brukes, fordi de første cellene kobler til API-et og sender et testspørsmål. Kostnaden for dette er ubetydelig.
3. **Notebook 3:** Leser `resultater_lr.csv` og `llm_test.csv` og lager alle resultatene i rapporten. Bruker ingen API. `manuell_feilanalyse.xlsx` blir ikke overskrevet hvis filen finnes.

---

## Reproduserbarhet

- **Faste startverdier:** `random_state=1` brukes for datasplitten, demoutvalget og utvalget til feilanalysen, og `default_rng(1)` brukes for paired bootstrapping.
- **LLM-kjøringen:**
  - Modell: `gpt-6-luna` via OpenAI API, kjørt 23.10.2026.
  - Innstillinger: `reasoning={'effort': 'none'}` og standard temperatur (1).
  - Prompt A og B står i notebook 2.
  - Alle input og svar er lagret i `llm_validering.csv` og `llm_test.csv`.
- **Svarene kan variere:** Fordi LLM-en kjøres med temperatur lik 1, kan en ny kjøring gi noe andre svar. Resultatene i rapporten bygger på de lagrede svarene.
