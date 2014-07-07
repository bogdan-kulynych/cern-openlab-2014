Implementing Memento HTTP API for Invenio Digital Library Software
==================================================================

Bogdan Kulynych `bogdan.kulynych@gmail.com`

*Supervisor*
Tibor Šimko `tibor.simko@cern.ch`

### Project Specification
Implement an [RFC 7089](http://www.mementoweb.org/guide/rfc/) Memento API: `TimeGate` and `TimeMap` endpoints for Invenio digital library. Implement an HTTP API and Web UI for browsing historical versions of Invenio records.

### Abstract
Memento is a technical framework for browsing and discovering of the historical versions of Web resources. This project aims to implement Memento API for Invenio digital library software, as well as the HTTP API and Web UI to facilitate and complement the Memento API.

### Introduction
Let $R$ be original library record (resource), $G_R$ the *TimeGate* endpoint for datetime negotiation, $T_R$ the *TimeMap* endpoint for discovering the historical versions of $R$, $M_R(D)$ the `Memento`, a historical version of $R$ corresponding to the datetime $D$. Denote $\text{UA}$ the HTTP User-Agent. As per the RFC, Memento [pattern 2.1](http://www.mementoweb.org/guide/rfc/#Pattern2.1) API is defined by following partial HTTP requests/responses:

#### $G_R$ discovery for given $R$
```
UA --- HTTP HEAD/GET ---------------------------------------> URI-R
UA <-- HTTP 200; Link: URI-G -------------------------------- URI-R
```
$$
\text{UA}\xrightarrow{\text{HTTP HEAD/GET}}\text{URI}(R) \\
\text{UA}\xleftarrow[\text{Link: URI}(G_R)]{\text{HTTP 200}}\text{URI}(R)
$$

#### Datetime negotiation for given $G_R$
```
UA --- HTTP HEAD/GET; Accept-Datetime: D -------------------> URI-G
UA <-- HTTP 302; Location: URI-M; Vary; Link:
       URI-R,URI-T -----------------------------------------> URI-G
UA --- HTTP GET URI-M; Accept-Datetime: D ------------------> URI-M
UA <-- HTTP 200; Memento-Datetime: D; Link:
       URI-R,URI-T,URI-G ------------------------------------ URI-M
```

$$
\text{UA}\xrightarrow{\text{HTTP HEAD/GET; Accept-Datetime: }D}\text{URI}(G_R) \\
\text{UA}\xleftarrow[\text{ Vary; Link: URI}(R), \text{URI}(T_R)]{\text{HTTP 302; Location: URI}(M_R(D))}\text{URI}(G_R)
$$
$$
\text{UA}\xrightarrow{\text{HTTP GET; Accept-Datetime: }D}\text{URI}(M_R(D)) \\
\text{UA}\xleftarrow[\text{Link: URI}(R),\text{URI}(T_R),\text{URI}(G_R)]{\text{HTTP 200; Memento-Datetime: }D'}\text{URI}(M_R(D)) \\
$$

#### $T_R$ discovery for given $G_R$
```
UA --- HTTP HEAD/GET ---------------------------------------> URI-G
UA <-- HTTP 302; Vary; Link:
       URI-R,URI-T -----------------------------------------> URI-G
```

$$
\text{UA}\xrightarrow{\text{HTTP HEAD/GET}}\text{URI}(G_R) \\
\text{UA}\xleftarrow[\text{Link: URI}(R),\text{URI}(T_R)]{\text{HTTP 302}}\text{URI}(G_R) \\
$$

#### Discovery of the historical versions for given $T(R)$
```
UA --- HTTP GET URI-T --------------------------------------> URI-T
UA <-- HTTP 200 [*] ----------------------------------------- URI-T
```

$$
\text{UA}\xrightarrow{\text{HTTP HEAD/GET}}\text{URI}(T_R) \\
\text{UA}\xleftarrow{\text{HTTP 200}\text{[*]}}\text{URI}(T_R) \\
$$

where the [\*] response body may come in two forms. If the number of mementos is smaller than *N*, then it is in the form [1]:
```
<URI-R>;rel="original",
<URI-T_R>
  ; rel="self"
  ; type="application/link-format"
  ; from="D[start]"
  ; until="D[end]",
<URI-G_R>
  ; rel="timegate",
<URI-M_R(D_start)>
  ; rel="first memento"
  ; datetime="D_start"
  ; license="URI-LICENSE"
<URI-M_R(D_end)>
  ; rel="last memento"
  ; datetime="D_end"
  ; license="URI-LICENSE"
<URI-M_R(D_i)>
  ; rel="memento"
  ; datetime="D_i"
  ; license="URI-LICENSE"
...
```

Otherwise, [\*] response body is an index response that provides links to paginated $T_R^k$, and is in form [2]:

```
<URI-R>;rel="original",
<URI-G_R>
  ; rel="timegate",
<URI-T_R^k>
  ; rel="self";type="application/link-format"
  ; from="D_start_k"
  ; until="D_end_k",
...
```

The `GET/HEAD` request on $\text{URI}(T_R^k)$ is responded with `HTTP 200` with response body in form [1].

#### HTTP Headers format

- All dates $D$ in `Accept-Datetime` and `Memento-Datetime` are to be in standard [HTTP-date form](http://tools.ietf.org/html/rfc2616):

```
Accept-Datetime: Wed, 30 May 2007 18:47:52 GMT
```
- `Vary` header may only have the `accept-datetime` value for Invenio
- `Link` header is to be of the following format:

```
 <URI-R>
   ; rel="original",
 <URI-G_R>
   ; rel="timegate",
 <URI-T_R>
   ; rel="timemap";
   ; type="application/link-format"
   ; from="D[start]"
   ; until="D[end]",
 <URI-M_R(D)>
   ; rel="memento"
   ; datetime="D"
  
```

Detailed specification can be found in [RFC 1123](http://www.mementoweb.org/guide/rfc/#RFC1123).
#### Memento Details:

- [Intro](http://www.mementoweb.org/guide/quick-intro/)
- [RFC 7089](http://www.mementoweb.org/guide/rfc/)


### Invenio Memento API

The proposed Memento API for Invenio defines Memento endpoints as follows:

$$
\begin{align}
\text{URI}(R) &= {\tt record/}\text{ID}(R){\tt /} \\
\text{URI}(M_R(D)) &= {\tt record/}\text{ID}(R){\tt /?revision= }\ \text{REV}(D') \\
\text{URI}(G_R) &= {\tt timegate/record/timegate/}\text{ID}(R) \\
\text{URI}(T_R) &= {\tt timemap/record/}\text{ID}(R) \\
\text{URI}(T_R^k) &= {\tt timemap/}k{\tt /record/}\text{ID}(R),
\end{align}
$$
where $D' = \text{sup} \{D_i~|~D_i<D, D_i - \text{dates of revisions of }R\}$ is the closest smaller than $D$ date of a revision of $R$.

Additionally, sections for citations, comments, files, etc. found under ${\tt record/}\text{ID}(R){\tt /}\text{section}{\tt/}$ when accessed with ${\tt ?revision=}\ \text{REV}(D')$ parameter are to be displayed in a historical state as closely resembling their state as of revision $\text{REV}(D')$ as possible.

### Implementation

The `memento` module responsible for `timegate` and `timemap` endpoints is to be developed. The `records` module responsible for the `records` endpoint is to be extended with the functionality defined above.

#### Versioning discussion

