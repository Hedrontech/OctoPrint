.. _sec-api-fileops:

***************
File operations
***************

.. contents::

.. _sec-api-fileops-retrieveall:

Retrieve all files
==================

.. http:get:: /api/files

   Retrieve information regarding all files currently available and regarding the disk space still available
   locally in the system.

   Returns a :ref:`Retrieve response <sec-api-fileops-datamodel-retrieveresponse>`.

   **Example request**:

   .. sourcecode:: http

      GET /api/files HTTP/1.1
      Host: example.com

   **Example response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json

      {
        "files": [
          {
            "name": "whistle_v2.gcode",
            "size": 1468987,
            "date": 1378847754,
            "origin": "local",
            "refs": {
              "resource": "http://example.com/api/files/local/whistle_v2.gcode",
              "download": "http://example.com/downloads/files/local/whistle_v2.gcode"
            },
            "gcodeAnalysis": {
              "estimatedPrintTime": 1188,
              "filament": {
                "length": 810,
                "volume": 5.36
              }
            },
            "print": {
              "failure": 4,
              "success": 23,
              "last": {
                "date": 1387144346,
                "success": true
              }
            }
          },
          {
            "name": "whistle_.gco",
            "origin": "sdcard",
            "refs": {
              "resource": "http://example.com/api/files/sdcard/whistle_.gco"
            }
          }
        ],
        "free": "3.2GB"
      }

   :statuscode 200: No error

.. _sec-api-fileops-retrievelocation:

Retrieve files from specific location
=====================================

.. http:get:: /api/files/(string:location)

   Retrieve information regarding the files currently available on the selected `location` and -- if targeting
   the ``local`` location -- regarding the disk space still available locally in the system.

   Returns a :ref:`Retrieve response <sec-api-fileops-datamodel-retrieveresponse>`.

   **Example request**:

   .. sourcecode:: http

      GET /api/files/local HTTP/1.1
      Host: example.com

   **Example response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json

      {
        "files": [
          {
            "name": "whistle_v2.gcode",
            "size": 1468987,
            "date": 1378847754,
            "origin": "local",
            "refs": {
              "resource": "http://example.com/api/files/local/whistle_v2.gcode",
              "download": "http://example.com/downloads/files/local/whistle_v2.gcode"
            },
            "gcodeAnalysis": {
              "estimatedPrintTime": 1188,
              "filament": {
                "length": 810,
                "volume": 5.36
              }
            },
            "print": {
              "failure": 4,
              "success": 23,
              "last": {
                "date": 1387144346,
                "success": true
              }
            }
          }
        ],
        "free": "3.2GB"
      }

   :param location: The origin location from which to retrieve the files. Currently only ``local`` and ``sdcard`` are
                    supported, with ``local`` referring to files stored in OctoPrint's ``uploads`` folder and ``sdcard``
                    referring to files stored on the printer's SD card (if available).
   :statuscode 200: No error
   :statuscode 404: If `location` is neither ``local`` nor ``sdcard``

.. _sec-api-fileops-uploadfile:

Upload file
===========

