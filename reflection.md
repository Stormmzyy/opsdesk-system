# Reflection

## The hardest bug I fixed

The hardest bug was one that nothing was failing on. Every error my checks provoked was one the application expected, and the logs looked clean. So I added a CHECK constraint the code knew nothing about, and broke it on purpose. PostgreSQL's "Failing row contains (…)" line then put the entire row into the log. It was a ticket's row, but the same failure on the users table would have logged an email address and a password hash. My first fix, raising Hibernate's error logger, would have hidden every other SQL error and still left the row in the 500 handler's line. The real fix was one setting on the driver, `logServerErrorDetail=false`, which keeps the error and the constraint's name and drops only the values. The hard part was finding it, not fixing it: a leak like that only shows when you force the error nobody planned for.

## One thing I'd design differently

The Edit page. It sent the whole form on every save, so someone fixing a typo on a page they opened ten minutes earlier would silently put back a status a colleague had just changed. The server can't fix that on its own, because it can't tell "I want this value" from "this was the value when my page loaded". They are the same bytes. `@Version` only catches two requests that overlap. The Edit page now sends only the fields that changed, but any client that sends the whole form, Swagger's "Try it out" included, reopens the hole. I would design edits as changes from the start, and decide on day one whether the client sends back the version it read.

## One thing I now understand that I didn't eight weeks ago

That a document about an API is a claim, including the one I wrote myself. When I compared the generated Swagger page with the contract I wrote in Week 4, the contract was right six times out of eight: Swagger typed the ticket list as a bare object, marked no response field as required, and showed none of the defaults. But the contract lost twice, both times to my own lines that the code had outgrown. Path ids are strings, not integers, and the setup conventions were out of date. Making Swagger match the contract on the id would have been backwards. Neither document decides. The running code does, and what keeps both documents honest is the tests that compare them with it.
