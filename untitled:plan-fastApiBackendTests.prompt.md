## Plan: FastAPI-backendtests

Voeg een aparte `tests/`-directory toe met HTTP-level tests voor alle bestaande FastAPI-routes. Gebruik `TestClient`, een autouse-fixture die de globale in-memory `activities`-state per test herstelt, en documenteer hoe de tests worden uitgevoerd.

**Stappen**
1. Voeg de dependency `pytest` als aparte regel toe aan het rootbestand `requirements.txt`; behoud de bestaande FastAPI/httpx-stack zodat `fastapi.testclient.TestClient` daarop kan steunen.
2. Maak `tests/conftest.py` met een autouse-fixture die vóór/na elke test een deep copy van `src.app.activities` gebruikt en de oorspronkelijke dictionary in-place herstelt. Dit voorkomt volgorde-afhankelijkheid zonder de modulevariabele te vervangen.
3. Maak `tests/test_app.py` met een gedeelde `TestClient(app)` en onafhankelijke tests volgens het AAA-patroon. Elke test krijgt herkenbare `Arrange`, `Act` en `Assert`-secties; assertions blijven gericht op statuscode, response-body en relevante state-mutatie. Dek minimaal af:
   - `GET /`: redirect naar `/static/index.html` met status `307`.
   - `GET /activities`: volledige activiteitenstructuur en deelnemers teruggeven.
   - `POST /activities/{activity_name}/signup`: succesvolle inschrijving en mutatie van de deelnemerslijst.
   - POST voor onbekende activiteit: `404` met `Activity not found`.
   - dubbele inschrijving: `400` met de bestaande foutmelding.
   - ontbrekende `email`: `422` voor POST en DELETE.
   - `DELETE /activities/{activity_name}/signup`: succesvolle uitschrijving.
   - DELETE voor onbekende activiteit: `404`.
   - DELETE voor een niet-ingeschreven deelnemer: `404`.
   - een gerichte isolatiecheck waaruit blijkt dat mutaties na een test niet doorwerken naar de volgende test.
4. Breid `src/README.md` beperkt uit met DELETE als API-endpoint, de actuele testinstallatie/-commando’s en de opmerking dat de state in memory wordt bewaard.
5. Controleer de implementatie met `python -m pytest -q`; voer daarnaast een gerichte test uit voor signup/delete en controleer dat importen vanuit `tests/` met de bestaande `pytest.ini` werken.

**Relevante bestanden**
- `/workspaces/skills-getting-started-with-github-copilot/src/app.py` — bestaand routecontract en de globale `activities`-state; niet wijzigen.
- `/workspaces/skills-getting-started-with-github-copilot/tests/conftest.py` — nieuwe state-isolatiefixture.
- `/workspaces/skills-getting-started-with-github-copilot/tests/test_app.py` — nieuwe route- en gedragstests.
- `/workspaces/skills-getting-started-with-github-copilot/requirements.txt` — `pytest` toevoegen.
- `/workspaces/skills-getting-started-with-github-copilot/src/README.md` — API- en testdocumentatie bijwerken.
- `/workspaces/skills-getting-started-with-github-copilot/pytest.ini` — waarschijnlijk ongewijzigd; `pythonpath = .` is al voldoende. `testpaths = tests` is optioneel en alleen nodig als de default discovery expliciet gemaakt moet worden.

**Verificatie**
1. Installeer dependencies met `python -m pip install -r requirements.txt`.
2. Voer `python -m pytest tests/test_app.py -q` uit voor de nieuwe tests.
3. Voer `python -m pytest -q` uit om discovery en de volledige suite te controleren.
4. Controleer specifiek dat signup gevolgd door delete werkt en dat elke test met de oorspronkelijke deelnemerslijsten start.

**Besluiten**
- Tests spreken de API aan via `TestClient`; hierdoor worden routing, queryvalidatie, statuscodes en response-body samen getest.
- De globale dictionary wordt in-place hersteld met een deep copy; alleen herassignen van `activities` is onvoldoende robuust voor bestaande referenties.
- De bestaande functionaliteit wordt vastgelegd, niet uitgebreid: maximale capaciteit en inhoudelijke e-mailvalidatie blijven buiten scope omdat `src/app.py` die momenteel niet afdwingt.
- `src/app.py` wordt niet aangepast; eventuele ontbrekende businessregels zijn een afzonderlijk wijzigingsvoorstel.