.. http:post:: /api/files/(string:location)

   Upload a file to the selected `location`.

   Other than most of the other requests on OctoPrint's API which are expected as JSON, this request is expected as
   ``Content-Type: multipart/form-data`` due to the included file upload.

   Returns a :http:statuscode:`201` response with a ``Location`` header set to the management URL of the uploaded
   file and an :ref:`Upload Response <sec-api-fileops-datamodel-uploadresponse>` as the body upon successful completion.

   **Example request**

   .. sourcecode:: http

      POST /api/files/sdcard HTTP/1.1
      Host: example.com
      X-Api-Key: abcdef...
      Content-Type: multipart/form-data; boundary=----WebKitFormBoundaryDeC2E3iWbTv1PwMC

      ------WebKitFormBoundaryDeC2E3iWbTv1PwMC
      Content-Disposition: form-data; name="file"; filename="whistle_v2.gcode"
      Content-Type: application/octet-stream

      ;Generated with Cura_SteamEngine 13.11.2
      M109 T0 S220.000000
      T0
      ;Sliced at: Wed 11-12-2013 16:53:12
      ;Basic settings: Layer height: 0.2 Walls: 0.8 Fill: 20
      ;Print time: #P_TIME#
      ;Filament used: #F_AMNT#m #F_WGHT#g
      ;Filament cost: #F_COST#
      ;M190 S70 ;Uncomment to add your own bed temperature line
      ;M109 S220 ;Uncomment to add your own temperature line
      G21        ;metric values
      G90        ;absolute positioning
      ...
      ------WebKitFormBoundaryDeC2E3iWbTv1PwMC
      Content-Disposition: form-data; name="select"

      true
      ------WebKitFormBoundaryDeC2E3iWbTv1PwMC
      Content-Disposition: form-data; name="print"

      true
      ------WebKitFormBoundaryDeC2E3iWbTv1PwMC--

   **Example response**

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      Location:

      {
        "files": {
          "local": {
            "name": "whistle_v2.gcode",
            "origin": "local",
            "refs": {
              "resource": "http://example.com/api/files/local/whistle_v2.gcode",
              "download": "http://example.com/downloads/files/local/whistle_v2.gcode"
            }
          },
          "sdcard": {
            "name": "whistle_.gco",
            "origin": "sdcard",
            "refs": {
              "resource": "http://example.com/api/files/sdcard/whistle_.gco"
            }
          }
        },
        "done": true
      }

   :param location: The target location to which to upload the file. Currently only ``local`` and ``sdcard`` are supported
                    here, with ``local`` referring to OctoPrint's ``uploads`` folder and ``sdcard`` referring to
                    the printer's SD card. If an upload targets the SD card, it will also be stored locally first.
   :form file:      The file to upload, including a valid ``filename``.
   :form select:    Whether to select the file directly after upload (``true``) or not (``false``). Optional, defaults
                    to ``false``.
   :form print:     Whether to start printing the file directly after upload (``true``) or not (``false``). If set, `select`
                    is implicitely ``true`` as well. Optional, defaults to ``false``.
   :statuscode 201: No error
   :statuscode 400: If no `file` is included in the request, or the request is otherwise invalid.
   :statuscode 404: If `location` is neither ``local`` nor ``sdcard`` or trying to upload to SD card and SD card support
                    is disabled
   :statuscode 409: If the upload of the file would override the file that is currently being printed or if an upload
                    to SD card was requested and the printer is either not operational or currently busy with a print job.
   :statuscode 415: If the file is neither a ``gcode`` nor an ``stl`` file (or it is an ``stl`` file but slicing support
                    is disabled)
   :statuscode 500: If the upload failed internally

.. _sec-api-fileops-retrievefileinfo:

Retrieve a specific file's information
======================================

.. http:get:: /api/files/(string:location)/(path:filename)

   Retrieves the selected file's information.

   If the file is unknown, a :http:statuscode:`404` is returned.

   On success, a :http:statuscode:`200` is returned, with a :ref:`file information item <sec-api-fileops-datamodel-fileinfo>`
   as the response body.

   **Example Request**

   .. sourcecode:: http

      GET /api/files/local/whistle_v2.gcode HTTP/1.1
      Host: example.com

   **Example Response**

   .. sourcecode:: http

      HTTP/1.1 200 Ok
      Content-Type: application/json

      {
        "name": "whistle_v2.gcode",
        "size": 1468987,
        "date": 1378847754,
        "origin": "local",
        "refs": {
          "resource": "http://example.com/api/files/local/whistle_v2.gcode",
          "download": "http://example.com/downloads/files/local/whistle_v2.gcode"
        },
        "gcodeAnalysis": {
          "estimatedPrintTime": 1188,
          "filament": {
            "length": 810,
            "volume": 5.36
          }
        },
        "print": {
          "failure": 4,
          "success": 23,
          "last": {
            "date": 1387144346,
            "success": true
          }
        }
      }

   :param location: The location of the file for which to retrieve the information, either ``local`` or ``sdcard``.
   :param filename: The filename of the file for which to retrieve the information
   :statuscode 200: No error
   :statuscode 404: If `target` is neither ``local`` nor ``sdcard``, ``sdcard`` but SD card support is disabled or the
                    requested file was not found

