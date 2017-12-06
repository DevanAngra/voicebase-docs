Hello, World
============

This Quickstart tutorial gets you up and running with your first transcription and speech analytics.


The examples assume you have these prerequisites:

- *Your Bearer token* (otherwise, see: :ref:`How to Get Your Bearer Token <token>` for details)
- *jq* for working with JSON (otherwise, see: :ref:`A Quick Note on Tools <tools>` for details)
- *curl* (comes with most Linux systems, see: :ref:`A Quick Note on Tools <tools>` for details)

Step **(1)**: Verify your Bearer Token and tools are working
------------------------------------------------------------

Start by verifying your Bearer token is working by running the following from a command prompt. Please remember to insert your Bearer token after *TOKEN=* in the first line of the example. You can also find more detailed instructions on :ref:`How to Get Your Bearer Token <token>` or :ref:`Understanding Your First Request <understanding_step1>` below.

.. code-block:: sh
  :linenos:
  :emphasize-lines: 3

  export TOKEN= # Insert your VoiceBase Bearer token after TOKEN=

  curl https://apis.voicebase.com/v3/media \
    --header "Authorization: Bearer ${TOKEN:?'(hint: insert your token after export TOKEN=)'}" \

You should see a response like this (otherwise, see :ref:`explanation <understanding_step1>` and/or :ref:`troubleshooting <hello-world-troubleshooting>`):

.. code-block:: json

  {
    "_links": {
      "self": {
        "href": "/v3/media"
      }
    },
    "media": []
  }

Step **(2)**: Upload a media file for transcription and analysis
----------------------------------------------------------------

To upload a recording for transcription and analysis, POST to /media with the recording as an attachment named media (you can also provide a URL to your recording instead using the form field 'mediaUrl').

Using local media:

.. code-block:: sh
  :linenos:
  :emphasize-lines: 2

  curl https://apis.voicebase.com/v3/media \
    --form 'media=@hello-world.mp3' \
    --header "Authorization: Bearer ${TOKEN}" \
    | tee media-post.json \
    | jq .

Using a remote media URL:

.. code-block:: sh
  :linenos:
  :emphasize-lines: 2

  curl https://apis.voicebase.com/v3/media \
    --form 'mediaUrl=http://myServer.com/mediaFile.mp3' \
    --header "Authorization: Bearer ${TOKEN}" \
    | tee media-post.json \
    | jq .



The response includes a *mediaId* (assigned by the API) and a status of *accepted*.

.. code-block:: json
  :emphasize-lines: 7

  {
    "_links": {
      "self": {
        "href": "/v3/media/10827f19-7574-4b54-bf9d-9387999eb5ec"
      },
      "progress": {
        "href": "/v3/media/10827f19-7574-4b54-bf9d-9387999eb5ec/progress"
      },
      "metadata": {
        "href": "/v3/media/10827f19-7574-4b54-bf9d-9387999eb5ec/metadata"
      }
    },
    "mediaId": "10827f19-7574-4b54-bf9d-9387999eb5ec",
    "status": "accepted",
    "dateCreated": "2017-06-22T18:23:02Z",
    "dateFinished": "2017-06-22T18:23:58Z",
    "mediaContentType": "audio/mp3",
    "length": 10031,
    "metadata": {}
  }

You can poll for status until the processing is done (for production, we recommend using `Callbacks <callbacks.html>`__).

.. code-block:: sh
  :linenos:
  :emphasize-lines: 7

  export MEDIA_ID=$( cat media-post.json | jq --raw-output .mediaId )
  export STATUS=$( cat media-post.json | jq --raw-output .status )

  while [[ ${STATUS} != 'finished' && ${STATUS} != 'failed' ]]; do
    sleep 1
    STATUS=$(
      curl https://apis.voicebase.com/v3/media/${MEDIA_ID}/progress \
        --header "Authorization: Bearer ${TOKEN}" \
        | jq --raw-output .progress.status
    )
    echo "Got status: ${STATUS} for mediaId: ${MEDIA_ID} on $( date )"
  done

Step **(3)**: Get your transcript and analytics
-----------------------------------------------

You can retrieve the JSON version of the transcript and all analytics with a simple API call.

.. code-block:: sh
  :linenos:
  :emphasize-lines: 1

  curl https://apis.voicebase.com/v3/media/${MEDIA_ID}/transcript \
    --header "Authorization: Bearer ${TOKEN}" \
    | jq .

