# Qflow-API - Benutzerdokumentation -

## Hinweise für Entwickler
Die Schnittstelle ist eine REST-API, die den Zugriff über einen POST-Request ermöglicht. Für jede Anfrage sind folgende Angaben erforderlich:

- Endpoint: Die URL der API, z. B.: <mark>https://your-api-endpoint.com/api/v1/model/</mark>
- Authentifizierung: Ein gültiger Bearer Token, der im HTTP-Header als <mark>Authorization</mark> übergeben wird.
Zum Erhalt des Endpoint-Zugangs sowie einer gültigen Authentifizierung kontaktieren Sie bitte das Team von conQrete! 

⚠️ Hinweis: Der Token ist __vertraulich__ zu behandeln und darf nicht öffentlich geteilt oder im Frontend-Code hardcodiert werden.

Beispielaufruf mit <mark>curl</mark>:
```bash
curl -X POST "https://api.conqrete.tech/MODEL NAME/VERSION/" \
     -H "Authorization: Bearer YOUR_API_KEY" \
     -F "image=@path/to/your/image.jpg;type=image/jpeg"
```

Beispielaufruf mit <mark>Python</mark>:
```python
import requests

# Replace with your API details
API_URL = "https://api.conqrete.tech/MODEL NAME/VERSION/"
API_KEY = 'YOUR_API_KEY'

image_path = "path/to/your/image.jpg"

with open(image_path, "rb") as image_file:
    response = requests.post(
        API_URL,
        headers={'Authorization': f'Bearer {API_KEY}'},
        files={'image': ('image.jpg', image_file, 'image/jpeg')}, 
    )
# Print result
try:
    print(response.json())
except ValueError:
    print(response.text)
```

Wir empfehlen die Übermittlung von Bilddaten im **JPEG Format** in UltraXGA **(1600x1200, 4:3)** oder in Full-HD Auflösung **(1920 x 1080, 16:9)**.

<br><br><br>

## Hinweise zur Anwendung
Ein einziges **Foto vom Ausbreitmaß** genügt und die conQrete-API ermittelt automatisch eine Vielzahl an Frischbetoneigenschaften. 

Die einzige Anforderung an die Bildaufnahme ist, dass der Ausbreittisch vollständig im Bild enthalten sein muss – andernfalls kann eine Ausbreitmaß-Analyse nicht stattfinden. Achten Sie also darauf, dass alle vier Ecken des Tisches im Bild zu sehen sind, z.B. so:

![01](assets/01.png)

Als Ergebnis gibt die API eine Visualisierung der Ausbreitmaßanalyse sowie alle ermittelten Parameter der Frischbetoneigenschaften im JSON-Format zurück (Details siehe unten). 

Die Visualisierung kann für eine manuelle Plausibilitätsprüfung der Messung verwendet werden. Ein blauer Tischrahmen wird auf das Bild projiziert, welcher bei korrekt erfolgter Messung den Kanten des Ausbreittisches entsprechen sollte. Der Ausbreitkuchen wird eingefärbt und sollte dem Bereich entsprechen, welche für die Messung des Ausbreitmaßdurchmessers verwendet werden soll.

<br><br><br>

## Rückgabewerte und –format
Die REST-API gibt standardmäßig alle Antworten im **JSON-Format** zurück (JavaScript Object Notation). Unabhängig davon, ob eine Messung erfolgreich oder nicht-erfolgreich war, besteht jede API-Antwort aus folgenden Bestandteilen:

![02](assets/02.png)

> <small>Abbildung 1: Beispiele für die zurückgegebenen Antworten der REST-API im JSON Format. Links: Beispiel für eine erfolgreiche Messung. Rechts: Beispiel für eine nicht-erfolgreiche Messung.</small>

#### Rückgabefelder im Detail:

| Feld       | Typ     | Beschreibung | Beispiel |
|------------|---------|--------------|----------|
| <mark>response</mark>| <mark>object</mark> | Objekt welches die gesamte zurückgegebene Antwort der API enthält |  |
| <mark>created_at</mark> | <mark>string</mark> | ISO 8601-formatierter Datums- und Zeitstempel |  |
| <mark>result</mark>  | <mark>object</mark> | Objekt, welches die Auswerteergebnisse des Ausbreitmaßes enthält |  |
| <mark>success</mark> | <mark>boolean</mark> | Gibt an, ob eine Messung stattgefunden hat (<mark>true</mark>) oder nicht (<mark>false</mark>), z. B. wenn kein Ausbreittisch im Bild vorhanden ist. |  |
| <mark>msg</mark>     | <mark>string</mark>  | Statusmeldung mit Details zum Erfolg/Fehlern:<br><mark>200</mark> – Messung erfolgreich<br><mark>300</mark> – kein Ausbreittisch im Bild gefunden<br><mark>400</mark> – Ausbreittisch nicht vollständig sichtbar (eine oder mehrere Ecken fehlen)<br><mark>500</mark> – Ausbreittisch gefunden, aber kein Betonkuchen detektiert |  |
|<mark>durchmesser_a</mark>| <mark>float / null</mark> | Durchmesser des Ausbreitkuchen [mm] entlang der einen Tischkante | ![03](assets/03.png) |
|<mark>durchmesser_b</mark>| <mark>float / null</mark> | Durchmesser des Ausbreitkuchen [mm] entlang der anderen Tischkante | ![04](assets/04.png) |
|<mark>durchmesser</mark>| <mark>float / null</mark> | Gemittelter Durchmesser [mm] aus <mark>durchmesser_a</mark> und <mark>durchmesser_b</mark> |  |
|<mark>konsistenz</mark>| <mark>string (Enum) / null</mark> | Gibt die aus dem durchmesser ermittelte Konsistenzklasse an. Mögliche Werte: <mark>F1</mark>, <mark>F2</mark>, <mark>F3</mark>, <mark>F4</mark>, <mark>F5</mark> und <mark>F6</mark> |  |
|<mark>groesstkorn</mark>| <mark>int (Enum) / null</mark> | Gibt das aus dem Bild ermittelte Größtkorn [mm] an. Mögliche Werte: 8, 16, 32 |  |
|<mark>sieblinie</mark>| <mark>string (Enum) / null</mark> | Gibt eine grobe Einschätzung der Siebline **(Achtung: dient nur als Richtwert, mit Ungenauigkeiten behaftet)**. Mögliche Werte: <mark>A</mark>, <mark>AB</mark>, <mark>B</mark>, <mark>BC</mark>, <mark>C</mark> |  |
|<mark>shapeMetrics</mark>| <mark>object</mark> | Objekt mit alternativen Beschreibungen des Durchmessers und zusätzlichen Form-Parametern, die die Form und Ausprägung des Ausbreitkuchens beschreiben |  |
|<mark>.f_max</mark> | <mark>float / null</mark> | Maximaler Durchmesser [mm] der um den Ausbreitkuchen gelegten rotierten best-fit Bounding Box. Entspricht der längeren Seite der Box. Kann abweichen von <mark>durchmesser_a bzw</mark>. <mark>durchmesser_b</mark>, je stärker die Form des Ausbreitkuchens von einem perfekten Kreis abweicht und je stärker die Orientierung des Ausbreitkuchens zu den Seiten des Tisches verdreht ist |![05](assets/05.png)|
| <mark>.f_min</mark>| <mark>float / null</mark> | wie <mark>.f_max</mark>, allerdings bezogen auf die kürzere Seite der rotierten Bounding Box | ![06](assets/06.png) |
| <mark>durchmesser</mark> | <mark>float / null</mark> | Gemittelter Durchmesser [mm] aus <mark>durchmesser_a</mark> und <mark>durchmesser_b</mark> | |
| <mark>konsistenz</mark> | <mark>string (Enum) / null</mark> | Gibt die aus dem <mark>durchmesser</mark> ermittelte Konsistenzklasse an. Mögliche Werte: <mark>F1, F2, F3, F4, F5 und F6</mark> | |
|<mark>groesstkorn</mark> | <mark>int (Enum) / null</mark> |Gibt das aus dem Bild ermittelte Größtkorn [mm] an. Mögliche Werte: <mark>8, 16, 32</mark>||
|<mark>sieblinie</mark> | <mark>string (Enum) / null</mark> |	Gibt eine grobe Einschätzung der Siebline **(Achtung: dient nur als Richtwert, mit Ungenauigkeiten behaftet)**. Mögliche Werte: <mark>A, AB, B, BC, C</mark> ||
|<mark>shapeMetrics</mark>|	<mark>object</mark> |	Objekt mit alternativen Beschreibungen des Durchmessers und zusätzlichen Form-Parametern, die die Form und Ausprägung des Ausbreitkuchens beschreiben ||