.. _sec-api-fileops-filecommand:

Issue a file command
====================

.. http:post:: /api/files/(string:target)/(path:filename)

   Issue a file command to an existing file. Currently supported commands are:

   select
     Selects a file for printing. Additional parameters are:

     * ``print``: Optional, if set to ``true`` the file will start printing directly after selection. If the printer
       is not operational when this parameter is present and set to ``true``, the request will fail with a response
       of ``409 Conflict``.

   Upon success, a status code of :http:statuscode:`204` and an empty body is returned.

   **Example Select Request**

   .. sourcecode:: http

      POST /api/files/local/whistle_v2.gcode HTTP/1.1
      Host: example.com
      Content-Type: application/json
      X-Api-Key: abcdef...

      {
        "command": "select",
        "print": true
      }

   :param target:        The target location on which to delete the file, either ``local`` (for OctoPrint's ``uploads``
                         folder) or ``sdcard`` for the printer's SD card (if available)
   :param filename:      The filename of the file for which to issue the command
   :json string command: The command to issue for the file, currently only ``select`` is supported
   :json boolean print:  ``select`` command: Optional, whether to start printing the file directly after selection,
                         defaults to ``false``.
   :statuscode 200:      No error
   :statuscode 400:      If the `command` is unknown or the request is otherwise invalid
   :statuscode 404:      If `target` is neither ``local`` nor ``sdcard`` or the requested file was not found
   :statuscode 409:      If a selected file is supposed to start printing directly but the printer is not operational.

.. _sec-api-fileops-delete:

Delete file
===========

.. http:delete:: /api/files/(string:target)/(path:filename)

   Delete the selected `filename` on the selected `target`.

   If the file to be deleted is currently being printed, a :http:statuscode:`409` will be returned.

   Returns a :http:statuscode:`204` after successful deletion.

   **Example Request**

   .. sourcecode:: http

      DELETE /api/files/local/whistle_v2.gcode HTTP/1.1
      Host: example.com
      X-Api-Key: abcdef...

   :param target:   The target location on which to delete the file, either ``local`` (for OctoPrint's ``uploads``
                    folder) or ``sdcard`` for the printer's SD card (if available)
   :param filename: The filename of the file to delete
   :statuscode 204: No error
   :statuscode 404: If `target` is neither ``local`` nor ``sdcard`` or the requested file was not found
   :statuscode 409: If the file to be deleted is currently being printed

.. _sec-api-fileops-datamodel:

Datamodel
=========

.. _sec-api-fileops-datamodel-retrieveresponse:

Retrieve response
-----------------

.. list-table::
   :widths: 15 5 10 30
   :header-rows: 1

   * - Name
     - Multiplicity
     - Type
     - Description
   * - ``files``
     - 0..*
     - Array of :ref:`File information items <sec-api-fileops-datamodel-fileinfo>`
     - The list of requested files. Might be an empty list if no files are available
   * - ``free``
     - 0..1
     - String
     - The amount of disk space in bytes available in the local disk space (refers to OctoPrint's ``uploads`` folder). Only
       returned if file list was requested for origin ``local`` or all origins.

.. _sec-api-fileops-datamodel-uploadresponse:

Upload response
---------------

.. list-table::
   :widths: 15 5 10 30
   :header-rows: 1

   * - Name
     - Multiplicity
     - Type
     - Description
   * - ``files``
     - 1
     - Object
     - Abridged information regarding the file that was just uploaded. If only uploaded to ``local`` this will only
       contain the ``local`` property. If uploaded to SD card, this will contain both ``local`` and ``sdcard`` properties.
   * - ``files.local``
     - 1
     - :ref:`sec-api-fileops-datamodel-fileinfo`
     - The information regarding the file that was just uploaded to the local storage (only the fields ``name``,
       ``origin`` and ``refs`` will be set).
   * - ``files.sdcard``
     - 0..1
     - :ref:`sec-api-fileops-datamodel-fileinfo`
     - The information regarding the file that was just uploaded to the printer's SD card (only the fields ``name``,
       ``origin`` and ``refs`` will be set).
   * - ``done``
     - 1
     - Boolean
     - Whether the file processing after upload has already finished (``true``) or not, e.g. due to first needing
       to perform a slicing step (``false``). Clients may use this information to direct progress displays related to
       the upload.

