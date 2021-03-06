# Callbacks

VoiceBase can optionally make a callback request to a specific url when media upload processing is complete.

## Uploading Media with Callbacks Enabled

To request a processing-completed callback from VoiceBase, include a JSON configuration attachment with your media POST. The configuration attachment should contain the key:

```json

{
    "configuration" : {
      "publish": {
        "callbacks": [
          { 
            "url" : "https://example.org/callback",
            "method" : "POST",
            "include" : [ "transcripts", "keywords", "topics", "metadata" ]
          }
        ]
      }
    }
}

```

### Configuration Description

- `configuration` : root object for configuration data
    - `publish` : object for publish-specific configuration
        - `callbacks` : array of callbacks, with one object per callback desired
            - `[n]` : callback array element
                - `url` : the https url for delivering a callback notification
                - `method` : the HTTPS method for callback delivery, with the following supported values:
                    - `POST`: deliver callbacks as an HTTPS POST
                - `include` :  array of data to include with the callback, with the following supported values:
                    - `transcripts`: include transcripts for the media
                    - `topics` : include topics and corresponding keywords for the media
                    - `metadata` : include supplied metadata, often useful for correlated to records in a different system
                    
For example, to upload media from a local file called recording.mp3 and receive a callback at https://example.org/callback, make the following POST request using curl, or an equivalent request using a tool of your choice:

```bash

curl https://apis.voicebase.com/v2-beta/media \
    --header "Authorization: Bearer $TOKEN" \
    --form media=@recording.mp3 \
    --form 'configuration={
        "configuration" : {
          "publish": {
            "callbacks": [
              { 
                "url" : "https://example.org/callback",
                "method" : "POST",
                "include" : [ "transcripts", "keywords", "topics", "metadata" ]
              }
            ]
          }
        }
      }'

```

When using callbacks, you can still query the status of the media processing using a GET request to /media/{mediaId}.

## Callback Data

When media processing is complete, VoiceBase will call back your specified endpoint by making an HTTPS POST request. The body is a JSON object with the following data:

        
```json
{
  "_links" : {  
    "self" : { "href" : "https://apis.voicebase.com/v2-beta/media/3c7499c2-aebd-4efc-aaf7-0ad4636cc160" }
  },
  "callback" : {
    "success" : true,
    "errors" : [ ],
    "warnings" : [ ],
    "event" : {
      "status" : "finished"
    }
  },
  "media" : {
    "mediaId" : "3c7499c2-aebd-4efc-aaf7-0ad4636cc160",
    "status" : "finished",
    "metadata" : {
      "latest" : {
        "title" : "An interesting conversation",
        "external" : {
          "id" : "fe26afe9-e5cf-4f48-976d-29dd758a0550"
        }
      }
    },
    "transcripts" : {
      "latest" : {
        "transcriptId" : "e0589bc2-5266-4c52-8347-42c2c186fe16",
        "type" : "machine",
        "engine" : "premium",
        "features" : [ "speakerTurns", "speakerId" ],
        "formats" : [ "json", "srt" ],
        "words" : [
          { "p" : 1, "c" : 0.927, "s" : 10, "e" : 1390, "w" : "him" }
        ]
      }
    },
    "topics" : {
      "latest" : {
        "revision" : "cc7bd8d1-bb76-47c9-bf52-54acd43aea57",
        "terms" : [
          { 
            "name" : "mobile industry",
            "id" : "6f71daf4-4f97-45c6-a29f-5726cbdba270",
            "spotting" : false,
            "keywords" : [ 
              "mobile advertising",
              "mobile usage"
            ]
          },
          {
            "name" : "mobile-phone",
            "spotting" : true,
            "keywords" : [
              "iPhone"
            ]
          }
        ]
      }
    }
  }
}

```

### Data Description

- `_links` : HAL metadata with a URL for the corresponding media item
    - `self` : section for the media item
        - `href` : URL for the media item
- `callback` : object with metadata about the callback event
    - `success` : true for success notifications and false for failure notifications
    - `errors` : an optional array of errors (omitted or empty for success notifications)
    - `warnings` : an optional array of warnings
    - `event` : a JSON object that event triggering the callback (typicall a status change)
        - `status` : the new status of the media processing job, with the following possible values:
            - `finished` : media processing completed successfully
            - `failed` : media processing failed
- `media` : the requested data for the media item
    - `mediaId` : the unique VoiceBase id for the media item
    - `status` : the status of processing for the media item
    - `metadata` : the metadata for the media item, typically for correlation to external systems (present if requested when media is uploaded)
    - `transcripts` : the transcipt(s) for the media (present if requested when media is uploaded)
    - `topics` : the topics and keywords for the media (present if requested when media is uploaded)
