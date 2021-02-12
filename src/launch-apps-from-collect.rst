.. spelling::

  myapp
  mycompany
  myform
  mygroup
  org
  someinteger
  sometext
  uuid

Launching External Apps
========================

.. seealso::

  :doc:`launch-collect-from-app`
	
.. _launch-apps-single-field:

Launching external apps to populate single fields
------------------------------------------------------

ODK Collect can launch external applications using the ``ex:intentAction(intentExtras)`` appearance on string, integer or decimal fields. The application launched could be one that the enumerator uses and quits without Collect needing any data from it. For example, a form could configure `the Maps.me application <https://github.com/mapsme/api-android/wiki/Build-Route-and-Show-On-Map-via-Intent>`_ to provide directions to a destination and not need any information back from the app. An external application could also be used to populate the string, integer or decimal field that launched it. In order to populate a field, the app that is launched must be designed to returned a value as described in the :ref:`external app design <external-app-design>` section below.

This feature configures an Android intent. Intents are messaging objects used to request an action from another app component. Learn more in `the Android docs <https://developer.android.com/guide/components/intents-filters.html>`_.

Collect builds the action using the text immediately after ``ex:`` and before an opening parenthesis (if it exists). For example, in the string ``ex:org.opendatakit.counter(intentExtras)``, the action name is ``org.opendatakit.counter``. An action name must include a namespace, such as ``org.opendatakit`` or ``com.google``. For example, ``ex:org.opendatakit.counter`` is valid but ``ex:counter`` is not.

The parameters defined in the optional parentheses represent extended data ("extras") to be added to the intent. Extras are specified by a comma-delimited list of ``name=value`` pairs. The text to the left of the equals sign represents the extra name and may require a namespace. The text to the right of the equals sign represents the extra value.

The values of extras can be:

- XPath expressions referring to other fields and including function calls
- String literals defined in single quotes
- Raw integers or decimals

An extra named ``value`` that holds the current value for the current field is always passed with the intent. Since v1.4.3, additional parameters with user-defined names can be specified. There are two reserved names: ``value`` and ``uri_data``.

Since Collect v1.16.0, the data that the intent operates on can be set by :ref:`using the reserved uri_data parameter <launch-apps-uri-data>`. This is particularly useful for implicit intents such as ``android.intent.action.SENDTO``.

XLSForm
~~~~~~~~~

.. csv-table:: survey
  :header: type, name, label, appearance

  integer, counter, Click launch to start the counter app, "ex:org.opendatakit.counter(form_id='counter-form', form_name='Counter Form', question_id='1', question_name='Counter')"

In the examples above, the extras specified have names ``form_id``, ``form_name``, ``question_id``, and ``question_name``.

.. _external-app-design:

Designing an external app to return a single value to Collect
---------------------------------------------------------------
When an activity that is launched returns to Collect, Collect will look for an intent extra named ``value`` and use its value to populate the field that triggered the application launch. See `a counter app <https://github.com/getodk/counter/blob/master/app/src/main/java/org/opendatakit/counter/activities/CounterActivity.java#L100>`_ to see an example of how this is done.

.. _launch-apps-multiple-fields:

Launching external apps to populate multiple fields
-------------------------------------------------------

Since v1.4.3, a ``field-list`` group can have an ``intent`` attribute that allows an external application to populate it. Notice that the ``ex:`` prefix used when populating a single field is not included to populate multiple fields.

XLSForm
~~~~~~~~~

.. csv-table:: survey
  :header: type, name, label, appearance, body::intent

  begin_group, mygroup, Fields to populate, field-list, "org.mycompany.myapp(my_text='Some text', uuid=/myform/meta/instanceID)"
  text, sometext, Some text
  integer, someinteger, Some integer
  decimal, somedecimal, Some decimal
  image, someimage, Some image
  video, somevideo, Some video
  audio, someaudio, Some audio
  file, somefile, Some file
  end_group                                        

.. code-block:: xml

  <group ref="/myform/mygroup" appearance="field-list" 
          intent="org.mycompany.myapp(my_text='Some text', 
                                      uuid=/myform/meta/instanceID)">
    <label>Fields to populate</label>
    <input ref="/myform/mygroup/sometext">
      <label>Some text</label>
    </input>
    <input ref="/myform/mygroup/someinteger">
      <label>Some integer</label>
    </input>
    <input ref="/myform/mygroup/somedecimal">
      <label>Some decimal</label>
    </input>
    <input ref="/myform/mygroup/someimage">
      <label>Some image</label>
    </input>
    <input ref="/myform/mygroup/somevideo">
      <label>Some video</label>
    </input>
    <input ref="/myform/mygroup/someaudio">
      <label>Some audio</label>
    </input>
    <input ref="/myform/mygroup/somefile">
      <label>Some file</label>
    </input>
  </group>

The ``intent`` attribute is only used when the group has an ``appearance`` of ``field-list``. The format and the functionality of the ``intent`` value is the same as above. If the bundle of values returned by the external application contains values with keys that match the type and the name of the sub-fields, then the values from the bundle overwrite the current values of those sub-fields.

The external app is launched with the parameters that are defined in the intent string plus the values of all the sub-fields that are either text, decimal, integer or binary. Any other sub-field is invisible to the external app.

.. _launch-apps-uri-data:

Specifying a URI as intent data
---------------------------------

Since Collect v1.16.0, the value for the reserved parameter name ``uri_data`` is converted to a URI and used as the data for the intent. The intent data determines which application to launch when using implicit intents such as `SENDTO <https://developer.android.com/reference/android/content/Intent#ACTION_SENDTO>`_. For example:

``ex:android.intent.action.SENDTO(uri_data='smsto:5555555', sms_body=${message})``
  Launches a new message in an SMS app with the destination number set to ``5555555`` and the message body set to the contents of the ``message`` field.

``ex:android.intent.action.SENDTO(uri_data='mailto:example@example.com?subject=${subject}&body=${message})``
  Launches a new message in an email app with destination address set to ``example@example.com``, the subject set to the contents of the ``subject`` field and the body set to the contents of the ``message`` field.

``ex:android.intent.action.DIAL(uri_data='tel:5555555')``
  Launches a phone dialer with the number ``5555555`` as the number to dial.

Notice that the URI must include a `scheme <https://www.iana.org/assignments/uri-schemes/uri-schemes.xhtml>`_, such as ``mailto:`` or ``https://``.
