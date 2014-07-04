Implementing Memento HTTP API for Invenio Digital Library Software
==================================================================

### Project Specification
Implement an [RFC 7089](http://www.mementoweb.org/guide/rfc/) Memento API: `TimeGate` and `TimeMap` endpoints for Invenio digital library. Implement an HTTP API and Web UI for browsing historical versions of Invenio records.

### Abstract
Memento is a proposed technical framework for browsing and discovery of the historical versions of Web resources. This project aims to implement Memento API for Invenio digital library software, as well as the HTTP API and Web UI, as to facilitate and complement the Memento API.

### Introduction
Let `R` be original library record (resource), `G(R)` the `TimeGate` endpoint for datetime negotiation, `T(R)` the `TimeMap` endpoint for discovering the historical versions of `R`, `M(R, D)` the `Memento`, a historical version of `R` corresponding to the datetime `D`. As per the RFC, [pattern 2.1](http://www.mementoweb.org/guide/rfc/#Pattern2.1) Memento API is defined by following partial HTTP requests/responses:

#### `G` discovery for given `R`
```
UA --- HTTP HEAD/GET ---------------------------------------> URI-R
UA <-- HTTP 200; Link: URI-G(R) ----------------------------- URI-R
```

#### Datetime negotiation for given `G(R)`
```
UA --- HTTP HEAD/GET; Accept-Datetime: D -------------------> URI-G(R)
UA <-- HTTP 302; Location: URI-M(R, D); Vary; Link:
       URI-R,URI-T(R) --------------------------------------> URI-G(R)
UA --- HTTP GET URI-M(R, D); Accept-Datetime: D ------------> URI-M(R, D)
UA <-- HTTP 200; Memento-Datetime: D; Link:
       URI-R,URI-T(R),URI-G(R) ------------------------------ URI-M(R, D)
```

#### `T` discovery for given `G(R)`
```
UA --- HTTP HEAD/GET ---------------------------------------> URI-G(R)
UA <-- HTTP 302; Vary; Link:
       URI-R,URI-T(R) --------------------------------------> URI-G(R)
```

#### Discovering the historical versions for given `T(R)`
```
UA --- HTTP GET URI-T(R)------------------------------------> URI-T(R)
UA <-- HTTP 200 [*]------------------------------------------ URI-T(R)
```
Where the `[*]` response body may come in two forms. If the number of mementos is smaller than *N*, then it is in the form [1]:
```
<URL-R>;rel="original",
<URL-T(R)>
  ; rel="self";type="application/link-format"
  ; from="D[start]"
  ; until="D[end]",
<URL-G(R)>
  ; rel="timegate",
<URL-M(R, D[start])>
  ; rel="first memento"
  ; datetime="D[start]"
  ; license="URL-LICENSE"
<URL-M(R, D[end])>
  ; rel="last memento"
  ; datetime="D[end]"
  ; license="URL-LICENSE"
<URL-M(R, D[i])>
  ; rel="memento"
  ; datetime="D[i]"
  ; license="URL-LICENSE"
...
```

Otherwise, `[*]` response body is in form [2], and it is an index providing links to paginated `T(R)[k]`:

```
<URL-R>;rel="original",
<URL-G(R)>
  ; rel="timegate",
<URL-T(R)[k]>
  ; rel="self";type="application/link-format"
  ; from="D[start]"
  ; until="D[end]",
...
```

The `GET/HEAD` request on `URL-T(R)[k]` is responded with `HTTP 200` of form [1]

#### Memento Details:

- [Guide](http://www.mementoweb.org/guide/quick-intro/)
- [RFC](http://www.mementoweb.org/guide/rfc/)