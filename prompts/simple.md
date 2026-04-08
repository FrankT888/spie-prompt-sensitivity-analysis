# Condition 1: Simple Prompt

## System Message

Analyze the provided satellite image to count the total number of airplanes and classify each plane based on the ontology provided. For each aircraft, determine its class based on the provided ontology. Return your analysis in the specified JSON format with confidence scores for both count and individual class identifications.


Aircraft Classification Ontology (from Ontology Manager):
```json
[
  {
    "id": 0,
    "name": "Small Civil Transport/Utility",
    "description": "Large civil aircraft used for commercial transport operations",
    "characteristics": [
      "Large size",
      "Civil registration",
      "Commercial transport",
      "Passenger/cargo capacity"
    ],
    "tags": [],
    "hasReferenceImage": false
  },
  {
    "id": 1,
    "name": "Medium Civil Transport/Utility",
    "description": "Medium-sized civil aircraft used for regional transport operations",
    "characteristics": [
      "Medium size",
      "Civil registration",
      "Regional transport",
      "Short to medium range"
    ],
    "tags": [],
    "hasReferenceImage": false
  },
  {
    "id": 2,
    "name": "Large Civil Transport/Utility",
    "description": "Military aircraft designed for bombing missions and strategic attacks",
    "characteristics": [
      "Military registration",
      "Bomb carrying capability",
      "Long range",
      "Strategic missions"
    ],
    "tags": [],
    "hasReferenceImage": false
  },
  {
    "id": 3,
    "name": "Military Transport/Utility/AWAC",
    "description": "Military aircraft designed for air-to-air and air-to-ground combat",
    "characteristics": [
      "Military registration",
      "Combat role",
      "High performance",
      "Weapons systems"
    ],
    "tags": [],
    "hasReferenceImage": false
  },
  {
    "id": 4,
    "name": "Military Fighter/Interceptor/Attack",
    "description": "Military aircraft used for pilot training and instruction",
    "characteristics": [
      "Military registration",
      "Training role",
      "Dual controls",
      "Instruction capability"
    ],
    "tags": [],
    "hasReferenceImage": false
  }
]
```


IMPORTANT: You must respond with a valid JSON object in the following format:
```json
{
  "objects": [
    {
      "class": 0,
      "className": "Large Civil Transport",
      "confidence": 0.95
    }
  ],
  "count": 1,
  "countConfidence": 0.92
}
```


Where:
- "objects": array of detected aircraft with class, className, and confidence (0-1)
- "count": total number of aircraft detected
- "countConfidence": confidence score (0-1) for the overall count accuracy


Class IDs:
- 0: Small Civil Transport/Utility
- 1: Medium Civil Transport/Utility
- 2: Large Civil Transport/Utility
- 3: Military Transport/Utility/AWAC
- 4: Military Fighter/Interceptor/Attack

## User Message

`[Image attachment]` Final target image
