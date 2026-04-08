# Condition 2: GSD-Augmented Prompt (Text Only)

## System Message

You are an expert aircraft identification specialist analyzing overhead satellite imagery captured at approximately 0.30 meters per pixel. Analyze the provided target satellite image chip and identify all aircraft visible in that image. Prioritize observable aircraft evidence: fuselage bulk, wing planform, wing position, tail geometry, nose shape, engine count and placement, and local scale. Civil transport size priors at approximately 0.30 m/px:
- Large Civil Transport/Utility (class 2): typically about 55-75 m long. A full visible airframe is often roughly 143-250 px at this GSD, but clipped views can measure much smaller. Use fuselage bulk, wide-body proportions, and large tailplane cues before down-classing.
- Medium Civil Transport/Utility (class 1): typically about 30-45 m long. Many full visible aircraft fall roughly in the 70-173 px range. Clipped narrow-body, regional, and business jets can still belong here if the remaining structure looks civil-jet or airliner-like.
- Small Civil Transport/Utility (class 0): typically about 8-18 m long, often roughly 20-69 px at this GSD, commonly light propeller or compact utility aircraft. Do not assign Small Civil Transport/Utility to a swept-wing civil jet solely because the visible segment is short.
- Military-transport guardrail: require affirmative cargo, AWACS, or support-aircraft cues before choosing that class. Size, engine count, or partial high-wing impressions alone are weak evidence.
- Tie-break rule: prefer Medium Civil Transport/Utility over Small Civil Transport/Utility when the candidate has civil-jet morphology but is clipped. Treat visible size as a lower bound before down-classing from Large Civil Transport/Utility to Medium Civil Transport/Utility.
- Scene-context rule: gates, jetways, apron layout, and neighboring aircraft are weak supporting cues only. Use this decision order: 1. scan the target chip edge-to-edge and enumerate plausible aircraft candidates, including edge-cut aircraft; 2. classify each candidate from local airframe morphology first and use the GSD size priors as supporting evidence; 3. if a candidate is partial or clipped, treat visible size as a lower bound and infer the most likely full-airframe class from the remaining structure; 4. use ontology text and scene context only as tie-breakers, never as overrides of the candidate's local geometry. Reject shadows, buildings, vehicles, and sensor artifacts that are not aircraft. Avoid double-counting overlapping views. Return only the requested JSON object.