.. _sec-api-fileops-datamodel-fileinfo:

File information
----------------

.. list-table::
   :widths: 15 5 10 30
   :header-rows: 1

   * - Name
     - Multiplicity
     - Type
     - Description
   * - ``name``
     - 1
     - String
     - The name of the file
   * - ``size``
     - 0..1
     - Number
     - The size of the file in bytes. Only available for ``local`` files.
   * - ``date``
     - 0..1
     - Unix timestamp
     - The timestamp when this file was uploaded. Only available for ``local`` files.
   * - ``origin``
     - 1
     - String, either ``local`` or ``sdcard``
     - The origin of the file, ``local`` when stored in OctoPrint's ``uploads`` folder, ``sdcard`` when stored on the
       printer's SD card (if available)
   * - ``refs``
     - 0..1
     - :ref:`sec-api-fileops-datamodel-ref`
     - References relevant to this file
   * - ``gcodeAnalysis``
     - 0..1
     - :ref:`GCODE analysis information <sec-api-fileops-datamodel-gcodeanalysis>`
     - Information from the analysis of the GCODE file, if available.
   * - ``prints``
     - 0..1
     - :ref:`Print information <sec-api-fileops-datamodel-prints>`
     - Information regarding prints of this file, if available.

.. _sec-api-fileops-datamodel-gcodeanalysis:

GCODE analysis information
--------------------------

.. list-table::
   :widths: 15 5 10 30
   :header-rows: 1

   * - Name
     - Multiplicity
     - Type
     - Description
   * - ``estimatedPrintTime``
     - 0..1
     - Integer
     - The estimated print time of the file, in seconds
   * - ``filament``
     - 0..1
     - Object
     - The estimated usage of filament
   * - ``filament.length``
     - 0..1
     - Integer
     - The length of filament used, in mm
   * - ``filament.volume``
     - 0..1
     - Float
     - The volume of filament used, in cm³


.. _sec-api-fileops-datamodel-prints:

Print information
-----------------

.. list-table::
   :widths: 15 5 10 30
   :header-rows: 1

   * - Name
     - Multiplicity
     - Type
     - Description
   * - ``failure``
     - 1
     - Number
     - The number of failed prints on record for the file
   * - ``success``
     - 1
     - Number
     - The number of successful prints on record for the file
   * - ``last``
     - 0..1
     - Object
     - Information regarding the last print on record for the file
   * - ``last.date``
     - 1
     - Unix timestamp
     - Timestamp when this file was printed last
   * - ``last.success``
     - 1
     - Boolean
     - Whether the last print on record was a success (``true``) or not (``false``)

.. _sec-api-fileops-datamodel-ref:

References
----------

.. list-table::
   :widths: 15 5 10 30
   :header-rows: 1

   * - Name
     - Multiplicity
     - Type
     - Description
   * - ``resource``
     - 1
     - URL
     - The resource that represents the file (e.g. for issuing commands to or for deleting)
   * - ``download``
     - 0..1
     - URL
     - The download URL for the file
   * - ``model``
     - 0..1
     - URL
     - The model from which this file was generated (e.g. an STL, currently not used)
