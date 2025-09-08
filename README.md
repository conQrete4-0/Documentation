# Qflow-API: Benutzerdokumentation — _powered by [conQrete 4.0](https://conqrete.tech)_

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
|<mark>konsistenz</mark>| <mark>string (Enum) / null</mark> | Gibt die aus dem <mark>durchmesser</mark> ermittelte Konsistenzklasse an. Mögliche Werte: <mark>F1</mark>, <mark>F2</mark>, <mark>F3</mark>, <mark>F4</mark>, <mark>F5</mark> und <mark>F6</mark> |  |
|<mark>groesstkorn</mark>| <mark>int (Enum) / null</mark> | Gibt das aus dem Bild ermittelte Größtkorn [mm] an. Mögliche Werte: <mark>8, 16, 32</mark> |  |
|<mark>sieblinie</mark>| <mark>string (Enum) / null</mark> | Gibt eine grobe Einschätzung der Siebline **(Achtung: dient nur als Richtwert, mit Ungenauigkeiten behaftet)**. Mögliche Werte: <mark>A</mark>, <mark>AB</mark>, <mark>B</mark>, <mark>BC</mark>, <mark>C</mark> |  |
|<mark>shapeMetrics</mark>| <mark>object</mark> | Objekt mit alternativen Beschreibungen des Durchmessers und zusätzlichen Form-Parametern, die die Form und Ausprägung des Ausbreitkuchens beschreiben |  |
|<mark>.f_max</mark> | <mark>float / null</mark> | Maximaler Durchmesser [mm] der um den Ausbreitkuchen gelegten rotierten best-fit Bounding Box. Entspricht der längeren Seite der Box. Kann abweichen von <mark>durchmesser_a bzw</mark>. <mark>durchmesser_b</mark>, je stärker die Form des Ausbreitkuchens von einem perfekten Kreis abweicht und je stärker die Orientierung des Ausbreitkuchens zu den Seiten des Tisches verdreht ist |![05](assets/05.png)|
| <mark>.f_min</mark>| <mark>float / null</mark> | wie <mark>.f_max</mark>, allerdings bezogen auf die kürzere Seite der rotierten Bounding Box | ![06](assets/06.png) |
| <mark>.x_A</mark> | <mark>float / null</mark> | Kreisäquivalenter Durchmesser des Ausbreitkuchens. Definiert als der Durchmesser eines Kreises mit derselben Fläche A wie der Ausbreitkuchen: <br /><br />$$x_A = \sqrt{\frac{4A}{\pi}}$$ | ![09](/assets/09.png) |
| <mark>.circularity</mark>	| <mark>float / null</mark> |Maß für die **Kreisähnlichkeit** des Segments. Der Wert liegt zwischen <mark>0 und 1, wobei 1</mark> einem perfekten Kreis entspricht. Der Wert berücksichtigt sowohl die Fläche (A) als auch die Glattheit des Umfangs (U) des Ausbreitkuchens: <br /><br /> $$circularity = \frac{4 \pi A}{U^2}$$ | |
| <mark>.compactness</mark>	| <mark>float / null</mark>	| Maß für die **Kompaktheit** des Ausbreitkuchens. Wertebereich von <mark>0 bis 1, wobei 1</mark> einem perfekten Kreis entspricht. Bewertet die Kreisähnlichkeit basierend auf der Gesamtform, nicht nur dem Umfang: <br /><br /> $$compactness = \frac{\sqrt{\tfrac{4A}{\pi}}}{f_{max}} = \frac{x_A}{f_{max}}$$| | |
| <mark>.solidity</mark> | <mark>float / null</mark> | Maß für die **Konkavität** (Eindellungen) des Ausbreitkuchens. Definiert als Verhältnis zwischen der Kuchenfläche (A) und der Fläche der konvexen Hülle (A_c). Wertebereich: <mark>0 (stark eingedellt) bis 1 (keine Einbuchtungen, wie z. B. bei einem Kreis) </mark>: <br /><br /> $$solidity = \frac{A}{A_c}$$ | ![10](/assets/10.png)|
| <mark>.convexity</mark> | <mark>float / null</mark> | Maß für die **Konvexität** des Ausbreitkuchens. Definiert als das Verhältnis zwischen dem Umfang der konvexen Hülle (U_c) und dem tatsächlichen Umfang des Ausbreitkuchens (U). Wertebereich: 0 bis 1, wobei 1 einer perfekten konvexen Form (z. B. Kreis) entspricht:<br /><br /> $$convexity = \frac{U_c}{U}$$| |
| <mark>.extent</mark> | <mark>float / null</mark> | Maß für die **Raumausfüllung** des Ausbreitkuchens. Definiert als das Verhältnis der Ausbreitkuchenfläche (A) zur Fläche der rotierten best-fit Bounding Box (siehe <mark>f_max und f_min</mark>). Wertebereich: 0 bis 1. <br /><br /> $$extent = \frac{A}{f_{max} \cdot f_{min}}$$| |
| <mark>.eccentricity</mark> | <mark>float / null</mark> | Maß für die **Exzentrizität** des Ausbreitkuchens. Beschreibt, wie „elliptisch“ oder „langgestreckt“ der Kuchen ist. Wertebereich: <mark>0 bis 1, wobei 0</mark> einem perfekten Kreis entspricht und 1 einer extrem gestreckten Ellipse (Strichform). ||
| <mark>.irregularity</mark> | <mark>float / null</mark> | Maß für die **Unregelmäßigkeit** der Ausbreitkuchenform. Wertebereich: <mark>0 (perfekte, glatte, konvexe Form) bis 1</mark> (maximal unregelmäßige Form mit rauem Rand und Einbuchtungen). Berechnet als <br /><br /> $$irregularity = 1 - (circularity \times solidity)$$ ||
| <mark>.radial_var</mark> | <mark>float / null</mark> | Maß für die **radiale Formabweichung** – berechnet als Verhältnis der Varianz der Abstände der Randpunkte zum Massenschwerpunkt zur mittleren Entfernung (<mark>radial_mean_diameter</mark>). Gibt an, wie stark die Randabstände streuen. ||
| <mark>.radial_mean_diameter</mark> | <mark>float / null</mark> | **Mittlere Entfernung** aller Randpunkte zum Massenschwerpunkt des Ausbreitkuchens. Gibt einen durchschnittlichen „Radius“ an – eine Art mittlerer Formdurchmesser. |![11](/assets/11.png)|
| <mark>processed_image</mark> | <mark>string</mark>(Base64) | Kodiertes **Rückgabebild** (Base64-kodiert als UTF-8-String), enthält die Visualisierung der KI-basierten Ausbreitmaßanalyse. Kann für einen manuellen Plausibilitäts-Check genutzt werden. Im Fall einer nicht erfolgreichen Messung wird das originale Bild zurückgegeben | ![12](/assets/12.png)|
|<mark>thumbnail</mark> | <mark>string</mark>(Base64) / null | Kodiertes **Rückgabebld** (Base64-kodiert als UTF-8-String), enthält den perspektivisch korrigierten Ausschnitt des Ausbreittisches inkl. Visualisierung der Messung | ![13](/assets/13.png)|
|<mark>Thumbnail_unmasked</mark> | <mark>string</mark>(Base64) / null |	Kodiertes **Rückgabebld** (Base64-kodiert als UTF-8-String), wie <mark>thumbnail</mark>, aber ohne eingefärbten Ausbreitkuchen | ![14](/assets/14.png)|
| <mark>request_path</mark> | <mark>string</mark> |	Genutzte Endpoint-URL ||
| <mark>request_size</mark> | <mark>int</mark> | Datengröße der übermittelten Datei ||
| <mark>request_method</mark> |	<mark>string</mark> | Genutze Request Methode der REST-API ||
| <mark>response_time</mark> | <mark>float</mark> |	Dauer der Response in [s] ||

---
![Logo](assets/conQrete-signet.jpg)
<br>
conqrete.tech
<br>
mail@conqrete.tech
