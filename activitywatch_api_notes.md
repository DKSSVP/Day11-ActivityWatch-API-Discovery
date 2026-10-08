# ActivityWatch API Notes --- Day 11

## 1. Objective

Stand up ActivityWatch locally and understand the REST API required for
the Day 12 focus-tracking integration.

ActivityWatch v0.14.0 was installed locally. The required watchers were
confirmed:

-   `aw-watcher-window`
-   `aw-watcher-afk`

Local server:

``` text
http://localhost:5600
```

## 2. Bucket List

### API

``` http
GET http://localhost:5600/api/0/buckets
```

### Buckets observed

  ---------------------------------------------------------------------------------------------
  Bucket ID                        Type              Client                Purpose
  -------------------------------- ----------------- --------------------- --------------------
  `aw-watcher-afk_5CG748074F`      `afkstatus`       `aw-watcher-afk`      Tracks active/away
                                                                           status

  `aw-watcher-window_5CG748074F`   `currentwindow`   `aw-watcher-window`   Tracks the active
                                                                           application/window
  ---------------------------------------------------------------------------------------------

Machine-specific suffix:

``` text
5CG748074F
```

## 3. Window Watcher Events

### API

``` http
GET http://localhost:5600/api/0/buckets/aw-watcher-window_5CG748074F/events
```

Sample event observed locally:

``` json
{
  "id": 57,
  "timestamp": "2026-10-08T07:34:31.931000+00:00",
  "duration": 8.255,
  "data": {
    "app": "chrome.exe",
    "title": "Python Gen AI Learning Plan - Google Chrome"
  }
}
```

### Event schema

``` text
id
timestamp
duration
data.app
data.title
```

For Day 12, the main fields are:

``` text
timestamp
duration
data.app
data.title
```

Applications observed during testing included:

``` text
chrome.exe
explorer.exe
SearchApp.exe
WhatsApp.Root.exe
```

## 4. AFK Watcher Events

### API

``` http
GET http://localhost:5600/api/0/buckets/aw-watcher-afk_5CG748074F/events
```

Sample event observed locally:

``` json
[
  {
    "id": 2,
    "timestamp": "2026-10-08T07:22:17.422000+00:00",
    "duration": 833.0,
    "data": {
      "status": "not-afk"
    }
  }
]
```

### AFK schema

``` text
timestamp
duration
data.status
```

The AFK bucket will later be used to exclude idle/away periods from
focus calculations.

Basic flow:

``` text
Window activity
      +
AFK activity
      ↓
Remove AFK periods
      ↓
Calculate actual focus time
```

## 5. Time-Range Query

### API

``` http
GET http://localhost:5600/api/0/buckets/aw-watcher-window_5CG748074F/events?start=2026-10-08T07:33:00Z&end=2026-10-08T07:35:00Z
```

This request was tested successfully.

The response contained window events within the requested time range,
including:

``` json
{
  "id": 63,
  "timestamp": "2026-10-08T07:34:55.523000+00:00",
  "duration": 4.477,
  "data": {
    "app": "chrome.exe",
    "title": "localhost:5600/api/0/buckets/aw-watcher-window_5CG748074F/events - Google Chrome"
  }
}
```

This confirms that `start` and `end` can be used to retrieve events for
a specific time window.

## 6. curl Examples

### List buckets

``` bash
curl http://localhost:5600/api/0/buckets
```

### Get window events

``` bash
curl http://localhost:5600/api/0/buckets/aw-watcher-window_5CG748074F/events
```

### Get AFK events

``` bash
curl http://localhost:5600/api/0/buckets/aw-watcher-afk_5CG748074F/events
```

### Get events by time range

``` bash
curl "http://localhost:5600/api/0/buckets/aw-watcher-window_5CG748074F/events?start=2026-10-08T07:33:00Z&end=2026-10-08T07:35:00Z"
```

## 7. Browser / Postman Examples

### List buckets

``` text
GET
http://localhost:5600/api/0/buckets
```

### Window events

``` text
GET
http://localhost:5600/api/0/buckets/aw-watcher-window_5CG748074F/events
```

### AFK events

``` text
GET
http://localhost:5600/api/0/buckets/aw-watcher-afk_5CG748074F/events
```

### Window events by time range

``` text
GET
http://localhost:5600/api/0/buckets/aw-watcher-window_5CG748074F/events?start=2026-10-08T07:33:00Z&end=2026-10-08T07:35:00Z
```

## 8. API Endpoint Summary

  ----------------------------------------------------------------------------------------------------------
  Requirement             Method                  Endpoint
  ----------------------- ----------------------- ----------------------------------------------------------
  List buckets            GET                     `/api/0/buckets`

  Get window events       GET                     `/api/0/buckets/<window_bucket>/events`

  Get AFK events          GET                     `/api/0/buckets/<afk_bucket>/events`

  Query by time range     GET                     `/api/0/buckets/<bucket>/events?start=<start>&end=<end>`
  ----------------------------------------------------------------------------------------------------------

ActivityWatch REST endpoints use the `/api/0/` API path.

## 9. Day 12 Integration Notes

The Day 12 `focus_tracker.py` will need to:

1.  Query the window watcher for the required period.
2.  Read `timestamp`, `duration`, `data.app`, and `data.title`.
3.  Identify development-related applications/windows.
4.  Query the AFK bucket for the same period.
5.  Exclude periods where the user was AFK.
6.  Aggregate the remaining activity into focus minutes.

## 10. Troubleshooting Note

The ActivityWatch local web server was accessible at:

``` text
http://localhost:5600
```

The ActivityWatch desktop/tray UI did not open normally from Windows
Search during testing. However, the local server and both required
watchers were working, so API exploration continued through the browser.

This did not block the Day 11 REST API work.

## 11. Reference

Official ActivityWatch REST API documentation:

https://docs.activitywatch.net/en/latest/api/rest.html

Official ActivityWatch data model documentation:

https://docs.activitywatch.net/en/latest/data-model.html