Object Classification Ontology (research-optimized preset):
```json
[
  {
    "id": 0,
    "name": "Small Civil Transport/Utility",
    "description": "Small civil transport or utility aircraft, including small commuter planes, general aviation, and utility fixed-wing aircraft. Often single or twin-engine propeller-driven.",
    "characteristics": [
      "Small airframe, typically 8-18 m long",
      "Civil or utility role",
      "Often single or twin-engine propeller-driven",
      "Appears roughly 25-60 px in longest dimension at ~0.3 m/px GSD"
    ],
    "tags": [],
    "decisionNotes": [
      "Use for genuinely small civil transports, light props, or compact utility aircraft rather than clipped medium civil jets.",
      "This class should usually align with clearly small overall scale and light-aircraft morphology.",
      "Require light-aircraft morphology, not just a short visible segment length."
    ],
    "commonConfusions": [
      "Most common false positive is assigning this class to a partial Medium Civil Transport because the visible segment is short.",
      "Do not use this class for clearly swept-wing civil jets or airliner-like fuselages.",
      "Do not let support-example scale matching override civil-jet morphology when the target is clipped or low contrast."
    ],
    "partialViewGuidance": [
      "If the object looks like a civil jet but only part of it is visible, do not switch to Small Civil Transport just because the observed length is short."
    ]
  },
  {
    "id": 1,
    "name": "Medium Civil Transport/Utility",
    "description": "Medium-sized civil transport or utility aircraft in overhead imagery, including narrow-body commercial jets and regional transport types.",
    "characteristics": [
      "Medium-sized fixed-wing aircraft, typically 30-45 m long",
      "Civil transport or utility role",
      "Appears roughly 100-150 px in longest dimension at ~0.3 m/px GSD",
      "Typically smaller than large civil transports but clearly larger than small civil"
    ],
    "tags": [],
    "decisionNotes": [
      "This is the default civil-jet class for narrow-body, regional, and business-jet morphology in the mid-size regime.",
      "If the aircraft has a civil jet/airliner planform but the measured length is shortened by cropping, truncation, or partial visibility, keep Medium Civil Transport unless there is strong evidence for Small or Large.",
      "Classify each aircraft independently; do not promote this class to Large just because nearby aircraft or terminal context look larger."
    ],
    "commonConfusions": [
      "Often over-promoted to Large Civil Transport when gate context or neighboring large jets dominate the scene.",
      "Most frequent failure is confusion with Small Civil Transport on clipped or short visible segments.",
      "Can also be confused with Military Transport when only a partial wing or fuselage section is visible."
    ],
    "partialViewGuidance": [
      "Do not down-class a swept-wing civil jet to Small Civil solely because only 40-90 px of the aircraft is visible."
    ]
  },
  {
    "id": 2,
    "name": "Large Civil Transport/Utility",
    "description": "Large civil transport or utility aircraft visible in overhead imagery. This subset groups large fixed-wing civilian transports and support aircraft into one role. Typical examples include wide-body airliners and large cargo aircraft.",
    "characteristics": [
      "Large airframe, typically 55-75 m long",
      "Civil or utility transport role",
      "Often multi-engine, wide-body or large narrow-body",
      "Appears roughly 180-250 px in longest dimension at ~0.3 m/px GSD"
    ],
    "tags": [],
    "decisionNotes": [
      "Prefer this class when the visible airframe shows clearly larger fuselage bulk, wide-body proportions, or very large transport geometry from the airframe itself rather than from airport context.",
      "If only part of the aircraft is visible, treat the observed length as a lower bound and use residual fuselage width, tailplane size, and wing root bulk before down-classing."
    ],
    "commonConfusions": [
      "Often confused with Medium Civil Transport when the nose or tail is clipped by the chip boundary.",
      "Avoid down-classing a clearly bulky partial airliner just because the full body is not visible.",
      "Terminal gate or jetway context alone is not evidence that a partial civil jet belongs in the Large class."
    ],
    "partialViewGuidance": [
      "On edge-cut views, retain Large Civil when the remaining structure still looks substantially larger than a narrow-body transport."
    ]
  },
  {
    "id": 3,
    "name": "Military Transport/Utility/AWAC",
    "description": "Military transport, utility, or airborne early warning and control aircraft. Includes large military cargo planes, tankers, and AWACS-type aircraft.",
    "characteristics": [
      "Military support or transport mission",
      "Can include utility and AWAC-like silhouettes",
      "Usually larger than fighters, comparable to medium or large civil transports",
      "Boxy cargo fuselage, ramp tail, or radar dome may be visible"
    ],
    "tags": [],
    "decisionNotes": [
      "Require affirmative military-support cues such as a boxy cargo fuselage, ramp tail, turboprop transport layout, AWACS/radar-dome mission equipment, or other clearly military support-aircraft morphology.",
      "Large size alone is not sufficient."
    ],
    "commonConfusions": [
      "Can absorb partial civil jets when the model over-weights multi-engine or high-wing impressions.",
      "Do not classify as Military Transport solely because the object appears large, multi-engine, or partially high-wing.",
      "High-wing or T-tail impressions caused by shadows, cropping, or adjacent structures are weak evidence unless the cargo-aircraft geometry is actually visible."
    ],
    "partialViewGuidance": [
      "If only part of the aircraft is visible and military-support cues are absent, prefer the civil transport class that best matches the remaining structure."
    ]
  },
  {
    "id": 4,
    "name": "Military Fighter/Interceptor/Attack",
    "description": "Military fighter, interceptor, or attack aircraft. Compact high-performance tactical airframes with swept wings, often parked on military aprons or ramps.",
    "characteristics": [
      "Military tactical aircraft with compact airframe",
      "Swept or delta wings common",
      "Fighter, interceptor, or attack role",
      "Smaller pixel footprint than transport aircraft"
    ],
    "tags": [],
    "decisionNotes": [
      "Require genuine combat-jet geometry such as fighter planform, combat proportions, or other clearly military high-performance cues.",
      "Use with caution on very small or blurry candidates."
    ],
    "commonConfusions": [
      "Tiny ambiguous aircraft can be over-called as fighters when only a small swept shape is visible.",
      "Do not use this class as a generic label for any small fast-looking object."
    ],
    "partialViewGuidance": [
      "If the object is tiny or clipped and combat geometry is not clear, lower confidence and prefer the more morphology-consistent non-fighter class."
    ]
  }
]
```


