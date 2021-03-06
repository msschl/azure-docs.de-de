---
title: Erkennen domänenspezifischer Inhalte – maschinelles Sehen
titleSuffix: Azure Cognitive Services
description: Hier erfahren Sie, wie Sie eine Bildkategorisierungsdomäne angeben, um ausführlichere Informationen zu einem Bild zurückzugeben.
services: cognitive-services
author: PatrickFarley
manager: nitinme
ms.service: cognitive-services
ms.subservice: computer-vision
ms.topic: conceptual
ms.date: 02/08/2019
ms.author: pafarley
ms.custom: seodec18
ms.openlocfilehash: e4b64e00f71768a8821c83a73b019f77089e1b3a
ms.sourcegitcommit: bf509e05e4b1dc5553b4483dfcc2221055fa80f2
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 04/22/2019
ms.locfileid: "59994485"
---
# <a name="detect-domain-specific-content"></a>Erkennen domänenspezifischer Inhalte

Zusätzlich zur Kennzeichnung und allgemeinen Kategorisierung wird beim maschinellen Sehen auch die weiter gehende domänenspezifische Analyse unterstützt. Hierfür werden Modelle verwendet, die anhand von speziellen Daten trainiert wurden.

Es gibt zwei Möglichkeiten für die Verwendung von domänenspezifischen Modellen: allein (bereichsbezogene Analyse) oder als Erweiterung der Kategorisierungsfunktion.

### <a name="scoped-analysis"></a>Bereichsbezogene Analyse

Sie können ein Bild analysieren, indem Sie nur das domänenspezifische Modell nutzen. Rufen Sie hierfür die API [Models/\<model\>/Analyze](https://westus.dev.cognitive.microsoft.com/docs/services/5adf991815e1060e6355ad44/operations/56f91f2e778daf14a499e200) auf.

Hier ist eine JSON-Beispielantwort angegeben, die von der API **models/celebrities/analyze** für das jeweilige Bild zurückgegeben wird:

![Satya Nadella (stehend, lächelnd)](./images/satya.jpeg)

```json
{
  "result": {
    "celebrities": [{
      "faceRectangle": {
        "top": 391,
        "left": 318,
        "width": 184,
        "height": 184
      },
      "name": "Satya Nadella",
      "confidence": 0.99999856948852539
    }]
  },
  "requestId": "8217262a-1a90-4498-a242-68376a4b956b",
  "metadata": {
    "width": 800,
    "height": 1200,
    "format": "Jpeg"
  }
}
```

### <a name="enhanced-categorization-analysis"></a>Verbesserte Kategorisierungsanalyse

Sie können domänenspezifische Modelle auch verwenden, um die allgemeine Bildanalyse zu erweitern. Dies ist im Rahmen der [allgemeinen Kategorisierung](concept-categorizing-images.md) möglich, indem domänenspezifische Modelle im Parameter *details* des [Analyze](https://westus.dev.cognitive.microsoft.com/docs/services/5adf991815e1060e6355ad44/operations/56f91f2e778daf14a499e1fa)-API-Aufrufs angegeben werden.

In diesem Fall wird zuerst die Klassifizierung der 86-Kategorien-Taxonomie aufgerufen. Wenn erkannte Kategorien über ein passendes domänenspezifisches Modell verfügen, wird das Bild auch über dieses Modell übergeben, und die Ergebnisse werden hinzugefügt.

Mit der folgenden JSON-Antwort wird veranschaulicht, wie die domänenspezifische Analyse als `detail`-Knoten in eine umfassendere Kategorisierungsanalyse eingebunden werden kann.

```json
"categories":[
  {
    "name":"abstract_",
    "score":0.00390625
  },
  {
    "name":"people_",
    "score":0.83984375,
    "detail":{
      "celebrities":[
        {
          "name":"Satya Nadella",
          "faceRectangle":{
            "left":597,
            "top":162,
            "width":248,
            "height":248
          },
          "confidence":0.999028444
        }
      ],
      "landmarks":[
        {
          "name":"Forbidden City",
          "confidence":0.9978346
        }
      ]
    }
  }
]
```

## <a name="list-the-domain-specific-models"></a>Auflisten der domänenspezifischen Modelle

Derzeit werden für das maschinelle Sehen die folgenden domänenspezifischen Modelle unterstützt:

| NAME | BESCHREIBUNG |
|------|-------------|
| Prominente | Erkennung berühmter Personen, die für Bilder unterstützt wird, die in die Kategorie `people_` klassifiziert werden |
| Wahrzeichen | Erkennung von Wahrzeichen, die für Bilder unterstützt wird, die in die Kategorien `outdoor_` oder `building_` klassifiziert werden |

Durch das Aufrufen der [Models](https://westus.dev.cognitive.microsoft.com/docs/services/5adf991815e1060e6355ad44/operations/56f91f2e778daf14a499e1fd)-API werden diese Informationen zusammen mit den Kategorien zurückgegeben, für die die einzelnen Modelle gelten können:

```json
{
  "models":[
    {
      "name":"celebrities",
      "categories":[
        "people_",
        "人_",
        "pessoas_",
        "gente_"
      ]
    },
    {
      "name":"landmarks",
      "categories":[
        "outdoor_",
        "户外_",
        "屋外_",
        "aoarlivre_",
        "alairelibre_",
        "building_",
        "建筑_",
        "建物_",
        "edifício_"
      ]
    }
  ]
}
```

## <a name="next-steps"></a>Nächste Schritte

Erfahren Sie mehr über das [Kategorisieren von Bildern](concept-categorizing-images.md).
