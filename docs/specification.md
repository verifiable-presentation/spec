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

## APIs

### Template Store

#### **POST `/templates`**

**Request Body**

| Field    | Type     | Required | Notes                                                   |
| -------- | -------- | -------- | ------------------------------------------------------- |
| template | `string` | `true`   |                                                         |
| renderer | `enum`   | `true`   | Can be one of the following: `ejs`, `jstl`, `nunjucks`. |
| schema   | `ajv`    | `false`  | The schema of the data required to render the template. |

Example:

```json
{
	"template": "Hello ${data.name}",
	"renderer": "jstl"
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
		"id": "lFTo9YDjnkV04rcuWQuOg8y93m",
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

#### **GET `/templates/{id}`**

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
		"id": "lFTo9YDjnkV04rcuWQuOg8y93m",
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

### Renderer

#### **POST `/render`**

**Request Body**

| Field    | Type     | Required | Notes                                                                                                    |
| -------- | -------- | -------- | -------------------------------------------------------------------------------------------------------- |
| template | `string` | `true`   | The ID of the template to use to render the data.                                                        |
| data     | `object` | `true`   | The data to render.                                                                                      |
| output   | `enum`   | `true`   | The format in which to output the presentation. Can be one of the following: `svg`, `pdf`, `txt`, `htm`. |

Example:

```json
{
	"template": "lFTo9YDjnkV04rcuWQuOg8y93m",
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

### Registry

#### **GET `/presentations`**

**Request Query Parameters**

| Field     | Type     | Notes                                                                        |
| --------- | -------- | ---------------------------------------------------------------------------- |
| `subject` | `string` | The ID of the subject in the credentials to return. (`credentialSubject.id`) |

Example:

```http
GET /presentations?subject=did%3Aexample%3AksqNItOVL1cAw7qhHXBxt
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
			"id": "http://example.edu/credentials/3732",
			"type": "VerifiablePresentation",
			"verifiableCredential": [
				{
					"@context": [
						"https://www.w3.org/2018/credentials/v1",
						"https://www.w3.org/2018/credentials/examples/v1"
					],
					"id": "http://example.edu/credentials/1872",
					"type": ["VerifiableCredential", "IdentityCredential"],
					"issuer": "https://example.edu/issuers/565049",
					"issuanceDate": "2010-01-01T19:23:24Z",
					"credentialSubject": {
						"id": "did:example:ksqNItOVL1cAw7qhHXBxt",
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
				"verificationMethod": "did:example:BPhmuLkq29AqTbhPSUL3c#keys-1",
				"challenge": "1f44d55f-f161-4938-a659-f8026467f126",
				"domain": "4jt78h47fh47",
				"jws": "..."
			}
		}
	]
}
```

#### **POST `/presentations`**

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
		"id": "http://example.edu/credentials/3732",
		"type": "VerifiablePresentation",
		"verifiableCredential": [
			{
				"@context": [
					"https://www.w3.org/2018/credentials/v1",
					"https://www.w3.org/2018/credentials/examples/v1"
				],
				"id": "http://example.edu/credentials/1872",
				"type": ["VerifiableCredential", "IdentityCredential"],
				"issuer": "https://example.edu/issuers/565049",
				"issuanceDate": "2010-01-01T19:23:24Z",
				"credentialSubject": {
					"id": "did:example:ksqNItOVL1cAw7qhHXBxt",
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
			"verificationMethod": "did:example:BPhmuLkq29AqTbhPSUL3c#keys-1",
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
		"id": "http://example.edu/credentials/3732",
		"type": "VerifiablePresentation",
		"verifiableCredential": [
			{
				"@context": [
					"https://www.w3.org/2018/credentials/v1",
					"https://www.w3.org/2018/credentials/examples/v1"
				],
				"id": "http://example.edu/credentials/1872",
				"type": ["VerifiableCredential", "IdentityCredential"],
				"issuer": "https://example.edu/issuers/565049",
				"issuanceDate": "2010-01-01T19:23:24Z",
				"credentialSubject": {
					"id": "did:example:ksqNItOVL1cAw7qhHXBxt",
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
			"verificationMethod": "did:example:BPhmuLkq29AqTbhPSUL3c#keys-1",
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

#### **GET `/presentations/{id}`**

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
		"id": "http://example.edu/credentials/3732",
		"type": "VerifiablePresentation",
		"verifiableCredential": [
			{
				"@context": [
					"https://www.w3.org/2018/credentials/v1",
					"https://www.w3.org/2018/credentials/examples/v1"
				],
				"id": "http://example.edu/credentials/1872",
				"type": ["VerifiableCredential", "IdentityCredential"],
				"issuer": "https://example.edu/issuers/565049",
				"issuanceDate": "2010-01-01T19:23:24Z",
				"credentialSubject": {
					"id": "did:example:ksqNItOVL1cAw7qhHXBxt",
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
			"verificationMethod": "did:example:BPhmuLkq29AqTbhPSUL3c#keys-1",
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

#### **PUT `/presentations/{id}`**

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
		"id": "http://example.edu/credentials/3732",
		"type": "VerifiablePresentation",
		"verifiableCredential": [
			{
				"@context": [
					"https://www.w3.org/2018/credentials/v1",
					"https://www.w3.org/2018/credentials/examples/v1"
				],
				"id": "http://example.edu/credentials/1872",
				"type": ["VerifiableCredential", "IdentityCredential"],
				"issuer": "https://example.edu/issuers/565049",
				"issuanceDate": "2010-01-01T19:23:24Z",
				"credentialSubject": {
					"id": "did:example:ksqNItOVL1cAw7qhHXBxt",
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
			"verificationMethod": "did:example:BPhmuLkq29AqTbhPSUL3c#keys-1",
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
