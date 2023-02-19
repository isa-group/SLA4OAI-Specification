# SLA for the OpenAPI Initiative Specification

**SLA4OAS** is an open source standard for describing SLA in APIs.

Based on the standards proposed by the [OpenAPI Initiative](https://openapis.org), SLA4OAS builds on top of the [OpenAPI Specification](https://openapis.org/specification) by defining a specification extension that describes Service Level Agreements (SLAs) for APIs.

This SLA definition in a neutral vendor flavor will allow fostering innovation in the area where APIs expose and documents its SLA,
API Management tools can import and measure such key metrics and composed SLAs for composed services aggregated way in a standard way.

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL"
in this document are to be interpreted as described in [RFC 2119](http://www.ietf.org/rfc/rfc2119.txt).

This specification is an on-going work developed by the [SLA4OAI Technical Committee](https://github.com/isa-group/SLA4OAI-TC) over the course of the last years. In the working discussions a number of practitioners and academics have participated, including: **Refael Botbol** (UP9), **Tim Burks** (Google), **Dave O'Neil** (Apimetrics), **Paul Cray** (Apimetrics), **Hyungjun Lim** (Google), **Khushan Adatiya** (Google), **Antonio Gamez** (Independent),  **Fran Mendez** (AsyncAPI), **Emmanuel Paraskakis** (Smartbear/Independent), **Phil Sturgeon** (Stoplight), **Nikhil Kolekar** (Paypal/Independent), **Marsh Gardiner** (Google), **Prithpal Bhogill** (Google), **Madhurranjan Mohaan** (Google), **Isaac Hepworth** (ex-Google), **Scott Ganyo** (Google), **Kin Lane** (API Evangelist), **Mike Ralphson** (Independent), **Jeffrey ErnstFriedman** (OpenTravel Alliance), **Antonio Ruiz-Cortes** (ISA Group/University of Sevilla), **Pedro J. Molina** (Metadev/ISA Group) and **Pablo Fernandez** (ISA Group/University of Sevilla).

## Revision History

|Version  | Date         | Notes            |
|:------- |:------------ |:---------------- |
| 1.0.0   | 2020-11-16   | Initial draft.   |
| 1.0.1   | 2022-06-08   | Renamed SLA Object's `sla` field to `sla4oas`.<br>Added field `plan` to the SLA Object (must be used in SLAs of type Agreement).<br>Added fields `quotas` and `rates` to the SLA Object (can be used in SLAs of type Plans).<br>Added fields `type`, `customer`, `apikeys` and `validity` to the Context Object.<br>Added field `name` to the Plan Object. |


## Extension Format

SLA extension can be mapped in an external YAML or JSON file. For the sake of clarity throughout this document YAML will be used in examples. The OpenAPI specification main document (inside the `info` section) can link to SLA definitions using the standard extension and `$ref` linking mechanism:

```yaml
openapi: 3.1
info:
  x-sla:
    $ref: ./sla.yml
```

## Lifecycle of Agreement

The lifecycle of any SLA agreement has two phases:

1. The service provider designs the levels (plans) of the service, then defines the **SLA Plans** document. See an example at [petstore-plans.yml](./samples/petstore/petstore-plans.yml).
2. The customer chooses the levels (plans) they need, then both provider and customer agree on the **SLA Agreement** document. See an example at [pro-petstore-sla.yml](./samples/petstore/pro-petstore-sla.yml).


##  Specification

A **SLA4OAS** document is built as an extension to a given [OAS](https://github.com/OAI/OpenAPI-Specification) document. This extension will add an SLA definition to all or part of the services exposed in the API by means of different plans that include limitations over the usage of the API. Specifically, a full SLA definition is a `YAML` (or `JSON`) document composed of the structure of an [SLA Object](#sla-object).


**Simple Example:**

```yaml
sla4oas: 1.0.0
context:
  id: petstore-sample
  type: plans
  api:
    $ref: ./petstore-service.yml
  provider: ISAGroup
metrics:  
  requests:
    type: integer
    format: int64
    description: "Number of requests"
plans:
  base:
    availability: R/00:00:00Z/23:00:00Z
  free:
    rates:
      /pets/{id}:
        get:
          requests:
            - max: 1
              period: second
    quotas:
      /pets:
        post:
          requests:
            - max: 10
              period: minute
  pro:
    pricing:
      cost: 5
      currency: EUR
      billing: monthly  
    rates:
      /pets/{id}:
        get:
          requests:
            - max: 100
              period: second
```

### SLA Object

The SLA Object represents the top placeholder for the SLA Document.


| Field Name | Type                                  | Description  |
| :--------- | :------------------------------------ | :----------- |
| sla4oas    | `string`                              | **Required** Identifies the version of the SLA4OAS used. |
| context    | [`ContextObject`](#1-context-object)  | **Required** Holds the main information of the SLA context.  |
| metrics    | [`MetricsObject`](#2-metrics-object)  | **Required** A list of metrics to use in the context of the SLA.  |
| plan       | [`PlanObject`](#31-plan-object)       | **Optional** Describes a usage plan for the API with its associate costs and availability. |
| plans      | [`PlansObject`](#3-plans-object)      | **Optional** A set of plans to define different service levels per plan. |
| quotas     | [`QuotasObject`](#312-quotas-object)  | **Optional** Specific quotas data for this plan. |
| rates      | [`RatesObject `](#313-rates-object)   | **Optional** Specific rates data for this plan. |

While `plan`, `plans`, `quotas` and `rates` are optional, there are some considerations:

- SLA Plans (i.e. `context.type` is set to `plans`):
  - Must have **only** one of the following:
    - `plans`
    - `quotas` and/or `rates`
- SLA Agreement (i.e. `context.type` is set to `agreement`):
  - Must have `plan`.
  - Cannot have `plans`, `quotas` or `rates`.

### 1. Context Object

The Context Object provides information about the general context of the actual SLA.

| Field Name   | Type        | Description  |
| :----------- | :-----------| :----------- |
| id           | `string`    | **Required** The identification of the SLA context.  |
| api          | `uri`       | **Required** Indicates a URI (absolute or relative) describing the API, described in the OpenAPI format, to be instrumented. If unspecified, the associated API is the one defined by the referring OpenAPI specification main document. |
| type         | `string`    | **Required** The type of SLA based on the [Lifecycle of Agreement](#lifecycle-of-agreement) (`plans` or `agreement`). |
| provider     | `string`    | **Required** Provider information: data about the owner/host of the API. |
| customer     | `string`    | **Optional** Customer information, data about the entity that consumes the service. This field is **required** if the context type is `agreement`. |
| apikeys      | `array`     | **Optional** List of apikeys valid for authentication of API calls. This field can only be used if the context type is `agreement`. |
| validity     | [`ValidityObject `](#11-validity-object)     | **Optional** Availability of the service expressed via time slots. This field can only be used if the context type is `agreement`. |


**Example:**

```yaml
context:
  id: petstore-sample
  api:
    $ref: ./petstore-service.yml
  type: agreement
  provider: ISAGroup
  customer: tenant1
```

#### 1.1 Validity Object

Holds the availability of the service expressed via time slots.

| Field Name     | Type             | Description  |
| :------------- | :--------------- | :----------- |
| from           | `string`         | **Optional** The starting date of the SLA agreement using the [ISO 8601](https://www.w3.org/TR/NOTE-datetime) time intervals format. |
| to             | `string`         | **Optional** The expiration date of the SLA agreement using the [ISO 8601](https://www.w3.org/TR/NOTE-datetime) time intervals format. |

**Example:**

```yaml
validity:
  from: "2022-11-23T20:20:40+00:00"
  to: "2023-11-23T20:20:40+00:00"
```


### 2. Metrics Object

Metrics sections describes the relevant technical indicators and business metrics for the SLA. There could be pure technical ones like *number of requests*, *response time* or *throughput* but also related to the business model such as *data export formats* or *number of persistent resources*.


| Field Name     | Type                                  | Description  |
| :------------- | :------------------------------------- | :----------- |
| {name}         | [`MetricObject`](#21-metric-object)    | **Optional** Definitions of metrics with name, types and descriptions. |
| {name}         | `object {"$ref": uri}`                 | **Optional** Reference to pre-existing metrics in an external file. |

**Example:**

```yaml
metrics:
  animalTypes:
    type: integer
    format: int64
    description: Number of different animal types.
  requests:
    $ref: ./metrics.yml#request
```

In the above example, we referred to a pre-existing metrics `metrics.yml` which has some predefined metrics like `requests` and `responseTime`.


#### 2.1 Metric Object

Contains a definition of a metric.

| Field Name     | Type      | Description   |
| :------------- | :-------- | :------------ |
| type           | `string`  | **Required** This is the metric type accordingly to [the OAI spec defined types](https://github.com/OAI/OpenAPI-Specification/blob/master/versions/3.0.2.md#dataTypes). |
| format         | `string`  | **Optional** The extending format for the previously mentioned type. See [Data Type Formats](https://github.com/OAI/OpenAPI-Specification/blob/master/versions/3.0.2.md#dataTypeFormat) for further details. |
| description    | `string`  | **Optional** A brief description of the metric. |

**Example:**

```yaml
animalTypes:
  type: integer
  format: int64
  description: Number of different animal types.
```

### 3. Plans Object

Contains a list of plans describing different levels of service and prices.

| Field Pattern  | Type                                            | Description  |
| :------------- | :------------------------------ | :----------- |
| {planName}     | [`PlanObject`](#31-plan-object) | Describes a usage plan for the API with its associate costs and availability. |

**Example:**

```yaml
plans:
  base:
    availability: R/00:00:00Z/23:00:00Z
  free:
    rates:
      /pets/{id}:
        get:
          requests:
            - max: 1
              period: second
  pro:
    pricing:
      cost: 50
    quotas:
      /pets:
        get:
          requests:
            - max: 20
              period: second
```

The `base` keyword is proposed as the common base plan. Any elements (*availability*, *pricing*, *quota* or *rate*) in a base plan apply to all plans. Consequently, a specific plan inherits definitions from `base` but it can override any element to make it more concrete or relaxed.


#### 3.1 Plan Object

Describes a plan in full.

| Field Name     | Type                                   | Description  |
| :------------- | :------------------------------------- | :----------- |
| availability   | `string`                               | **Optional** Availability of the service for this plan expressed via time slots using the ISO 8601 time intervals format. |
| name           | `string`                               | **Optional** The name of the plan. |
| pricing        | [`PricingObject`](#311-pricing-object) | **Optional** Specific pricing data for this plan. |
| quotas         | [`QuotasObject`](#312-quotas-object)   | **Optional** Specific quotas data for this plan. |
| rates          | [`RatesObject `](#313-rates-object)    | **Optional** Specific rates data for this plan. |

**Example:**

```yaml
pro:
  availability: R/00:00:00Z/23:00:00Z
  pricing:
    cost: 50
  quotas:
    /pets:
      get:
        requests:
          - max: 20
            period: second
```

#### 3.1.1 Pricing Object

Describes the general information of the pricing of the a given plan.

| Field Name     | Type          | Description  |
| :------------- | :------------:| :------------|
| cost           | `number`      | **Optional** Units of cost associated with this service. Defaults to `0` if unspecified. |
| currency       | `string`      | **Optional** Currency used to express the cost. Supported currency values are expressed in [ISO 4217](https://en.wikipedia.org/wiki/ISO_4217) format. Samples: `USD`, `EUR`, or `BTC` for US dollar, euro, or bitcoin, respectively. Defaults to `USD` if unspecified. |
| billing        | `string`      | **Optional** Billing frequency. Possible values are: `onepay`, an unique payment before start using the service; `daily` , billed every day; `weekly`, billed every week; `monthly`, billed every month; `quarterly`, billed every quarter; `yearly`, billed every year. If it is not specified, the default value assumed is `monthly`|

**Example:**

```yaml
pricing:
  cost: 5
  currency: "EUR"
  billing: "monthly"
```


#### 3.1.2 Quotas Object

 Quotas are limits computed in **static** time windows independent from the usage of the service; as an example, a *daily quota* would be reset every midnight (in the given timezone where the service is operated).

 The *Quotas Object* is composed of a map from name to PathObject describing the quota limits for the service on the current plan.

| Field Pattern  | Type                              | Description  |
| :------------- | :-------------------------------- | :----------- |
| {pathName}     | [`PathObject`](#314-path-object) | **Required** Describes the API endpoint path quota configurations. |

**Example:**

```yaml
quotas:
  /pets:
    post:
      requests:
        - max: 100
          period: hour
```

Path names can include the special path name "default", which can be used to specify quotas for all paths that aren't otherwise specified.

#### 3.1.3 Rates Object

Rates are limits computed in **dynamic** time windows relatives to the usage of the service; as an example, a *rate per minute* refers to the period of 60 seconds immediately before the last operation of each API customer.

In a similar way to quotas, the Rates Object contains a map from name to PathObject describing the rate limits for the service on the current plan.

| Field Pattern  | Type                             | Description  |
| :------------- | :------------------------------- | :----------- |
| {pathName}     | [`PathObject`](#314-path-object) | **Required** Describes the API endpoint path rate configurations. |


**Example:**

```yaml
rates:
  /pets/{id}:
    get:
      requests:
        - max: 5
          period: second
```

Path names can include the special path name "default", which can be used to specify rate limits for all paths that aren't otherwise specified.

#### 3.1.4 Path Object

The API endpoint path.

| Field Pattern  | Type                                       | Description  |
| :------------- | :----------------------------------------- | :----------- |
| {methodName}   | [`OperationObject`](#3141-operation-object) | **Required** the operations attached to this path. |


**Example:**

```yaml
/pets/{id}:
  post:
    requests:
      - max: 1000
        period: day
```

#### 3.1.4.1 Operation Object

The operations attached to the path.

| Field Pattern  | Type                                               | Description  |
| :------------- | :------------------------------------------------- | :----------- |
| {metricName}   | [`LimitObject`](#31411-limit-object) | **Required** The allowed limits of the request. |


**Example:**

```yaml
get:
  requests:
    - max: 20
      period: minute
```

#### 3.1.4.1.1 Limit Object

The allowed limits of the metric (e.g. *requests*).

| Field Pattern  | Type       | Description  |
| :------------- | :--------- | :------------|
| max            | `number`   |  **Required** Maximum value that can be reached so the limit is not violated. |
| period         | `string`   |  **Optional** The period specified for the given limit; it should be one of the following possible values: `second`, `minute`, `hour`, `day`, `month` or `year`. In case it is not specified, it would be a **permanent** limit over the metric. |
| type           | `string`   |  **Optional** Specifies how the API handles requests to the given endpoint, can be one of `burst` or `average` (for further explanation see below). The default value is `burst`. |

The difference between `burst` and `average` is better explained with an example. Supposing an endpoint that allows 10 requests per minute:

- `average`: Would allow an average of 10 requests per minute, meaning only one request every 6 seconds would be accepted. Any other request received within these 6-second intervals would be rejected with HTTP code 429. 
- `burst`: Would allow all 10 requests to be served at the any time during the minute. Once 10 requests have been served and until the end of the minute, all successive requests would be rejected with HTTP code 429.

**Example:**
```yaml
max: 2
period: second
type: burst
```

## References

1. An Analysis of RESTful APIs Offerings
in the Industry. A. Gamez-Diaz(B), P. Fernandez, and A. Ruiz-Cortes. In M. Maximilien et al. (Eds.): ICSOC 2017, LNCS 10601, pp. 589–604, 2017. DOI: [10.1007/978-3-319-69035-3_43](https://doi.org/10.1007/978-3-319-69035-3_43)

2. The Role of SLA in the API Industry. SLA4OAI-TC Members. [10.1145/3338906.3340445](https://doi.org/10.1145/3338906.3340445)

3. ISO-8601. Data elements and interchange formats – Information interchange – Representation of dates and times [ISO-8601](https://en.wikipedia.org/wiki/ISO_8601)