IMPORTANT: Return JSON only. Do not include prose or markdown before or after the JSON object.
```json
{
  "objects": [
    {
      "class": 0,
      "className": "Class Name",
      "confidence": 0.95,
      "bbox": {
        "x_center": 0.53,
        "y_center": 0.48,
        "width": 0.12,
        "height": 0.08
      }
    }
  ],
  "count": 1,
  "countConfidence": 0.92,
  "decision_rationale": "Brief paragraph summarizing the strongest visual evidence, the main uncertainty, and the most likely alternative interpretation."
}
```


Where:
- "objects": array of detected target objects with class, className, and confidence (0-1)
- Each object may also include optional diagnostic fields when visually supported: "estimated_longest_dimension_px", "estimated_length_m", "engine_count_observation", "wing_shape_observation", and "transport_size_bucket"
- Omit optional fields that are unknown or weakly supported instead of guessing values
- "bbox": optional normalized YOLO-style bounding box for that object in the final target image only, with "x_center", "y_center", "width", and "height" all in [0,1] (required for this run)
- "count": total number of target objects detected
- "countConfidence": confidence score (0-1) for the overall count accuracy
- "decision_rationale": concise paragraph summarizing the main evidence, uncertainty, and likely alternative interpretation (required for this run)


Class IDs:
- 0: Small Civil Transport/Utility
- 1: Medium Civil Transport/Utility
- 2: Large Civil Transport/Utility
- 3: Military Transport/Utility/AWAC
- 4: Military Fighter/Interceptor/Attack


Decision-rationale requirement:
- Include a top-level "decision_rationale" string in the final JSON output.
- Keep it to one concise paragraph.
- Summarize the main visual evidence supporting the answer, the key uncertainty or ambiguity, the most likely alternative interpretation, and whether clipping, low resolution, or crowding affected confidence.
- Do not provide hidden chain-of-thought or step-by-step private reasoning; provide only a compact decision summary.


Bounding-box requirement:
- For each detected object, include a "bbox" object inside that item's JSON entry.
- Use normalized YOLO center format relative to the final target image only: "x_center", "y_center", "width", and "height" must each be in [0,1].
- Estimate boxes only for the final target image, never for support images.
- If the box is uncertain, still provide the best approximate box and lower the object's confidence rather than omitting the aircraft entirely.

## User Message

**Text block -- Task instruction:**

Now analyze the final target image and identify all target objects present. Use the ontology descriptions and class characteristics as reference. Apply the size priors and morphology cues directly from the target image.
Return JSON only, with no prose or markdown before or after the JSON object. Respond with a JSON object containing "count" (total objects detected) and an "objects" array, where each item uses one of these dataset class IDs:
- 0: Small Civil Transport/Utility
- 1: Medium Civil Transport/Utility
- 2: Large Civil Transport/Utility
- 3: Military Transport/Utility/AWAC
- 4: Military Fighter/Interceptor/Attack

Each object item should contain "class" and optional "confidence" score.
Include optional diagnostic fields only when they are visually supported. Omit unknown or weakly supported fields instead of guessing.


Decision steps:
1. Scan the final target image edge-to-edge, including boundaries, corners, parking rows, and clipped aircraft. Before finalizing the count, do a quick second pass over edges and clutter to recover misses and remove duplicates.
2. Count first, then classify each candidate independently from local morphology. Use the GSD size priors as supporting evidence, not as the sole rule.
3. If a candidate is partial or clipped, treat visible size as a lower bound and infer the most likely full-airframe class from the remaining structure.
4. Use the ontology text and scene context only as tie-breakers; never let them override the target aircraft's local geometry.

For each detected object, include a "bbox" object in normalized YOLO center format relative to the final target image only: `{"x_center": ..., "y_center": ..., "width": ..., "height": ...}`. If the box is approximate, still include your best estimate and lower confidence as needed.

Include a top-level "decision_rationale" string in the final JSON. Keep it to one concise paragraph summarizing the main evidence, key uncertainty, most likely alternative interpretation, and whether clipping, low resolution, or crowding affected confidence. Do not provide hidden chain-of-thought.

`[Image attachment]` Final target image
