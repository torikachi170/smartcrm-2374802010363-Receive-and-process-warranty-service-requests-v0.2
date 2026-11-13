# Smart CRM API Reference — Track SE

This reference documents the HTTP API currently registered in
`src/presentation/api.py`. Warranty-ticket endpoints shown in earlier design
documents are not implemented and are intentionally excluded. The interactive
OpenAPI page is available at `/docs` when the service is running.

## Conventions

- Base URL: `http://localhost:8000` by default. Change the host port with
  `APP_PORT`.
- Request and response bodies use JSON and UTF-8.
- Protected operations require `Authorization: Bearer <access_token>`.
- Validation errors return HTTP 400 with this envelope:

```json
{
  "error": {
    "code": "VALIDATION_FAILED",
    "message": "Dữ liệu yêu cầu không hợp lệ",
    "fields": {
      "issue_desc": ["String should have at least 10 characters"]
    }
  }
}
```

- Authentication errors return HTTP 401 with `UNAUTHORIZED`; the login
  endpoint instead returns `INVALID_CREDENTIALS` for incorrect credentials.
- Database errors return HTTP 503 with a generic error envelope. Detailed
  database exceptions are logged server-side by exception type, not returned
  to the caller.
- The prototype has one environment-configured demo account. It does not have
  employee roles or per-owner access control.

## Endpoints

### 1. `GET /health`

1. **Method and path:** `GET /health`
2. **Description:** Confirms that the API process responds.
3. **Authentication:** Not required.
4. **Parameters:** None.
5. **Successful response:** HTTP 200, `{"status":"ok"}`.
6. **Errors:** No application-specific error response is defined.
7. **Limits:** This endpoint does not query PostgreSQL; check both Compose
   service health states separately.

Verified response: `{"status":"ok"}`.

### 2. `POST /api/auth/login`

1. **Method and path:** `POST /api/auth/login`
2. **Description:** Authenticates the configured demo account and issues a
   short-lived bearer access token.
3. **Authentication:** Not required.
4. **Parameters:** JSON body with non-empty `username` (maximum 100 characters)
   and `password` (maximum 200 characters).
5. **Successful response:** HTTP 200:

```json
{"access_token":"<redacted>","token_type":"bearer","expires_in":1800}
```

6. **Errors:** HTTP 400 `VALIDATION_FAILED`; HTTP 401
   `INVALID_CREDENTIALS`; HTTP 503 `AUTH_NOT_CONFIGURED`.
7. **Limits:** Token lifetime is configured by `JWT_ACCESS_TOKEN_MINUTES`
   (default 30). The token is signed using HS256. Never put credentials or
   tokens in a URL or publish them in examples.

### 3. `GET /api/customers`

1. **Method and path:** `GET /api/customers`
2. **Description:** Lists customers, optionally filtering by exact phone
   number.
3. **Authentication:** Bearer token required.
4. **Parameters:** Optional query parameter `phone`; when supplied it must
   contain exactly 10 ASCII digits and start with `0`.
5. **Successful response:** HTTP 200, an array of customer objects:

```json
[
  {
    "customer_id": 1,
    "full_name": "Khách kiểm thử",
    "phone": "090****001",
    "address": null
  }
]
```

6. **Errors:** HTTP 400 `VALIDATION_FAILED`; HTTP 401 `UNAUTHORIZED`; HTTP 503
   `DATABASE_NOT_CONFIGURED` or `DATABASE_UNAVAILABLE`.
7. **Limits:** Phone numbers are masked in responses. An omitted `phone`
   returns the list available to the prototype; there is no role-based
   filtering.

### 4. `POST /api/customers`

1. **Method and path:** `POST /api/customers`
2. **Description:** Creates a customer.
3. **Authentication:** Bearer token required.
4. **Parameters:** JSON body: required `full_name` (string, maximum 120),
   required `phone` (string, 10 ASCII digits starting with `0`), optional
   `address` (string or null, maximum 255). Text is trimmed; unknown fields
   are rejected.
5. **Successful response:** HTTP 201; customer object with
   `customer_id`, `full_name`, masked `phone`, and `address`.
