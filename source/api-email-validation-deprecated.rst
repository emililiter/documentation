.. _api-email-validation:

Email Validation
================

.. note:: Our Email Validation service has been renamed to Email Verification service. While the names are different, nothing within our codebase has changed to cause a disruption in service.

This API endpoint is an email address verification service. We will verify the given address based on:

- Mailbox detection

- Syntax checks (RFC defined grammar)

- DNS validation

- Spell checks

- Email Service Provider (ESP) specific local-part grammar (if available).

Pricing details for Mailgun's email verification service can be found on our `pricing page`_.

Mailgun's email verification service is intended to verify email addresses submitted through forms like newsletters, online registrations and shopping carts.  Refer to our `Acceptable Use Policy (AUP)`_ for more information about how to use the service appropriately.

.. _pricing page: https://www.mailgun.com/pricing
.. _Acceptable Use Policy (AUP): http://mailgun.com/aup


Different email verification rates are given based on the API endpoint. Both the public and private
API endpoints are limited to a burst per minute rate. The public endpoints have a default
limit of calls per month, which can be changed, to prevent abuse of the public API key.
The use of private API endpoints for email verification is encouraged and there is no limit past the initial
burst per minute rate. It is highly suggested that the private key is used whenever possible.

.. warning:: Do not use your Mailgun private API key on publicly accessible code. Instead, use your Mailgun public key, available in the "Security" tab under the **Account** section of the Control Panel.

.. warning:: This version is deprecated. The new version of the Validations API is here :ref:`api-email-validations`


.. code-block:: url

     GET /address/validate

Given an arbitrary address, verifies address based off defined checks.

.. container:: ptable

 ====================== ========================================================
 Parameter         	Description
 ====================== ========================================================
 address           	An email address to verify. (Maximum: 512 characters)
 api_key          	If you cannot use HTTP Basic Authentication (preferred),
                   	you can pass your public api_key in as a parameter.
 mailbox_verification	If set to true, a mailbox verification check will be 
 			performed against the address.
 			
			The default is `False`.
 ====================== ========================================================

.. code-block:: url

    GET /address/parse

Parses a delimiter-separated list of email addresses into two lists: parsed addresses and unparsable portions. The parsed addresses are a list of addresses that are syntactically valid (and optionally pass DNS and ESP specific grammar checks). The unparsable list is a list of character sequences that could not be parsed (or optionally failed DNS or ESP specific grammar checks). Delimiter characters are comma (,) and semicolon (;).

.. container:: ptable

 ================= ========================================================
 Parameter         Description
 ================= ========================================================
 addresses         A delimiter separated list of addresses.
                   (Maximum: 8000 characters)
 syntax_only       Perform only syntax checks or DNS and ESP specific
                   verification as well. (true by default)
 api_key           If you cannot use HTTP Basic Authentication (preferred),
                   you can pass your public api_key in as a parameter.
 ================= ========================================================

.. code-block:: url

    GET /address/private/validate

Addresses are verified based off defined checks.

This operation is only accessible with the private API key and not subject to the
daily usage limits.

.. container:: ptable

 ====================== ========================================================
 Parameter         	Description
 ====================== ========================================================
 address           	An email address to verify. (Maximum: 512 characters)
 mailbox_verification	If set to true, a mailbox verification check will be 
 			performed against the address.
 			
			The default is `False`.
 ====================== ========================================================

.. code-block:: url

    GET /address/private/parse

Parses a delimiter-separated list of email addresses into two lists: parsed addresses and unparsable portions. The parsed addresses are a list of addresses that are syntactically valid (and optionally pass DNS and ESP specific grammar checks). The unparsable list is a list of character sequences that could not be parsed (or optionally failed DNS or ESP specific grammar checks). Delimiter characters are comma (,) and semicolon (;).

This operation is only accessible with the private API key and not subject to the
daily usage limits.

.. container:: ptable

 ================= ========================================================
 Parameter         Description
 ================= ========================================================
 addresses         A delimiter separated list of addresses.
                   (Maximum: 8000 characters)
 syntax_only       Perform only syntax checks or DNS and ESP specific
                   verification as well. (true by default)
 api_key           If you can not use HTTP Basic Authentication (preferred),
                   you can pass your private api_key in as a parameter.
 ================= ========================================================

Example
~~~~~~~

Verify a single email address.

.. include:: samples/get-validate-deprecated.rst

Sample response *without* mailbox verification enabled:

.. code-block:: javascript

  {
    "address": "foo@mailgun.net",
    "did_you_mean": null,
    "is_disposable_address": false,
    "is_role_address": false,
    "is_valid": true,
    "mailbox_verification": null,
    "parts": {
        "display_name": null,
        "domain": "mailgun.net",
        "local_part": "foo"
    },
    "reason": null
  }
  
Sample response *with* mailbox verification enabled:

.. code-block:: javascript

  {
      "address": "foo@mailgun.net",
      "did_you_mean": null,
      "is_disposable_address": false,
      "is_role_address": true,
      "is_valid": true,
      "mailbox_verification": "true",
      "parts": {
          "display_name": null,
          "domain": "mailgun.net",
          "local_part": "foo"
      }
  }


Field Explanation:

=====================    =========    ============================================================================================================
Parameter                Type         Description
=====================    =========    ============================================================================================================
address                  string       Email address being verified
did_you_mean             string       Null if nothing, however if a potential typo is made, the closest suggestion is provided
is_disposable_address    boolean      If the domain is in a list of disposable email addresses, this will be appropriately categorized
is_role_address          boolean      Checks the mailbox portion of the email if it matches a specific role type ('admin', 'sales', 'webmaster')
is_valid                 boolean      Runs the email segments across a valid known provider rule list. If a violation occurs this value is false
mailbox_verification     string       If the mail_verification flag is enabled, a call is made to the ESP to return existence. (true, false, unknown or null)
parts                    object       (display_name, domain, local_part): Parsed segments of the provided email address
=====================    =========    ============================================================================================================


.. note:: is_valid returns true when an address is parsable, passes known grammar checks and an SMTP server is present. The role-based and disposable address check will not impact the state of the `is_valid` result. The user should determine whether or not to permit the address to be used.

.. note:: The mailbox verification attribute will return true if the email is valid, false if the email was invalid, or unknown if the SMTP request could not be completed. Unknown will also be returned if mailbox verification is not supported on the target mailbox provider. The outcome of the verification check will not impact the state of the is_valid result.

Parse a list of email addresses.

.. include:: samples/get-parse.rst

Sample response:

.. code-block:: javascript

    {
        "parsed": [
            "Alice <alice@example.com>",
            "bob@example.com"
        ],
        "unparsable": [
        ]
    }

jQuery Plugin
~~~~~~~~~~~~~

We also have a `jQuery plugin`_ you can use for front-end email verification.

Just remember to use your public Mailgun API key.

.. _jQuery plugin: https://github.com/mailgun/validator-demo

Attaching to a form:

.. code-block:: javascript

   $('jquery_selector').mailgun_validator({
       api_key: 'api-key',
       in_progress: in_progress_callback, // called when request is made to validator
       success: success_callback,         // called when validator has returned
       error: validation_error,           // called when an error reaching the validator has occured
   });