You can also retrieve a plain-text version using *transcript/text* and the *Accept* HTTP header.

.. code-block:: sh
  :linenos:
  :emphasize-lines: 1-2

  curl https://apis.voicebase.com/v3/media/${MEDIA_ID}/transcript/text \
    --header 'Accept: text/plain' \
    --header "Authorization: Bearer ${TOKEN}"

.. _token:


How to Get Your Bearer Token
----------------------------

First, sign into the `Developer Portal <https://apis.voicebase.com/developer-portal>`__.

.. image:: /_static/Sign-Into-Developer-Portal.png
   :width: 200

Click the *Bearer Token Management* widget in the lower-left of the portal.

.. image:: /_static/Bearer-Token-Management.png
   :width: 300

Click the *+ New Token* button to generate a new Bearer token

.. image:: /_static/New-Token.png
   :width: 450

Click through on *Create Token* to generate the token.

.. image:: /_static/Create-Token.png

Save your token by Copying it to the clipboard or downloading it.

.. image:: /_static/Copy-Token-To-Clipboard.png


.. _understanding_step1:

Understanding Your First Request
--------------------------------

The root URL of the VoiceBase V3 API is **https://apis.voicebase.com/v3**. Every recording you submit for analysis appears in the **/media** collection. The first request is to GET the **/media** collection (which will be empty when you first sign up). We pro-actively limit the page size to 10 (*?limit=10*) to avoid an overwhelming response as the media collection grows.

.. code-block:: sh
  :linenos:

  export TOKEN= # Insert your VoiceBase Bearer token after TOKEN=

  curl https://apis.voicebase.com/v3/media?limit=10 \
    --header "Authorization: Bearer ${TOKEN:?'(hint: insert your token after export TOKEN=)'}" \
    | jq

If you're running this for the first time, the API returns (see: :ref:`Troubleshooting <hello-world-troubleshooting>` if you hit issues):

.. code-block:: json

  {
    "_links": {
      "self": {
        "href": "/v3/media"
      }
    },
    "media": []
  }

All successful responses from the API will include an *_links* section with `HAL`_ metadata that helps navigate the API.

.. _HAL: https://en.wikipedia.org/wiki/Hypertext_Application_Language

.. code-block:: json
   :emphasize-lines: 2

  {
    "_links": { }
  }

The *media* section the list of media in your account (up to 10 due to the limit parameter). If you have previously uploaded media, it will appear in the list.

.. code-block:: json
  :emphasize-lines: 2

  {
    "media": []
  }

Understanding Your First Upload
-------------------------------

The next step is to upload a recording to the API for transcription and analysis, but making a POST to /media, with the recording as an attachment named media.

.. code-block:: sh
  :linenos:
  :emphasize-lines: 2

  curl https://apis.voicebase.com/v3/media \
    --form media=@hello-world.mp3 \
    --header "Authorization: Bearer ${TOKEN}" \
    | jq

When you add the *--form media=@filename.mp3* parameters, *curl* automatically sets the HTTP method to *POST* and the *Content-Type* to *multipart/form-data*. This is equivalent to the more explicit:

.. code-block:: sh
  :linenos:
  :emphasize-lines: 4-5

  curl https://apis.voicebase.com/v3/media \
    --form media=@hello-world.mp3 \
    --header "Authorization: Bearer ${TOKEN}" \
    --request POST \
    --header "Content-Type: multipart/form-data" \
    | jq

Finally, many operations will rely on providing a configuration JSON attachment with additional processing instructions. Omitting the attachment is equivalent to including the following default configuration:

.. code-block:: sh
  :linenos:
  :emphasize-lines: 3

  curl https://apis.voicebase.com/v3/media \
    --form media=@hello-world.mp3 \
    --form configuration='{}' \
    --header "Authorization: Bearer ${TOKEN}" \
    | jq

Many of the Developer Guides will address how to use specific options in the configuration attachment to address various Use Cases.

.. _tools:

A Quick Note on Tools
---------------------

- **curl**: The examples in this documentation make heavy use of `curl`_ for making HTTP requests to the API.
- **jq**: The `jq`_ tool helps parse JSON responses and work with JSON data.

.. _curl: https://curl.haxx.se/docs/manpage.html
.. _jq: http://stedolan.github.io/jq/




.. _hello-world-troubleshooting:

Troubleshooting
---------------