6. **Errors:** HTTP 400 `VALIDATION_FAILED`; HTTP 401 `UNAUTHORIZED`; HTTP 409
   `PHONE_EXISTS`; HTTP 503 database configuration/availability error.
7. **Limits:** Phone values must be unique. The response never returns the
   full phone number.

Example request:

```json
{"full_name":"Khách kiểm thử","phone":"0900000001","address":"12 Lê Lợi"}
```

### 5. `GET /api/customers/{customer_id}`

1. **Method and path:** `GET /api/customers/{customer_id}`
2. **Description:** Reads one customer.
3. **Authentication:** Bearer token required.
4. **Parameters:** Path parameter `customer_id`, integer greater than or equal
   to 1.
5. **Successful response:** HTTP 200 customer object as documented for
   `GET /api/customers`.
6. **Errors:** HTTP 400 for an invalid path value; HTTP 401 `UNAUTHORIZED`;
   HTTP 404 `CUSTOMER_NOT_FOUND`; HTTP 503 database configuration/availability
   error.
7. **Limits:** Phone is masked. No per-owner authorization is implemented.

### 6. `PATCH /api/customers/{customer_id}`

1. **Method and path:** `PATCH /api/customers/{customer_id}`
2. **Description:** Updates the supplied customer fields.
3. **Authentication:** Bearer token required.
4. **Parameters:** Path parameter `customer_id` (positive integer); JSON body
   may contain `full_name` (maximum 120), `phone` (10 ASCII digits starting
   with `0`), and/or `address` (maximum 255 or null). At least one field is
   required; unknown fields are rejected.
5. **Successful response:** HTTP 200 updated customer object.
6. **Errors:** HTTP 400 `VALIDATION_FAILED`; HTTP 401 `UNAUTHORIZED`; HTTP 404
   `CUSTOMER_NOT_FOUND`; HTTP 409 `PHONE_EXISTS`; HTTP 503 database
   configuration/availability error.
7. **Limits:** Phone remains masked in the response. Text values are trimmed.

Example request:

```json
{"address":"12 Lê Lợi"}
```

### 7. `DELETE /api/customers/{customer_id}`

1. **Method and path:** `DELETE /api/customers/{customer_id}`
2. **Description:** Deletes a customer only when no device or ticket refers to
   that customer.
3. **Authentication:** Bearer token required.
4. **Parameters:** Path parameter `customer_id`, positive integer; no body.
5. **Successful response:** HTTP 204 with an empty body.
6. **Errors:** HTTP 400 for an invalid path value; HTTP 401 `UNAUTHORIZED`;
   HTTP 404 `CUSTOMER_NOT_FOUND`; HTTP 409 `CUSTOMER_HAS_DEVICES`,
   `CUSTOMER_HAS_TICKETS`, or `RECORD_IN_USE`; HTTP 503 database
   configuration/availability error.
7. **Limits:** No cascading delete is performed.

### 8. `GET /api/customers/{customer_id}/devices`

1. **Method and path:** `GET /api/customers/{customer_id}/devices`
2. **Description:** Lists devices belonging to one customer.
3. **Authentication:** Bearer token required.
4. **Parameters:** Path parameter `customer_id`, positive integer.
5. **Successful response:** HTTP 200 array of device objects with
   `device_id`, `customer_id`, `model_name`, `serial_no`, `purchase_date`,
   `warranty_months`, `note`, and `priority`.
6. **Errors:** HTTP 400 for an invalid path value; HTTP 401 `UNAUTHORIZED`;
   HTTP 404 `CUSTOMER_NOT_FOUND`; HTTP 503 database configuration/availability
   error.
7. **Limits:** The customer must exist. `priority` may be null.

### 9. `POST /api/customers/{customer_id}/devices`

1. **Method and path:** `POST /api/customers/{customer_id}/devices`
2. **Description:** Registers a device for an existing customer.
3. **Authentication:** Bearer token required.
4. **Parameters:** Path parameter `customer_id`, positive integer. JSON body:
   required `model_name` (maximum 120), `serial_no` (maximum 50), and `note`
   (maximum 255); optional `purchase_date` (ISO date) and
   `warranty_months` (default 12, range 1–32767). Unknown fields are rejected.
5. **Successful response:** HTTP 201 device object with its generated IDs and
   submitted device details.
