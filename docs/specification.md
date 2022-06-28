# Verifiable Presentation Generation Service Specification

> A plugin-based service that allows issuers to render verifiable presentations
> from templates, and store it in a queryable database for clients to
> list/retrieve.

## Terms and Definitions

### Defined by the [W3C VC Specification](https://www.w3.org/TR/vc-data-model)

- [Credential](https://www.w3.org/TR/vc-data-model/#dfn-credential)
- [Presentation](https://www.w3.org/TR/vc-data-model/#dfn-presentations)
- [Issuer](https://www.w3.org/TR/vc-data-model/#dfn-issuers)
- [Holder](https://www.w3.org/TR/vc-data-model/#dfn-holders)
- [Verifier](https://www.w3.org/TR/vc-data-model/#dfn-verifier)
- [Registry](https://www.w3.org/TR/vc-data-model/#dfn-verifiable-data-registries)

### Defined by this Specification

- Generator: A plugin-based service that is setup by, and allows, an issuer to
  generate a verifiable presentation from a template and store it in a queryable
  database for holders/clients to list, verify and retrieve.
- Renderer: A service that accepts data and a template and renders the
  presentation in the requested format.
- Template Store: A service that stores templates designed by the Issuer.

## Architecture

![VPGS Architecture](media/architecture.png)

### Flow of Events

- Issuance of one or more verifiable credentials.
- Storage of verifiable credentials in a credential repository (such as a
  digital wallet).
- Composition and storage of verifiable presentation templates in a template
  store.
- Setting up of a generator by the issuer.
- Making a request to the generator set up in the previous step to generate a
  verifiable presentation.
- Verification of the verifiable presentation by the verifier.

## Template Store API

### **POST `/templates`**

> Store a new template in the store.

**Request Body**

| Field    | Type     | Required | Notes                                                       |
| -------- | -------- | -------- | ----------------------------------------------------------- |
| template | `string` | `true`   |                                                             |
| renderer | `enum`   | `true`   | Can be one of the following: `ejs`, `jstl`, `nunjucks`.     |
| schema   | `object` | `false`  | The AJV schema of the data required to render the template. |

Example:

```json
{
	"template": "Hello ${data.name}",
	"renderer": "jstl",
	"schema": {
		"name": {
			"type": "string",
			"required": true
		}
	}
}
```

**Response**

Example:

```json
{
	"meta": {
		"status": 201
	},
	"data": {
		"id": "PTqVo635KbGZXZ4KzUJV86iBpixt",
		"template": "Hello ${data.name}",
		"renderer": "jstl",
		"schema": {
			"name": {
				"type": "string",
				"required": true
			}
		}
	}
}
```

**Errors**

_Template-Renderer Mismatch_

```json
{
	"meta": {
		"status": 400
	},
	"error": {
		"code": "improper-payload",
		"message": "The template provided cannot be rendered using the specified renderer."
	}
}
```

_Missing/Invalid Payload_

```json
{
	"meta": {
		"status": 400
	},
	"error": {
		"code": "improper-payload",
		"message": "Invalid value provided for field `renderer`."
	}
}
```

### **GET `/templates/{id}`**

> Retrieve a template and its metadata from the store.

**Request Parameters**

| Field | Type     | Required | Notes                               |
| ----- | -------- | -------- | ----------------------------------- |
| id    | `string` | `true`   | The ID of the template to retrieve. |

**Response**

Example:

```json
{
	"meta": {
		"status": 200
	},
	"data": {
		"id": "PTqVo635KbGZXZ4KzUJV86iBpixt",
		"template": "Hello ${data.name}",
		"renderer": "jstl",
		"schema": {
			"name": {
				"type": "string",
				"required": true
			}
		}
	}
}
```

**Errors**

_Template Not Found_

```json
{
	"meta": {
		"status": 404
	},
	"error": {
		"code": "entity-not-found",
		"message": "A template with the specified ID does not exist."
	}
}
```

## Renderer API

### **POST `/render`**

> Render a certificate using the given data from a specified template.

**Request Body**

| Field    | Type     | Required | Notes                                                                                                    |
| -------- | -------- | -------- | -------------------------------------------------------------------------------------------------------- |
| template | `string` | `true`   | The ID of the template to use to render the data.                                                        |
| data     | `object` | `true`   | The data to render.                                                                                      |
| output   | `enum`   | `true`   | The format in which to output the presentation. Can be one of the following: `svg`, `pdf`, `txt`, `htm`. |

Example:

```json
{
	"template": "PTqVo635KbGZXZ4KzUJV86iBpixt",
	"data": {
		"name": "Happy"
	},
	"output": "htm"
}
```

**Response**

Example:

```json
{
	"meta": {
		"status": 200
	},
	"data": "Hello Happy"
}
```

**Errors**

_Missing/Invalid Payload_

```json
{
	"meta": {
		"status": 400
	},
	"error": {
		"code": "improper-payload",
		"message": "Invalid value provided for field `output`."
	}
}
```

_Template Not Found_

```json
{
	"meta": {
		"status": 404
	},
	"error": {
		"code": "entity-not-found",
		"message": "A template with the specified ID does not exist."
	}
}
```

_Insufficient Data/Schema Mismatch_

```json
{
	"meta": {
		"status": 412
	},
	"error": {
		"code": "precondition-failed",
		"message": "The data provided was insufficient to render the presentation using the specified template."
	}
}
```

## Registry API

### **GET `/presentations`**

> Search for presentations related to a certain subject.

**Request Query Parameters**

| Field     | Type     | Required | Notes                                                                        |
| --------- | -------- | -------- | ---------------------------------------------------------------------------- |
| `subject` | `string` | `false`  | The ID of the subject in the credentials to return. (`credentialSubject.id`) |

Example:

```http
GET /presentations?subject=did%3Aexample%3AF4RGIuxcKIjygFThqsXW9GX6HocV
```

**Response**

```json
{
	"meta": {
		"status": 200
	},
	"data": [
		{
			"@context": [
				"https://www.w3.org/2018/credentials/v1",
				"https://www.w3.org/2018/credentials/examples/v1"
			],
			"id": "did:example:mTvrhQMMf6KBEJFItejG0tpohz5U",
			"type": "VerifiablePresentation",
			"verifiableCredential": [
				{
					"@context": [
						"https://www.w3.org/2018/credentials/v1",
						"https://www.w3.org/2018/credentials/examples/v1"
					],
					"id": "did:example:F4RGIuxcKIjygFThqsXW9GX6HocV",
					"type": ["VerifiableCredential", "IdentityCredential"],
					"issuer": "https://example.edu/issuers/565049",
					"issuanceDate": "2010-01-01T19:23:24Z",
					"credentialSubject": {
						"id": "did:example:Fibyu0uFoIHW7Eq3jY7v0rQs5OLb",
						"name": "Happy"
					},
					"proof": {
						"type": "RsaSignature2018",
						"created": "2017-06-18T21:19:10Z",
						"proofPurpose": "assertionMethod",
						"verificationMethod": "https://example.edu/issuers/565049#key-1",
						"jws": "..."
					}
				}
			],
			"proof": {
				"type": "RsaSignature2018",
				"created": "2018-09-14T21:19:10Z",
				"proofPurpose": "authentication",
				"verificationMethod": "did:example:yNsJn2b6YEaLlAMYQyOk24vb3fss#keys-1",
				"challenge": "1f44d55f-f161-4938-a659-f8026467f126",
				"domain": "4jt78h47fh47",
				"jws": "..."
			}
		}
	]
}
```

### **POST `/presentations`**

> Store a presentation in the registry.

**Request Body**

| Field        | Type     | Required | Notes                                     |
| ------------ | -------- | -------- | ----------------------------------------- |
| presentation | `object` | `true`   | A valid verifiable presentation to store. |

Example:

```json
{
	"presentation": {
		"@context": [
			"https://www.w3.org/2018/credentials/v1",
			"https://www.w3.org/2018/credentials/examples/v1"
		],
		"id": "did:example:mTvrhQMMf6KBEJFItejG0tpohz5U",
		"type": "VerifiablePresentation",
		"verifiableCredential": [
			{
				"@context": [
					"https://www.w3.org/2018/credentials/v1",
					"https://www.w3.org/2018/credentials/examples/v1"
				],
				"id": "did:example:F4RGIuxcKIjygFThqsXW9GX6HocV",
				"type": ["VerifiableCredential", "IdentityCredential"],
				"issuer": "https://example.edu/issuers/565049",
				"issuanceDate": "2010-01-01T19:23:24Z",
				"credentialSubject": {
					"id": "did:example:Fibyu0uFoIHW7Eq3jY7v0rQs5OLb",
					"name": "Happy"
				},
				"proof": {
					"type": "RsaSignature2018",
					"created": "2017-06-18T21:19:10Z",
					"proofPurpose": "assertionMethod",
					"verificationMethod": "https://example.edu/issuers/565049#key-1",
					"jws": "..."
				}
			}
		],
		"proof": {
			"type": "RsaSignature2018",
			"created": "2018-09-14T21:19:10Z",
			"proofPurpose": "authentication",
			"verificationMethod": "did:example:yNsJn2b6YEaLlAMYQyOk24vb3fss#keys-1",
			"challenge": "1f44d55f-f161-4938-a659-f8026467f126",
			"domain": "4jt78h47fh47",
			"jws": "..."
		}
	}
}
```

**Response**

Example:

```json
{
	"meta": {
		"status": 201
	},
	"data": {
		"@context": [
			"https://www.w3.org/2018/credentials/v1",
			"https://www.w3.org/2018/credentials/examples/v1"
		],
		"id": "did:example:mTvrhQMMf6KBEJFItejG0tpohz5U",
		"type": "VerifiablePresentation",
		"verifiableCredential": [
			{
				"@context": [
					"https://www.w3.org/2018/credentials/v1",
					"https://www.w3.org/2018/credentials/examples/v1"
				],
				"id": "did:example:F4RGIuxcKIjygFThqsXW9GX6HocV",
				"type": ["VerifiableCredential", "IdentityCredential"],
				"issuer": "https://example.edu/issuers/565049",
				"issuanceDate": "2010-01-01T19:23:24Z",
				"credentialSubject": {
					"id": "did:example:Fibyu0uFoIHW7Eq3jY7v0rQs5OLb",
					"name": "Happy"
				},
				"proof": {
					"type": "RsaSignature2018",
					"created": "2017-06-18T21:19:10Z",
					"proofPurpose": "assertionMethod",
					"verificationMethod": "https://example.edu/issuers/565049#key-1",
					"jws": "..."
				}
			}
		],
		"proof": {
			"type": "RsaSignature2018",
			"created": "2018-09-14T21:19:10Z",
			"proofPurpose": "authentication",
			"verificationMethod": "did:example:yNsJn2b6YEaLlAMYQyOk24vb3fss#keys-1",
			"challenge": "1f44d55f-f161-4938-a659-f8026467f126",
			"domain": "4jt78h47fh47",
			"jws": "..."
		}
	}
}
```

**Errors**

_Missing/Invalid Payload_

```json
{
	"meta": {
		"status": 400
	},
	"error": {
		"code": "improper-payload",
		"message": "Invalid verifiable presentation provided."
	}
}
```

### **GET `/presentations/{id}`**

> Retrive a presentation from the registry.

**Request Parameters**

| Field | Type     | Required | Notes                                 |
| ----- | -------- | -------- | ------------------------------------- |
| id    | `string` | `true`   | The ID of the presentation to revoke. |

**Response**

Example:

```json
{
	"meta": {
		"status": 200
	},
	"data": {
		"@context": [
			"https://www.w3.org/2018/credentials/v1",
			"https://www.w3.org/2018/credentials/examples/v1"
		],
		"id": "did:example:mTvrhQMMf6KBEJFItejG0tpohz5U",
		"type": "VerifiablePresentation",
		"verifiableCredential": [
			{
				"@context": [
					"https://www.w3.org/2018/credentials/v1",
					"https://www.w3.org/2018/credentials/examples/v1"
				],
				"id": "did:example:F4RGIuxcKIjygFThqsXW9GX6HocV",
				"type": ["VerifiableCredential", "IdentityCredential"],
				"issuer": "https://example.edu/issuers/565049",
				"issuanceDate": "2010-01-01T19:23:24Z",
				"credentialSubject": {
					"id": "did:example:Fibyu0uFoIHW7Eq3jY7v0rQs5OLb",
					"name": "Happy"
				},
				"proof": {
					"type": "RsaSignature2018",
					"created": "2017-06-18T21:19:10Z",
					"proofPurpose": "assertionMethod",
					"verificationMethod": "https://example.edu/issuers/565049#key-1",
					"jws": "..."
				}
			}
		],
		"proof": {
			"type": "RsaSignature2018",
			"created": "2018-09-14T21:19:10Z",
			"proofPurpose": "authentication",
			"verificationMethod": "did:example:yNsJn2b6YEaLlAMYQyOk24vb3fss#keys-1",
			"challenge": "1f44d55f-f161-4938-a659-f8026467f126",
			"domain": "4jt78h47fh47",
			"jws": "..."
		}
	}
}
```

**Errors**

_Presentation Not Found_

```json
{
	"meta": {
		"status": 404
	},
	"error": {
		"code": "entity-not-found",
		"message": "A presentation with the specified ID does not exist."
	}
}
```

### **PUT `/presentations/{id}`**

> Update a presentation in the registry.

**Request Parameters**

| Field | Type     | Required | Notes                                 |
| ----- | -------- | -------- | ------------------------------------- |
| id    | `string` | `true`   | The ID of the presentation to revoke. |

**Request Body**

| Field        | Type     | Required | Notes                                     |
| ------------ | -------- | -------- | ----------------------------------------- |
| presentation | `object` | `true`   | A valid verifiable presentation to store. |

**Response**

Example:

```json
{
	"meta": {
		"status": 200
	},
	"data": {
		"@context": [
			"https://www.w3.org/2018/credentials/v1",
			"https://www.w3.org/2018/credentials/examples/v1"
		],
		"id": "did:example:mTvrhQMMf6KBEJFItejG0tpohz5U",
		"type": "VerifiablePresentation",
		"verifiableCredential": [
			{
				"@context": [
					"https://www.w3.org/2018/credentials/v1",
					"https://www.w3.org/2018/credentials/examples/v1"
				],
				"id": "did:example:F4RGIuxcKIjygFThqsXW9GX6HocV",
				"type": ["VerifiableCredential", "IdentityCredential"],
				"issuer": "https://example.edu/issuers/565049",
				"issuanceDate": "2010-01-01T19:23:24Z",
				"credentialSubject": {
					"id": "did:example:Fibyu0uFoIHW7Eq3jY7v0rQs5OLb",
					"name": "Happy"
				},
				"proof": {
					"type": "RsaSignature2018",
					"created": "2017-06-18T21:19:10Z",
					"proofPurpose": "assertionMethod",
					"verificationMethod": "https://example.edu/issuers/565049#key-1",
					"jws": "..."
				}
			}
		],
		"proof": {
			"type": "RsaSignature2018",
			"created": "2018-09-14T21:19:10Z",
			"proofPurpose": "authentication",
			"verificationMethod": "did:example:yNsJn2b6YEaLlAMYQyOk24vb3fss#keys-1",
			"challenge": "1f44d55f-f161-4938-a659-f8026467f126",
			"domain": "4jt78h47fh47",
			"jws": "..."
		}
	}
}
```

**Errors**

_Missing/Invalid Payload_

```json
{
	"meta": {
		"status": 400
	},
	"error": {
		"code": "improper-payload",
		"message": "Invalid verifiable presentation provided."
	}
}
```

_Presentation Not Found_

```json
{
	"meta": {
		"status": 404
	},
	"error": {
		"code": "entity-not-found",
		"message": "A presentation with the specified ID does not exist."
	}
}
```

> **Note**
>
> There is no endpoint to delete/revoke presentations as they are automatically
> invalidated when the credentials they encapsulate are expired/revoked/deleted.

## Generator API

### **POST `/application`**

> Create an application to configure the generator. Each application uses a
> single template from a store to render presentations using a renderer service.

**Request Body**

| Field    | Type     | Required | Notes                             |
| -------- | -------- | -------- | --------------------------------- |
| name     | `string` | `true`   | The name of the application.      |
| template | `object` | `true`   | The template store configuration. |
| renderer | `object` | `true`   | The renderer configuration.       |

Example:

```json
{
	"name": "Happy Inc",
	"template": {
		"store": "https://example.edu/template-store/",
		"id": "PTqVo635KbGZXZ4KzUJV86iBpixt"
	},
	"renderer": {
		"uri": "https://example.edu/renderer/"
	}
}
```

**Response**

Example:

```json
{
	"meta": {
		"status": 201
	},
	"data": {
		"name": "Happy Inc",
		"id": "WSJ6ZiU3tWjllJjHuA9mm",
		"template": {
			"store": "https://example.edu/template-store/",
			"id": "PTqVo635KbGZXZ4KzUJV86iBpixt"
		},
		"renderer": {
			"uri": "https://example.edu/renderer/"
		},
		"keys": []
	}
}
```

**Errors**

_Invalid Server URI_

```json
{
	"meta": {
		"status": 400
	},
	"error": {
		"code": "improper-payload",
		"message": "The URI provided is not a valid template store."
	}
}
```

_Missing/Invalid Payload_

```json
{
	"meta": {
		"status": 400
	},
	"error": {
		"code": "improper-payload",
		"message": "Invalid value provided for field `renderer.uri`."
	}
}
```

### **POST `/application/{id}/keys`**

> Generate a new key pair used to sign verifiable presentations.

**Request Body**

| Field     | Type      | Required | Notes                                                                        |
| --------- | --------- | -------- | ---------------------------------------------------------------------------- |
| algorithm | `enum`    | `true`   | The algorithm for the key. Could be one of the following: `ed25519` or `rsa` |
| size      | `integer` | `true`   | The size of the keypair generated.                                           |

**Response**

Example:

```json
{
	"meta": {
		"status": 201
	},
	"data": [
		{
			"id": "oLrLoSxG8nNupXoNZD8fJ",
			"uri": "https://example.edu/keys/public/1",
			"algorithm": "rsa",
			"size": 4096,
			"certificate": "...",
			"public": "..."
		}
	]
}
```

**Errors**

_Missing/Invalid Payload_

```json
{
	"meta": {
		"status": 400
	},
	"error": {
		"code": "improper-payload",
		"message": "Invalid value provided for field `algorithm`."
	}
}
```

### **DELETE `/applications/{id}/keys/{kid}`**

> Deletes a key pair that was used to sign verifiable presentations.

**Request Parameters**

| Field | Type     | Required | Notes                        |
| ----- | -------- | -------- | ---------------------------- |
| id    | `string` | `true`   | The ID of the application.   |
| kid   | `string` | `true`   | The ID of the key to delete. |

**Response**

Example:

```json
{
	"meta": {
		"status": 200
	},
	"data": []
}
```

**Errors**

_Application/Key Not Found_

```json
{
	"meta": {
		"status": 404
	},
	"error": {
		"code": "entity-not-found",
		"message": "A key with the specified ID does not exist for the given application."
	}
}
```

### **GET `/applications/{id}`**

> Retrieve the application metadata and configuration.

**Request Parameters**

| Field | Type     | Required | Notes                                  |
| ----- | -------- | -------- | -------------------------------------- |
| id    | `string` | `true`   | The ID of the application to retrieve. |

**Response**

Example:

```json
{
	"meta": {
		"status": 200
	},
	"data": {
		"id": "WSJ6ZiU3tWjllJjHuA9mm",
		"name": "Happy Inc",
		"template": {
			"store": "https://example.edu/template-store/",
			"id": "PTqVo635KbGZXZ4KzUJV86iBpixt"
		},
		"renderer": {
			"uri": "https://example.edu/renderer/"
		},
		"keys": [
			{
				"id": "oLrLoSxG8nNupXoNZD8fJ",
				"uri": "https://example.edu/keys/public/1",
				"algorithm": "rsa",
				"size": 4096,
				"certificate": "...",
				"public": "..."
			}
		]
	}
}
```

**Errors**

_Application Not Found_

```json
{
	"meta": {
		"status": 404
	},
	"error": {
		"code": "entity-not-found",
		"message": "An application with the specified ID does not exist."
	}
}
```

### **PATCH `/application/{id}`**

> Update an application's configuration.

**Request Body**

| Field    | Type     | Required | Notes                             |
| -------- | -------- | -------- | --------------------------------- |
| name     | `string` | `false`  | The name of the application.      |
| template | `object` | `true`   | The template store configuration. |
| renderer | `object` | `true`   | The renderer configuration.       |

Example:

```json
{
	"name": "Happy Co"
}
```

**Response**

Example:

```json
{
	"meta": {
		"status": 200
	},
	"data": {
		"name": "Happy Co",
		"id": "WSJ6ZiU3tWjllJjHuA9mm",
		"template": {
			"store": "https://example.edu/template-store/",
			"id": "PTqVo635KbGZXZ4KzUJV86iBpixt"
		},
		"renderer": {
			"uri": "https://example.edu/renderer/"
		},
		"keys": []
	}
}
```

**Errors**

_Invalid Server URI_

```json
{
	"meta": {
		"status": 400
	},
	"error": {
		"code": "improper-payload",
		"message": "The URI provided is not a valid template store."
	}
}
```

_Missing/Invalid Payload_

```json
{
	"meta": {
		"status": 400
	},
	"error": {
		"code": "improper-payload",
		"message": "Invalid value provided for field `renderer.uri`."
	}
}
```

_Application Not Found_

```json
{
	"meta": {
		"status": 404
	},
	"error": {
		"code": "entity-not-found",
		"message": "An application with the specified ID does not exist."
	}
}
```

### **DELETE `/applications/{id}`**

> Delete an application.

**Request Parameters**

| Field | Type     | Required | Notes                                |
| ----- | -------- | -------- | ------------------------------------ |
| id    | `string` | `true`   | The ID of the application to delete. |

**Response**

Example:

```json
{
	"meta": {
		"status": 204
	}
}
```

**Errors**

_Application Not Found_

```json
{
	"meta": {
		"status": 404
	},
	"error": {
		"code": "entity-not-found",
		"message": "An application with the specified ID does not exist."
	}
}
```

#### **POST `/applications/{id}/issue`**

> Issue a new verifiable presentation. The data passed to the template is the
> contents of the `credentialSubject` field in all the credentials.

**Request Body**

| Field       | Type            | Required | Notes                                                                                                             |
| ----------- | --------------- | -------- | ----------------------------------------------------------------------------------------------------------------- |
| credentials | `array<object>` | `true`   | The credentials to use to create a presentation.                                                                  |
| output      | `enum`          | `true`   | The format in which the presentation should be rendered. Can be one of the following: `svg`, `pdf`, `txt`, `htm`. |

**Response**

Example:

```json
{
	"meta": {
		"status": 201
	},
	"data": {
		"certificate": "<svg ...",
		"presentation": {
			"@context": [
				"https://www.w3.org/2018/credentials/v1",
				"https://www.w3.org/2018/credentials/examples/v1"
			],
			"id": "did:example:mTvrhQMMf6KBEJFItejG0tpohz5U",
			"type": "VerifiablePresentation",
			"verifiableCredential": [
				{
					"@context": [
						"https://www.w3.org/2018/credentials/v1",
						"https://www.w3.org/2018/credentials/examples/v1"
					],
					"id": "did:example:F4RGIuxcKIjygFThqsXW9GX6HocV",
					"type": ["VerifiableCredential", "IdentityCredential"],
					"issuer": "https://example.edu/issuers/565049",
					"issuanceDate": "2010-01-01T19:23:24Z",
					"credentialSubject": {
						"id": "did:example:Fibyu0uFoIHW7Eq3jY7v0rQs5OLb",
						"name": "Happy"
					},
					"proof": {
						"type": "RsaSignature2018",
						"created": "2017-06-18T21:19:10Z",
						"proofPurpose": "assertionMethod",
						"verificationMethod": "https://example.edu/issuers/565049#key-1",
						"jws": "..."
					}
				}
			]
		}
	}
}
```

**Errors**

_Missing/Invalid Payload_

```json
{
	"meta": {
		"status": 400
	},
	"error": {
		"code": "improper-payload",
		"message": "Invalid value provided for field `output`."
	}
}
```
