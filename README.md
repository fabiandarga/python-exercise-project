# Python Workshop: Employee Data Processing Service

## Aufgabenstellung

In diesem Workshop entwickeln wir einen REST-Service mit FastAPI, der CSV-Daten von Mitarbeitern validiert, transformiert und in einer PostgreSQL-Datenbank speichert. 

## Lernziele

- Entwicklung einer REST-API mit FastAPI
- Implementierung von Unittests mit pytest
- Verarbeitung und Validierung von CSV-Daten
- Datenbank-Integration mit pg8000
- Anwendung von Python-Konzepten: Typing, Exception-Handling, Comprehensions, etc.

## Vorbedingungen

- Python 3.7+
- PostgreSQL-Datenbank
- Grundkenntnisse in Python und REST-APIs

## Aufgabe 1: Projektstruktur aufsetzen

Erstelle folgende Projektstruktur:

```
employee_processor/
├── app/
│   ├── __init__.py
│   ├── main.py           # FastAPI Hauptanwendung
│   ├── models.py         # Employee-Datenmodell
│   ├── processors.py     # CSV-Verarbeitung
│   └── database.py       # DB-Verbindung mit pg8000
├── tests/
│   ├── __init__.py
│   ├── test_api.py       # API-Tests
│   ├── test_processors.py # Verarbeitungs-Tests
│   └── test_database.py  # Datenbank-Tests
```

**Installieren der Abhängigkeiten:**
mit pipenv:  
```bash
pipienv install fastapi uvicorn pytest pytest-cov pg8000
```

mit venv + pip:  
```bash
python -m venv .venv
source .venv/bin/activate
python -m pip install fastapi uvicorn pytest pytest-cov pg8000
```

## Aufgabe 2: Employee-Datenmodell implementieren

Erstelle in `models.py` ein Employee-Datenmodell mit folgenden Feldern:

```python
from typing import Literal, Dict, Any
from datetime import date

EducationType = Literal['none', 'hauptschule', 'realschule', 'abitur', 'bachelor', 'master']

class Employee:
    def __init__(
        self,
        first_name: str,
        last_name: str,
        birthday: date,
        salary: int,
        education: EducationType,
        married: bool,
        kids: int = 0
    ):
        self.first_name = first_name
        self.last_name = last_name
        self.birthday = birthday
        self.salary = salary
        self.education = education
        self.married = married
        self.kids = kids
    
    # def to_dict():

    # def from_dict():
```

Implementiere die Methoden `to_dict()` und `from_dict()`, um zwischen Employee-Objekten und Dictionaries zu konvertieren. Berücksichtige dabei Typ-Konvertierungen, z.B. von Strings zu Datumswerten und booleans.

## Aufgabe 3: CSV-Verarbeitung implementieren

Erstelle in `processors.py` Funktionen zum Verarbeiten von CSV-Daten:

```python
import csv
from io import StringIO
from typing import List, Dict, Any, Tuple, Optional
from app.models import Employee

def parse_csv(csv_data: str) -> List[Dict[str, Any]]:
    """CSV-String in eine Liste von Dictionaries umwandeln"""
    # ...
    pass

def validate_employee_data(data: Dict[str, Any]) -> Tuple[bool, Optional[str]]:
    """
    Validiere Mitarbeiterdaten und gib (is_valid, error_message) zurück
    
    Validierungsregeln:
    - Alle Pflichtfelder müssen vorhanden sein
    - birthday muss ein gültiges Datum sein und darf nicht in der Zukunft liegen
    - salary muss eine positive Ganzzahl sein
    - education muss einer der erlaubten Werte sein
    - kids muss eine nicht-negative Ganzzahl sein
    - married muss als boolean interpretierbar sein
    """
    # ...
    pass

def process_employees(csv_data: str) -> Tuple[List[Employee], List[Dict[str, Any]]]:
    """
    Verarbeite CSV-Daten und gib zwei Listen zurück:
    - Gültige Mitarbeiter als Employee-Objekte
    - Ungültige Zeilen mit Fehlermeldungen
    """
    # ...
    pass
```

Beispiel-CSV-Daten für Tests:

```csv
first_name,last_name,birthday,salary,education,married,kids
John,Doe,1980-01-15,60000,master,true,2
Jane,Smith,1985-05-20,75000,bachelor,false,0
Max,Mustermann,1990-03-10,55000,abitur,true,1
```

## Aufgabe 4: Datenbank-Integration implementieren

Erstelle eine PostgreSQL-Tabelle mit folgendem Schema:

```sql
CREATE TABLE IF NOT EXISTS employees (
    id SERIAL PRIMARY KEY,
    first_name VARCHAR(100) NOT NULL,
    last_name VARCHAR(100) NOT NULL,
    birthday DATE NOT NULL,
    salary INTEGER NOT NULL,
    education VARCHAR(20) CHECK (education IN ('none', 'hauptschule', 'realschule', 'abitur', 'bachelor', 'master')),
    married BOOLEAN NOT NULL,
    kids INTEGER NOT NULL DEFAULT 0,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

Implementiere in `database.py` Funktionen für die Datenbank-Kommunikation:

```python
import pg8000
from typing import List
from app.models import Employee
import os

def get_db_connection():
    """Datenbankverbindung herstellen"""
    # ...
    pass

def insert_employee(employee: Employee) -> int:
    """Füge einen Mitarbeiter in die Datenbank ein und gib die ID zurück"""
    # ...
    pass

def insert_employees(employees: List[Employee]) -> List[int]:
    """Füge mehrere Mitarbeiter in die Datenbank ein und gib die IDs zurück"""
    # ...
    pass
```

## Aufgabe 5: FastAPI-Endpunkte implementieren

Implementiere in `main.py` die API-Endpunkte:

```python
from fastapi import FastAPI, HTTPException
from typing import Dict, Any

from app.processors import process_employees
from app.database import insert_employees
from app.models import Employee

app = FastAPI(title="Employee Data Processor")

@app.post("/employee/validate")
async def validate_employees(data: Dict[str, Any]):
    """Validiere CSV-Mitarbeiterdaten ohne sie in die Datenbank einzufügen"""
    # ...
    pass

@app.post("/employee/insert")
async def insert_employee_data(data: Dict[str, Any]):
    """Validiere und füge CSV-Mitarbeiterdaten in die Datenbank ein"""
    # ...
    pass
```

Beispiel-Request-Body:

```json
{
  "csv_data": "first_name,last_name,birthday,salary,education,married,kids\nJohn,Doe,1980-01-15,60000,master,true,2\nJane,Smith,1985-05-20,75000,bachelor,false,0"
}
```

## Aufgabe 6: Tests implementieren

Implementiere Unittests für die Komponenten. Beispiel für einen Test in `test_processors.py`:

```python
import pytest
from app.processors import validate_employee_data
from datetime import date

def test_validate_employee_valid_data():
    data = {
        "first_name": "John",
        "last_name": "Doe",
        "birthday": "1980-01-15",
        "salary": "60000",
        "education": "master",
        "married": "true",
        "kids": "2"
    }
    
    is_valid, error = validate_employee_data(data)
    
    assert is_valid is True
    assert error is None

def test_validate_employee_invalid_education():
    data = {
        "first_name": "John",
        "last_name": "Doe",
        "birthday": "1980-01-15",
        "salary": "60000",
        "education": "phd",  # Ungültiger Wert
        "married": "true",
        "kids": "2"
    }
    
    is_valid, error = validate_employee_data(data)
    
    assert is_valid is False
    assert "Ungültiger Ausbildungsgrad" in error
```

Implementiere auch API-Tests in `test_api.py` mit dem TestClient von FastAPI:

```python
from fastapi.testclient import TestClient
from app.main import app
from unittest.mock import patch

client = TestClient(app)

def test_validate_endpoint_success():
    # Implementieren Sie diesen Test
    pass
```

## Aufgabe 7: Erweiterte Features (wenn Zeit bleibt)

Mögliche Erweiterungen:

1. **Datei-Upload-Endpunkt**: Implementieren einen Endpunkt, der CSV-Dateien per Multipart-Upload akzeptiert
2. **Statistik-Endpunkt**: Erstelle einen Endpunkt, der Statistiken über die Mitarbeiterdaten liefert (z.B. Durchschnittsgehalt)
3. **Verbessertes Error-Handling**: Implementiere detailliertere Fehlermeldungen und Logging
4. **Typisierte Eingaben mit Pydantic**: Erweiter die API um Pydantic-Modelle für typsichere Eingaben

## Beispieldaten

Hier ist ein umfangreicheres Beispiel für CSV-Daten zum Testen:

```csv
first_name,last_name,birthday,salary,education,married,kids
John,Doe,1980-01-15,60000,master,true,2
Jane,Smith,1985-05-20,75000,bachelor,false,0
Max,Mustermann,1990-03-10,55000,abitur,true,1
Erika,Musterfrau,1975-12-25,85000,master,false,3
Hans,Schmidt,1982-07-30,50000,realschule,true,0
Invalid,,1990-01-01,not-a-number,phd,maybe,two
Peter,Parker,1996-02-20,3000,abitur,false,0
Tina,Müller,1984-04-12,43500,realschule,false,2
```

## Hilfreiche Tipps

- Nutze Type Hints konsequent für bessere Code-Qualität
- Verwende List Comprehensions für kompakteren Code
- Arbeite mit try-except Blöcken für robuste Fehlerbehandlung
- Teste die API mit Swagger UI unter http://localhost:8000/docs
