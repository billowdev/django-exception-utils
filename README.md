# Application Exception Django Rest framework Code

# CUSTOM STANDARD
# base on https://www.django-rest-framework.org/api-guide/status-codes/
- for custom response that if want to use only 
- if want to response code only 200 ok but want to seperate response type
- if want to response code only 400 bad req. but want to seperate response type
- if want to response code only 500 server down. but want to seperate response type
```py

def get_default_message_response(response_type):


def get_custom_response_code(status_code):
    error_codes = {
		# Informational - 1xx
        "10000": "CONTINUE",
        "10100": "SWITCHING_PROTOCOLS",
		# Successful - 2xx
        "20000": "SUCCESS",
        "20001": "CREATED",
        "20002": "SUCCESS_UPDATE",
        "20003": "SUCCESS_REMOVE",
        "20100": "CREATED",
        "20200": "ACCEPTED",
        "20300": "NON_AUTHORITATIVE_INFORMATION",
        "20400": "NO_CONTENT",
        "20500": "RESET_CONTENT",
        "20600": "PARTIAL_CONTENT",
        "20700": "MULTI_STATUS",
        "20800": "ALREADY_REPORTED",
        "22600": "IM_USED",
        # Redirection - 3xx
        "30000": "MULTIPLE_CHOICES",
        "30100": "MOVED_PERMANENTLY",
        "30200": "FOUND",
        "30300": "SEE_OTHER",
        "30400": "NOT_MODIFIED",
        "30500": "USE_PROXY",
        "30600": "RESERVED",
        "30700": "TEMPORARY_REDIRECT",
        "30800": "PERMANENT_REDIRECT",
        # Client Error - 4xx
        "40000": "BAD_REQUEST",
        "40001": "BAD_REQUEST_CREATE",
        "40002": "BAD_REQUEST_RETRIEVE",
        "40003": "BAD_REQUEST_UPDATE",
        "40004": "BAD_REQUEST_DELETE",
        "40100": "UNAUTHORIZED",
        "40200": "PAYMENT_REQUIRED",
        "40300": "FORBIDDEN",
        "40301": "BUNDLE_EXISTED",
        "40400": "NOT_FOUND",
        "40401": "UNKNOWN_URL",
        "40402": "INSTANCE_NOT_FOUND",
        "40500": "METHOD_NOT_ALLOWED",
        "40600": "NOT_ACCEPTABLE",
        "40700": "PROXY_AUTHENTICATION_REQUIRED",
        "40800": "REQUEST_TIMEOUT",
        "40900": "CONFLICT",
        "40110": "GONE",
        "41100": "LENGTH_REQUIRED",
        "41200": "PRECONDITION_FAILED",
        "41300": "REQUEST_ENTITY_TOO_LARGE",
        "41400": "REQUEST_URI_TOO_LONG",
        "41500": "UNSUPPORTED_MEDIA_TYPE",
        "41600": "REQUESTED_RANGE_NOT_SATISFIABLE",
        "41700": "EXPECTATION_FAILED",
        "42200": "UNPROCESSABLE_ENTITY",
        "42300": "LOCKED",
        "42400": "FAILED_DEPENDENCY",
        "42600": "UPGRADE_REQUIRED",
        "42800": "PRECONDITION_REQUIRED",
        "42900": "TOO_MANY_REQUESTS",
        "43100": "REQUEST_HEADER_FIELDS_TOO_LARGE",
        "45100": "UNAVAILABLE_FOR_LEGAL_REASONS",
        # Server Error - 5xx
        "50000": "INTERNAL_SERVER_ERROR",
        "50100": "NOT_IMPLEMENTED",
        "50200": "BAD_GATEWAY",
        "50300": "SERVICE_UNAVAILABLE",
        "50300": "SERVICE_UNAVAILABLE",
        "50400": "GATEWAY_TIMEOUT",
        "50500": "HTTP_VERSION_NOT_SUPPORTED",
        "50600": "VARIANT_ALSO_NEGOTIATES",
        "50700": "INSUFFICIENT_STORAGE",
        "50800": "LOOP_DETECTED",
        "50900": "BANDWIDTH_LIMIT_EXCEEDED",
        "50110": "NOT_EXTENDED",
        "51100": "NETWORK_AUTHENTICATION_REQUIRED",
    }

    # Use the status_code to look up the error code in the dictionary
    error_code = error_codes.get(status_code)
    
    return error_code
```