6. **Errors:** HTTP 400 `VALIDATION_FAILED`; HTTP 401 `UNAUTHORIZED`; HTTP 404
   `CUSTOMER_NOT_FOUND`; HTTP 409 `SERIAL_EXISTS`; HTTP 503 database
   configuration/availability error.
7. **Limits:** Serial/IMEI must be unique.

Example request:

```json
{
  "model_name": "Phone X",
  "serial_no": "SERIAL-001",
  "purchase_date": "2026-01-01",
  "warranty_months": 12,
  "note": "Demo"
}
```

### 10. `GET /api/devices/{device_id}`

1. **Method and path:** `GET /api/devices/{device_id}`
2. **Description:** Reads one device.
3. **Authentication:** Bearer token required.
4. **Parameters:** Path parameter `device_id`, positive integer.
5. **Successful response:** HTTP 200 device object with `device_id`,
   `customer_id`, `model_name`, `serial_no`, `purchase_date`,
   `warranty_months`, and `note`.
6. **Errors:** HTTP 400 for an invalid path value; HTTP 401 `UNAUTHORIZED`;
   HTTP 404 `DEVICE_NOT_FOUND`; HTTP 503 database configuration/availability
   error.
7. **Limits:** No per-owner authorization is implemented.

### 11. `PATCH /api/devices/{device_id}`

1. **Method and path:** `PATCH /api/devices/{device_id}`
2. **Description:** Updates the supplied device fields.
3. **Authentication:** Bearer token required.
4. **Parameters:** Path parameter `device_id`, positive integer. Partial JSON
   body may contain `model_name`, `serial_no`, `purchase_date`,
   `warranty_months`, and/or `note`; unknown fields such as `customer_id` are
   rejected.
5. **Successful response:** HTTP 200 updated device object.
6. **Errors:** HTTP 400 `VALIDATION_FAILED`; HTTP 401 `UNAUTHORIZED`; HTTP 404
   `DEVICE_NOT_FOUND`; HTTP 409 `SERIAL_EXISTS`; HTTP 503 database
   configuration/availability error.
7. **Limits:** At least one field must be provided; serial/IMEI must remain
   unique; warranty months range from 1 to 32767.

### 12. `DELETE /api/devices/{device_id}`

1. **Method and path:** `DELETE /api/devices/{device_id}`
2. **Description:** Deletes a device only when no ticket refers to it.
3. **Authentication:** Bearer token required.
4. **Parameters:** Path parameter `device_id`, positive integer; no body.
5. **Successful response:** HTTP 204 with an empty body.
6. **Errors:** HTTP 400 for an invalid path value; HTTP 401 `UNAUTHORIZED`;
   HTTP 404 `DEVICE_NOT_FOUND`; HTTP 409 `DEVICE_HAS_TICKETS` or
   `RECORD_IN_USE`; HTTP 503 database configuration/availability error.
7. **Limits:** No cascading delete is performed.

### 13. `POST /api/classify`

1. **Method and path:** `POST /api/classify`
2. **Description:** Suggests an issue category and its default priority from
   the issue description; it does not create a warranty ticket.
3. **Authentication:** Bearer token required.
4. **Parameters:** JSON body with required `issue_desc` string, 10–2000
   characters. Leading/trailing whitespace is trimmed.
5. **Successful response:** HTTP 200:

```json
{"category_id":3,"category_name":"SAC","priority":"TRUNG_BINH"}
```

6. **Errors:** HTTP 400 `VALIDATION_FAILED`; HTTP 401 `UNAUTHORIZED`; HTTP 503
   `DATABASE_NOT_CONFIGURED`, `DATABASE_UNAVAILABLE`, or
   `CATEGORY_NOT_CONFIGURED`.
7. **Limits:** Classification is keyword-based and falls back to the `KHAC`
   category; a staff member must review the suggestion. No ticket endpoint is
   currently implemented.

## Browser pages

- `GET /` redirects to `/login`.
- `GET /login` serves the prototype login page.
- `GET /docs` serves FastAPI's interactive OpenAPI UI.
- Static assets are served under `/static/`.

These pages are not business API endpoints and are not listed as operations in
the generated OpenAPI schema.
