# 0x00. Pagination

__`Back-end`__

### Resources
- [REST API Design: Pagination](https://intranet.alxswe.com/rltoken/7Kdzi9CH1LdSfNQ4RaJUQw "REST API Design: Pagination")
- - [HATEOAS](https://intranet.alxswe.com/rltoken/tfzcEbTSdMYSYxsspJH_oA "HATEOAS")

---
## REST API Design: Filtering, Sorting, and Pagination
![rest](https://blog.moesif.com/images/posts/technical/api-design/Filtering-Sorting-and-Pagination.png)

__`Filtering`__
- url parameters are the easiest way to add basic filtering to REST APIs. If you have an /items endpoint you can filter via the property name such as `GET /items?state=active` or `GET /items?state=active&seller_id=1234`. However this only work for exact matches.
- The problem is URL parameters only have a key and a value but filters are composed of three components:
	- The property or field name
	- The operator such as _eq_, _lte_, _gte_
	- The filter value
- There are various ways to encode three components into URL param key/values.
**LHS Brackets**
- one way to encode operators is the use of square brackets []  on the key name e.g `GET /item?price[gte]=10&\\[gte], [exists], [regex], [before] and [after]
- They are a little harder to parse on server side but provides greater flexibility i what the filter vaue is for clients. No need to handle speacial characters differently\
- Benefits is ease use of clients, simple to parse on server side, no need to escape speacial characters

**RHS Colon**
- similar to LHS brackets approach, you can design API to take the operator on the RHS instead of left e.g `GET /items?price=gte:10&price=lte100`
-  Benefits are easier to parse on server side
\
**Search Query Param**
- You can add support for filters and ranges directly with search parameters to search for endpoints, e.g you can search items that contain the term red chair and the  price is gte 10 or lte 100 : `GET /items?q=title:red chair AND price:[10 TO 100]`
- benefits are they are most flexible queries for API users and almost no parsing required on backend.

__`pagination`__
- most endpoints that returns a list of entities need to have some sort of pagination, without pagination a simple search could return millions or even billions of hits causing extrenous network traffic
- Paging requires implied ordering . By default this maybe the items unique identifier but can be any other ordered field such as created date.
**Offset pagination**
- offset paging would look like `GET .items?limit=20&offset=100` This query would return the 20 rows starting with the 100th row\
	Example
	(Assume the query is ordered by created date descending)
	1. Client makes request for most recent items: `GET /items?limit=20`
	2. On scroll/next page, client makes second request `GET /items?limit=20&offset=20`
	3. On scroll/next page, client makes third request `GET /items?limit=20&offset=40`
	As a SQL statement, the third request would look like:

```
SELECT
    *
FROM
    Items
ORDER BY Id
LIMIT 20
OFFSET 40;
```
- benefits are they are the easiest to implement and almost no coding is required other than passing parameters directly to SQL query, stateless on server and works regardless of custom sort_by parameters

**Keyset Pagination**
- They uses the filter values of the last page to fetch the next set of items e.g
(Assume the query is ordered by created date descending)
1. Client makes request for most recent items: `GET /items?limit=20`
2. On scroll/next page, client finds the minimum created date of _2021-01-20T00:00:00_ from previously returned results. and then makes second query using date as a filter: `GET /items?limit=20&created:lte:2021-01-20T00:00:00`
3. On scroll/next page, client finds the minimum created date of _2021-01-19T00:00:00_ from previously returned results. and then makes third query using date as a filter: `GET /items?limit=20&created:lte:2021-01-19T00:00:00`

```
SELECT
    *
FROM
    Items
WHERE
  created <= '2021-01-20T00:00:00'
ORDER BY Id
LIMIT 20
```

- benefits are they work with e\xisting logic with no need to add backend logic
**Seek Pagination**
- its an extension of Keyset paging. By adding an _after_id_ or _start_id_ URL parameter, we can remove the tight coupling of paging to filters and sorting.
- The problem with seek based pagination is it’s hard to implement when a custom sort order is needed. e.g
(Assume the query is ordered by created date ascending)
1. Client makes request for most recent items: `GET /items?limit=20`
2. On scroll/next page, client finds the last id of ‘20’ from previously returned results. and then makes second query using it as the starting id: `GET /items?limit=20&after_id=20`
3. On scroll/next page, client finds the last id of ‘40’ from previously returned results. and then makes third query using it as the starting id: `GET /items?limit=20&after_id=40`
Seek pagination can be distilled into a `where` clause

```sql
SELECT
    *
FROM
    Items
WHERE
  Id > 20
LIMIT 20
```
**Sorting**
- To enable sorting many APIs add a `sort` or `sort_by` URL parameter that can take a field name as the value e.g  `GET /users?sort_by=asc(email)` and `GET /users?sort_by=desc(email)`

**Multi-column sort**
- to encode multi column sort , you could allow multiple field names such as `GET /users?sort_by=desc(last_modified),asc(email)`

---
# HATEOAS
__`Hypermedia as the engine of application state`__
- *Hateoas* is a constrain of the rest software architecture that distinguishes it from other network architectural styles.
- with hateoas a client can interact with a network application whose application servers provide information dynamically through hypermedia
- Restrictions imposed by HATEOAS  decouple client and server enabling server functionality to evolve independently.