- default message response
```py
def get_default_message_response(response_type):
    default_messages = {
        "1xx": "This is an informational response.",
        "2xx": "The request was successful.",
        "3xx": "The request was redirected.",
        "4xx": "The request was failed.",
        "5xx": "Server error occurred."
    }

    default_message = default_messages.get(response_type)
    
    return default_message
```

### You can modified as you wish in ApplicationException that what you want to reply or response to View
- exceptions.py
```py
class FormatChildErrorExecption:
	# TODO: For use with normal exception that is custom exception
	def format_exception(self, exception, exception_type):
		if not isinstance(exception, dict):
			error_data = exception.to_dict()
		else:
			error_data = exception
		
		new_child_error = []
		if error_data.get('child_error') in (None, []):
			error_data.pop("child_error", None)
			new_child_error = [{exception_type: error_data}]
		else:
			new_child_error = [{exception_type: error_data}]

		return new_child_error

	# TODO: to use with serializer validation error
	def format_serializer_validation_error(self, exception, error_detail="serializers.ValidationError", exception_type="appValidationException"):
		error_dict = dict(exception.detail)
		child_errors = []

		for field, errors in error_dict.items():
			for error in errors:
				child_errors.append({
					"exception_type": exception_type,
					"field": field,
					"message": str(error),
					"error": error_detail,
				})

		return child_errors
        
class ApplicationException(Exception):
	def __init__(self, error_detail, field, message, child_error=None, exception_type="AppException"):
		super().__init__(f"{error_detail} Error: Field '{field}' - {message}")
		# TODO: Exception Type
		self.exception_type = exception_type
		# * The description for error_detail argument
		# TODO: This can provide more technical and detailed information about the error.
		# TODO: It can include specifics like variable values, API codes, or any other relevant technical details.
		# TODO: The error_detail is typically longer and more detailed than the message.
		self.error_detail = error_detail
		self.field = field
		# * The description for message argument
		# TODO: This should provide a concise, high-level description of the error.
		# TODO: It should convey what went wrong in a clear and user-friendly manner.
		# TODO: The message should be brief and to the point.
		self.message = message
		# TODO: child exception
		self.child_error = child_error if child_error is not None else []

	def to_dict(self):
		return {
			"exception_type": self.exception_type,
			"field": self.field,
			"message": self.message,
			"error": f"Error: {self.error_detail}",
			"child_error": self.child_error,
		}

	def __str__(self):
		return self.__repr__()

	def __repr__(self):
		return str(self.to_dict())

class AppValidateException(ApplicationException):
	def __init__(self, field, message, child_error=None, error_detail=None, exception_type="AppValidateException"):
		if error_detail:
			super().__init__(error_detail, field, message, child_error, exception_type)
		else:
			super().__init__("Serializer Exception", field, message, child_error, exception_type)
			
class AppSerializerException(ApplicationException):
	def __init__(self, field, message, child_error=None, error_detail=None, exception_type="AppSerializerException"):
		if error_detail:
			super().__init__(error_detail, field, message, child_error)
		else:
			super().__init__("Serializer Exception", field, message, child_error, exception_type)
			
class AppRequestException(ApplicationException):
	def __init__(self, field, message, child_error=None, error_detail=None, exception_type="AppRequestException"):
		if error_detail:
			super().__init__(error_detail, field, message, child_error, exception_type)
		else:
			super().__init__("Request Exception", field, message, child_error, exception_type)
```

# Application Exception format Example usage

## Instance Not found

- example 1 if data is process in system

