# ChatGPT Running coach from Strava data

Guide for configuration of a custom GPT for fetching and analysing Strava data, then acting as a running coach based on that data.

Most of the setup steps are taken from [this Reddit post](https://www.reddit.com/r/Strava/comments/1n2fmy8/tutorial_work_with_strava_data_in_chatgpt_plus/?utm_source=share&utm_medium=web3x&utm_name=web3xcss&utm_term=1&utm_content=share_button)

## Pre-requisites

- ChatGTP account with a paid plan that allows creation of custom GPTs (currently starting at the "plus" tier: https://chatgpt.com/pricing/)
- Strava account with running activity data
- Optional: Garmin Connect account or similar for creating running plan workouts

## 1. Create a Strava API app

This is required for giving the GPT access to your Strava data
1. Go to https://www.strava.com/settings/api (login if prompted) and create a new app. Strava has a [beginner guide](https://developers.strava.com/docs/getting-started/), but the things you need to fill in are:
   - Application name: e.g. "ChatGPT analysis"
   - Category: Select something appropriate (e.g. Performance analysis)
   - Club: (optional, leave it blank or input your club)
   - Website: `chat.openai.com`
   - Application description: e.g. "ChatGPT Running coach from Strava data". Consider adding a link to this guide as a reminder for your future self.
   - Authorization Callback Domain: `chat.openai.com`
2. After saving the new app, copy the Client ID and Client Secret as you will need them for configuring the GPT

## 2. Create a custom GPT

The custom GPT is what allows ChatGPT to access your strava data, understand the data format, and give instructions to your AI Running Coach.
1. Login the ChatGPT and go to https://chatgpt.com/gpts
2. Click **+ Create** in the top right corner to add your own, new GPT
3. Fill in the following fields:
    - Name: e.g. "Running Coach and Strava analysis"
    - Description: "GPT for fetching and analysing Strava data, then acting as a running coach based on that data". Consider adding a link to this guide as a reminder for your future self.
    - Instructions: Copy/paste from the Instructions section below
    - Conversation starters: Optional, refer to section below for some suggestions
    - Knowledge: Optional, upload documents of things like training guides, plans, methodologies and scientific studies you want the "coach" to use
    - Recommended Model: Probably best to pick the latest general model, but worth experimenting with other models too
    - Capabilities: Tick all 4 options
4. Click **Create new action** to set up the Strava integration
5. Change Authentication to **OAuth**
   - Client ID: Paste from step 1
   - Client Secret: Paste form step 1
   - Authorization URL: `https://www.strava.com/oauth/authorize`
   - Token URL: `https://www.strava.com/oauth/token`
   - Scope: `read,read_all,profile:read_all,activity:read_all`
   - Token Exchange Method: Default (POST request)
   - Save the Authorization config
6. Schema: Copy/paste from Schema section below
7. Configuration complete! Test it out by clicking the preview button and ask it to fetch the latest Strava activity and analyse it.

## Instructions

This is where you can provide instructions to the GPT on how to use the Strava integration and data, and how the Coach should behave. Plus anything else you want to be consistent for every time you use the GPT. This is obviously very subjective, and will change depending on how YOU are using the GTP. That said, here are the instructions I'm currently using:

```
Fetch activity data from Strava as required to analyze training load, performance and progression. Tailor running training plans based on past activity data and defined future goals. Be strict in training suggestions and reasoning and adhere to authoritative, well-known and science based sources. Act like an experienced, analytical and science based running coach. Do not be overly encouraging or eager to please. Do not be overly verbose in replies.

When asked to analyze training or an activity, first determine the minimum Strava data required to answer the question and avoid unnecessary API calls.

Strava reports all dates and times in UTC. Always convert to/from the user’s local time zone unless otherwise specified. For week or date-range queries in local time, apply a ±1 day UTC buffer to avoid missing activities at timezone boundaries.

*Data Retrieval Priority*
1. Use getAthleteActivities to identify relevant activities.
2. Use getActivityById for overall summary metrics and split summaries (splits_metric, splits_standard, segment efforts).
3. Use getActivityLaps when isolating structured workout segments (intervals, threshold blocks, warm-up, recovery, cool-down). Laps are preferred over splits when available.
4. Use getActivityStreams only when:
  a. Laps or splits are unavailable or insufficient.
  b. Time-series analysis is required (HR drift, pace variability, power trends, cadence dynamics).
  c. Reconstruction of interval blocks is required.
Only escalate to streams when necessary.

*Activity Analysis Rules*
Strava activity-level averages are often misleading due to warm-up, recovery, and cool-down periods.
When analyzing quality sessions:
 - Identify and isolate the “work” portion of the activity.
 - Exclude warm-up, recovery, and cool-down segments from performance metrics.
 - Prefer lap data to identify structured segments.
 - If laps are not structured, infer work segments from HR, pace, and power patterns.
When analyzing splits:
 - Do not assume splits align with workout structure.
 - Do not treat km splits as interval reps unless clearly structured that way.

*Metric Selection*
 - Do not assume which metrics matter. Only compute and report metrics relevant to the user’s question. Clearly state any assumptions used to isolate work segments or classify workout type.
 - If expected fields (e.g., splits) are missing, fall back to laps. If laps are unavailable or insufficient, fall back to streams and reconstruct required metrics.

*Activity Type Inference*
Strava activity titles are often generic (e.g., “Morning Run”). To infer workout type:
1. First review summary metrics (duration, pace, HR, elevation).
2. If ambiguous, review laps.
3. If still ambiguous, analyze streams to detect interval patterns or intensity blocks.
Do not rely solely on the activity name.

*Weekly training plans*
Weekly training plans should adhere to longer term plans (race goal, duration, training block phase) but also take into account training responses from previous weeks and user reported factors such as injuries, fatigue, sickness and life events.
The weekly plans should be presented in a table format that makes it easy to copy/paste into Garmin Connect workout builder. Column headings are:
Day (Day of the week)
Workout (name of the workout, should contain activity type, duration, interval number and duration if applicable, target as HR or Pace depending on activity type)
Duration (total duration in minutes)
Garmin Segments (Summarised outline of activity segments, abbreviated segment types, duration and target where applicable)
Notes (Brief additional notes, tips and explanations for the activity, if applicable)

Example weekly training plan (as csv):
Day,Workout,Duration,Garmin Segments,Notes
Mon,Rest,—,—,Mobility optional
Tue,Threshold 58 (4×8),58 min,10 WU → 4×(8:00 HR 150–155 / 2:00 easy ≤140) → 10 CD,Controlled
Wed,Aerobic 60 Z2,60 min,10 WU → 45 Z2 (≤140) → 5 CD,Steady aerobic
Thu,MP 67 (3×12 @5:35),67 min,10 WU → 3×(12:00 @ 5:35/km / 3:00 easy) → 10 CD,Pace target
Fri,Rest,—,—,Full rest
Sat,Easy 65 + Strides,65 min,10 WU → 10 Z2 → 6×(0:20 fast / 1:40 easy) → 29 Z2 → 5 CD,Relaxed-fast strides
Sun,Long 160 w/20 MP,160 min,10 WU → 125 Z2 (≤140) → 20:00 @ 5:35/km → 5 CD,MP under fatigue
```

## Conversation starters

These are "short cuts' that show up as a button in the chat for common, recurring activities. Examples I've been using:

- Fetch the latest Strava activity and analyse it
- Fetch last weeks Strava activities, analyse it and generate next weeks training plan for Garmin connect
- Help me define a long term training plan based on upcoming running races and goals as well as Strava data for the last year



## Schema

This tells the GPT what Strava data is available, how it's formatted and how to get it.

<details>
<summary>Click to expand Schema details</summary>

```JSON
{
  "openapi": "3.1.0",
  "info": {
    "title": "Strava API",
    "description": "OpenAPI specification for accessing Strava fitness data via OAuth2. Configured for passthrough responses to avoid schema trimming and enable flexible downstream analysis.",
    "version": "v1.1.1"
  },
  "servers": [
    {
      "url": "https://www.strava.com/api/v3"
    }
  ],
  "paths": {
    "/athlete": {
      "get": {
        "operationId": "getAuthenticatedAthlete",
        "summary": "Get the currently authenticated athlete",
        "security": [
          {
            "stravaOAuth": [
              "profile:read_all"
            ]
          }
        ],
        "responses": {
          "200": {
            "description": "Returns the authenticated athlete",
            "content": {
              "application/json": {
                "schema": {
                  "$ref": "#/components/schemas/Athlete"
                }
              }
            }
          }
        }
      }
    },
    "/athlete/zones": {
      "get": {
        "operationId": "getAthleteZones",
        "summary": "Get heart rate and power zones of the athlete",
        "security": [
          {
            "stravaOAuth": [
              "profile:read_all"
            ]
          }
        ],
        "responses": {
          "200": {
            "description": "Athlete zones data (passthrough)",
            "content": {
              "application/json": {
                "schema": {
                  "type": "object",
                  "properties": {},
                  "additionalProperties": true
                }
              }
            }
          }
        }
      }
    },
    "/athletes/{id}/stats": {
      "get": {
        "operationId": "getAthleteStats",
        "summary": "Get aggregate stats of an athlete",
        "security": [
          {
            "stravaOAuth": [
              "profile:read_all"
            ]
          }
        ],
        "parameters": [
          {
            "name": "id",
            "in": "path",
            "required": true,
            "description": "The ID of the athlete",
            "schema": {
              "type": "integer"
            }
          }
        ],
        "responses": {
          "200": {
            "description": "Aggregate statistics for the athlete (passthrough)",
            "content": {
              "application/json": {
                "schema": {
                  "type": "object",
                  "properties": {},
                  "additionalProperties": true
                }
              }
            }
          }
        }
      }
    },
    "/athlete/activities": {
      "get": {
        "operationId": "getAthleteActivities",
        "summary": "List activities of the authenticated athlete",
        "security": [
          {
            "stravaOAuth": [
              "activity:read_all"
            ]
          }
        ],
        "parameters": [
          {
            "name": "before",
            "in": "query",
            "description": "Return activities before this UNIX timestamp",
            "schema": {
              "type": "integer"
            }
          },
          {
            "name": "after",
            "in": "query",
            "description": "Return activities after this UNIX timestamp",
            "schema": {
              "type": "integer"
            }
          },
          {
            "name": "page",
            "in": "query",
            "description": "Page number",
            "schema": {
              "type": "integer"
            }
          },
          {
            "name": "per_page",
            "in": "query",
            "description": "Number of items per page",
            "schema": {
              "type": "integer"
            }
          }
        ],
        "responses": {
          "200": {
            "description": "List of activities (untrimmed ActivitySummary passthrough)",
            "content": {
              "application/json": {
                "schema": {
                  "type": "array",
                  "items": {
                    "$ref": "#/components/schemas/ActivitySummary"
                  }
                }
              }
            }
          }
        }
      }
    },
    "/activities/{id}": {
      "get": {
        "operationId": "getActivityById",
        "summary": "Get a specific activity by ID (full ActivityDetail passthrough)",
        "security": [
          {
            "stravaOAuth": [
              "activity:read_all"
            ]
          }
        ],
        "parameters": [
          {
            "name": "id",
            "in": "path",
            "required": true,
            "description": "The ID of the activity",
            "schema": {
              "type": "integer"
            }
          },
          {
            "name": "include_all_efforts",
            "in": "query",
            "description": "Include all segment efforts",
            "schema": {
              "type": "boolean"
            }
          }
        ],
        "responses": {
          "200": {
            "description": "Activity details (passthrough; includes splits_metric/splits_standard when Strava provides them)",
            "content": {
              "application/json": {
                "schema": {
                  "$ref": "#/components/schemas/ActivityDetail"
                }
              }
            }
          }
        }
      }
    },
    "/activities/{id}/laps": {
      "get": {
        "operationId": "getActivityLaps",
        "summary": "Get laps for a specific activity",
        "security": [
          {
            "stravaOAuth": [
              "activity:read_all"
            ]
          }
        ],
        "parameters": [
          {
            "name": "id",
            "in": "path",
            "required": true,
            "description": "The ID of the activity",
            "schema": {
              "type": "integer"
            }
          }
        ],
        "responses": {
          "200": {
            "description": "Activity laps (passthrough)",
            "content": {
              "application/json": {
                "schema": {
                  "type": "array",
                  "items": {
                    "type": "object",
                    "properties": {},
                    "additionalProperties": true
                  }
                }
              }
            }
          }
        }
      }
    },
    "/activities/{id}/streams": {
      "get": {
        "operationId": "getActivityStreams",
        "summary": "Get detailed streams for an activity (GPS, HR, Power, etc.)",
        "security": [
          {
            "stravaOAuth": [
              "activity:read_all"
            ]
          }
        ],
        "parameters": [
          {
            "name": "id",
            "in": "path",
            "required": true,
            "description": "The ID of the activity",
            "schema": {
              "type": "integer"
            }
          },
          {
            "name": "keys",
            "in": "query",
            "description": "Requested stream keys (e.g., time, latlng, heartrate, watts)",
            "schema": {
              "type": "string"
            }
          },
          {
            "name": "key_by_type",
            "in": "query",
            "description": "If true, stream types will be keys in the response",
            "schema": {
              "type": "boolean"
            }
          }
        ],
        "responses": {
          "200": {
            "description": "Activity streams data (passthrough; supports both array and key_by_type object responses)",
            "content": {
              "application/json": {
                "schema": {
                  "oneOf": [
                    {
                      "type": "array",
                      "items": {
                        "type": "object",
                        "properties": {},
                        "additionalProperties": true
                      }
                    },
                    {
                      "type": "object",
                      "properties": {},
                      "additionalProperties": true
                    }
                  ]
                }
              }
            }
          }
        }
      }
    },
    "/activities/{id}/zones": {
      "get": {
        "operationId": "getActivityZones",
        "summary": "Get zones distribution for an activity",
        "security": [
          {
            "stravaOAuth": [
              "activity:read_all"
            ]
          }
        ],
        "parameters": [
          {
            "name": "id",
            "in": "path",
            "required": true,
            "description": "The ID of the activity",
            "schema": {
              "type": "integer"
            }
          }
        ],
        "responses": {
          "200": {
            "description": "Activity zones data (passthrough)",
            "content": {
              "application/json": {
                "schema": {
                  "type": "object",
                  "properties": {},
                  "additionalProperties": true
                }
              }
            }
          }
        }
      }
    },
    "/routes/{id}": {
      "get": {
        "operationId": "getRouteById",
        "summary": "Get details of a route",
        "security": [
          {
            "stravaOAuth": [
              "read"
            ]
          }
        ],
        "parameters": [
          {
            "name": "id",
            "in": "path",
            "required": true,
            "description": "The ID of the route",
            "schema": {
              "type": "integer"
            }
          }
        ],
        "responses": {
          "200": {
            "description": "Route details (passthrough)",
            "content": {
              "application/json": {
                "schema": {
                  "type": "object",
                  "properties": {},
                  "additionalProperties": true
                }
              }
            }
          }
        }
      }
    },
    "/segments/{id}": {
      "get": {
        "operationId": "getSegmentById",
        "summary": "Get details of a segment",
        "security": [
          {
            "stravaOAuth": [
              "read"
            ]
          }
        ],
        "parameters": [
          {
            "name": "id",
            "in": "path",
            "required": true,
            "description": "The ID of the segment",
            "schema": {
              "type": "integer"
            }
          }
        ],
        "responses": {
          "200": {
            "description": "Segment details (passthrough)",
            "content": {
              "application/json": {
                "schema": {
                  "type": "object",
                  "properties": {},
                  "additionalProperties": true
                }
              }
            }
          }
        }
      }
    },
    "/segment_efforts/{id}": {
      "get": {
        "operationId": "getSegmentEffortById",
        "summary": "Get details of a segment effort",
        "security": [
          {
            "stravaOAuth": [
              "activity:read_all"
            ]
          }
        ],
        "parameters": [
          {
            "name": "id",
            "in": "path",
            "required": true,
            "description": "The ID of the segment effort",
            "schema": {
              "type": "integer"
            }
          }
        ],
        "responses": {
          "200": {
            "description": "Segment effort details (passthrough)",
            "content": {
              "application/json": {
                "schema": {
                  "type": "object",
                  "properties": {},
                  "additionalProperties": true
                }
              }
            }
          }
        }
      }
    },
    "/segment_efforts/{id}/streams": {
      "get": {
        "operationId": "getSegmentEffortStreams",
        "summary": "Get detailed streams for a segment effort",
        "security": [
          {
            "stravaOAuth": [
              "activity:read_all"
            ]
          }
        ],
        "parameters": [
          {
            "name": "id",
            "in": "path",
            "required": true,
            "description": "The ID of the segment effort",
            "schema": {
              "type": "integer"
            }
          },
          {
            "name": "keys",
            "in": "query",
            "description": "Requested stream keys",
            "schema": {
              "type": "string"
            }
          }
        ],
        "responses": {
          "200": {
            "description": "Segment effort streams (passthrough)",
            "content": {
              "application/json": {
                "schema": {
                  "oneOf": [
                    {
                      "type": "array",
                      "items": {
                        "type": "object",
                        "properties": {},
                        "additionalProperties": true
                      }
                    },
                    {
                      "type": "object",
                      "properties": {},
                      "additionalProperties": true
                    }
                  ]
                }
              }
            }
          }
        }
      }
    }
  },
  "components": {
    "schemas": {
      "Athlete": {
        "type": "object",
        "additionalProperties": true,
        "properties": {
          "id": {
            "type": "integer"
          },
          "username": {
            "type": "string"
          },
          "firstname": {
            "type": "string"
          },
          "lastname": {
            "type": "string"
          },
          "city": {
            "type": "string"
          },
          "country": {
            "type": "string"
          },
          "sex": {
            "type": "string"
          },
          "created_at": {
            "type": "string",
            "format": "date-time"
          }
        }
      },
      "ActivitySummary": {
        "type": "object",
        "additionalProperties": true,
        "properties": {
          "id": {
            "type": "integer"
          },
          "name": {
            "type": "string"
          },
          "distance": {
            "type": "number"
          },
          "moving_time": {
            "type": "integer"
          },
          "elapsed_time": {
            "type": "integer"
          },
          "type": {
            "type": "string"
          }
        }
      },
      "ActivityDetail": {
        "type": "object",
        "additionalProperties": true,
        "properties": {
          "id": {
            "type": "integer"
          },
          "name": {
            "type": "string"
          },
          "description": {
            "type": "string"
          },
          "distance": {
            "type": "number"
          },
          "moving_time": {
            "type": "integer"
          },
          "elapsed_time": {
            "type": "integer"
          },
          "total_elevation_gain": {
            "type": "number"
          },
          "type": {
            "type": "string"
          }
        }
      }
    },
    "securitySchemes": {
      "stravaOAuth": {
        "type": "oauth2",
        "flows": {
          "authorizationCode": {
            "authorizationUrl": "https://www.strava.com/oauth/authorize",
            "tokenUrl": "https://www.strava.com/oauth/token",
            "scopes": {
              "read": "Read public segments and routes",
              "read_all": "Read all segments and routes",
              "profile:read_all": "Read all profile info",
              "activity:read": "Read public activities",
              "activity:read_all": "Read all activities",
              "activity:write": "Write activities"
            }
          }
        }
      }
    }
  },
  "security": [
    {
      "stravaOAuth": [
        "activity:read_all",
        "profile:read_all"
      ]
    }
  ]
}
```

</details>
