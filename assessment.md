Meridian Orders API — Product Analyst Assessment



Task 1 — What doesn't match?



1\. Pagination



\* Docs: `has\_more` tells the client whether another page exists.

\* Data: `orders\_page1.json` has `has\_more: false` but also provides `next\_cursor: "cur\_8f2a19bd"`. `orders\_page2.json` contains two more orders.

\* Impact: A client following the docs would stop after page 1 and miss orders, causing incomplete reports and revenue totals.



2\. Monetary units



\* Docs: Monetary amounts are integers in the smallest currency unit.

\* Data: `orders\_page2.json`, `ord\_1006`, contains decimal values: `subtotal: 44.0`, `tax: 3.63`, `shipping: 5.99`, `total: 53.62`.

\* Impact: A client cannot safely assume one consistent money format, which can cause incorrect financial calculations.



3\. Total calculation



\* Docs: `total` always equals `subtotal + tax + shipping`.

\* Data: For `ord\_1004`, 6200 + 511 + 599 = 7310, but `total` is 6810 — a $5.00 difference.

\* Impact: Systems calculating totals from the documented formula can disagree with the API's `total`.



4\. Undocumented status



\* Docs: Status is one of `pending`, `shipped`, `delivered`, or `cancelled`.

\* Data: `ord\_1003` has status `refunded`.

\* Impact: Clients validating allowed statuses may reject or mishandle this order, and refund treatment is undefined.



5\. Customer email can be null



\* Docs: Customer email is always present.

\* Data: `orders\_page2.json`, `ord\_1005`, has `email: null`.

\* Impact: Downstream systems expecting a string may fail validation or processing.



6\. Missing order returns 200



\* Docs: `GET /v1/orders/{id}` returns `404` when an order does not exist.

\* Data: `order\_ord\_9999.json` represents a nonexistent order but the README records an HTTP 200 response with `{"order": null}`.

\* Impact: Clients relying on HTTP status codes may treat a missing order as a successful request.



Worst issue — Pagination



Pagination is the most serious because it can silently truncate the dataset. A client can receive a successful response, believe it has all orders, and produce incomplete financial reports without realizing records were missed.



Task 2 — What's the total revenue?



The six returned order totals add up to $328.03 gross order value.



I interpreted `ord\_1006`'s decimal values as dollars because its components sum to $53.62, despite the documentation specifying integer minor units.



I used the returned `total` for `ord\_1004` rather than silently correcting it, but its components imply $73.10 while the API reports $68.10.



I cannot determine recognized/net revenue from the supplied data because the treatment and amount of the refunded `ord\_1003` are not defined. I would need Meridian's refund/revenue rules or the dashboard calculation logic.



Task 3 — Reply to Priya



Hi Priya,



Thanks for flagging this. We reviewed the orders returned by the API and found several inconsistencies that can explain the reconciliation difference.



Most importantly, the first page reports that there are no more pages even though a second page contains two additional orders. A report following the API response could therefore miss those orders. We also found inconsistent money formatting, a $5.00 mismatch between order components and the reported total for one order, and a refunded order whose treatment in revenue calculations is not documented.



The six returned order totals add up to $328.03 as gross order value. However, we cannot determine the correct recognized revenue from the supplied data because the treatment of the refunded order is undefined.



To reconcile this with the Meridian dashboard, we would need the dashboard's revenue rules, including how refunds are handled.



Best regards,

Adam Albinus



Task 3 — Bug Report



&#x20;Bug: Orders API reports no more pages when another page exists



What to look at: `GET /v1/orders` — `orders\_page1.json`



Actual behavior: The response returns HTTP 200 with `has\_more: false`, but also provides `next\_cursor: "cur\_8f2a19bd"`. A second request using that cursor returns `orders\_page2.json`, containing `ord\_1005` and `ord\_1006`.



Expected behavior: When another page exists, `has\_more` should be `true` and `next\_cursor` should provide the cursor needed to retrieve it. `has\_more` should only be `false` on the final page.



Impact: A client following the documented pagination logic will stop after the first page and miss valid orders, producing incomplete order lists and potentially incorrect financial reports.



Suggested fix: Make `has\_more` accurately reflect whether another page exists and ensure `next\_cursor` is consistent with it. Add tests covering first, middle, and final pages.