```py
raise AppValidateException(
    field="EXCEPTION_FIELD",
    message="Failed to create a EXCEPTION_FIELD due to a validation error.",
    error_detail="The specified 'EXCEPTION_FIELD' instance was not found in the system.",
    exception_type="EXCEPTION_TYPE"
)
```

- example 2 if data from the request

```py
raise AppValidateException(
    field="EXCEPTION_FIELD",
    message="Failed to create a EXCEPTION_FIELD due to a validation error.",
    error_detail="The requested EXCEPTION_FIELD instance could not be located.",
    exception_type="EXCEPTION_TYPE"
)
```

- exampleal 3 process instance by system not found
```py
raise AppValidateException(
    field="FIELD_EXCEPTION",
    message="Failed to retrieve a FIELD_EXCEPTION due to a validation error.",
    error_detail="The specified 'FIELD_EXCEPTION' instance was not found in the system.",
    exception_type="EXCEPTION_TYPE"
)
```

# Validation

## REQUEST VALUE NOT FOUND

- example 1 for error that when create someting
  
```py
raise AppValidateException(
    field="EXCEPTION_FIELD",
    message="Failed to create a EXCEPTION_TYPE due to a validation error.",
    error_detail="The 'booking_reference' value is missing or None, and a EXCEPTION_TYPE instance cannot be created.",
    exception_type="EXCEPTION_TYPE"
)
```

- example 2 for error that when find someting
  
```py
raise AppValidateException(
    field="EXCEPTION_FIELD",
    message="EXCEPTION_TYPE due to a validation error.",
    error_detail="The 'EXCEPTION_FIELD' value is missing or None",
    exception_type="EXCEPTION_TYPE"
)
```

- example 3 for find somthing with the data from request but that something not found

```py
raise AppValidateException(
    field="EXCEPTION_FIELD",
    message="Failed to locate the requested EXCEPTION_FIELD instance using the provided Reference due to a validation error.",
    error_detail=f"No EXCEPTION_FIELD instance matching Reference={booking_reference} was found in the system.",
    exception_type="EXCEPTION_TYPE"
)
```

## The provide value not match with status on the system

```py
raise AppValidateException(
    field="EXCEPTION_FIELD",
    message="Booking has been canceled and cannot be modified.",
    error_detail="The provided value does not match the expected system status.",
    exception_type="CancellationException"
)
```

## Validation Max length exceeded

```py
raise AppValidateException(
    field=field,
    message=f"The provided value exceeds the maximum length of {str(max_length)} characters.",
    error_detail=f"The length of the value '{field}' exceeded the maximum allowed limit of {max_length} characters.",
    exception_type="EXCEPTION_TYPE"
)
```

## AppSerializerException have AppValidationChild

- have custom validation on serializer error

```py
new_child_error = self.format_exception(exception, exception.exception_type)
raise AppSerializerException(
    field="EXCEPTION_FIELD",
    message="Failed to validate the EXCEPTION_FIELD.",
    exception_type="AppValidationFailure",
    error_detail="Validation error occurred for 'EXCEPTION_TYPE'.",
    child_error=new_child_error
)
```

- have serializer validation error

```py
raise AppSerializerException(
    field="EXCEPTION_FIELD",
    message="Failed to validate the EXCEPTION_FIELD.",
    exception_type="SerializerValidationFailure",
    error_detail="Validation error occurred for 'EXCEPTION_TYPE'.",
    child_error=new_child_error
)
```

- mainly serializer throw an error
```py
child_errors = self.format_serializer_validation_error(exception=exception, exception_type="SERIALIZER_CLASS_NAME")
raise AppSerializerException(
    field="SERIALIZER_NAME",
    message="SERIALIZER_NAME serialization encountered an error.",
    error_detail="An error occurred while serializing a SERIALIZER_NAME in SERIALIZER_CLASS_NAME.",
    exception_type="SERIALIZER_CLASS_NAME",
    child_error=child_errors
)
```
