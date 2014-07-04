Implementing Memento HTTP API for Invenio Digital Library software
==================================================================

### Project Specification
Implement an [RFC 7089](http://www.mementoweb.org/guide/rfc/) Memento API: `TimeGate` and `TimeMap` endpoints. Implement an HTTP API and Web UI for browsing historical versions of digital library records as to facilitate and complement the Memento API.

### Abstract


### Introduction
Let `R` be original library record (resource), `G(R)` the `TimeGate` endpoint for datetime-version negotiation, `M(R)` the `TimeMap` endpoint for discovering the historical versions of `R`. As per the RFC, Memento API is defined by following partial HTTP requests/responses:

#### `TimeGate` discovery for given `R`
```
UA --- HTTP HEAD/GET ------------------------------------------> URI-R
UA <-- HTTP 200; Link: URI-G(R) -------------------------------- URI-R
```

#### `TimeMap` discovery for given `G(R)`
```
UA --- HTTP HEAD/GET ------------------------------------------> URI-G(R)
UA <-- HTTP 302; Location: URI-M; Vary; Link: URI-R,URI-T(R) --> URI-G(R)
UA --- HTTP GET URI-T(R)---------------------------------------> URI-T(R)
UA <-- HTTP 200 ------------------------------------------------ URI-T(R)
```

#### Details:

- [Guide](http://www.mementoweb.org/guide/quick-intro/)
- [RFC](http://www.mementoweb.org/guide/rfc/)