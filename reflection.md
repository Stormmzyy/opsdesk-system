**The hardest bug I fixed**

It was a bug nothing was failing on. I had spent an afternoon searching the logs for passwords, tokens and email addresses and found none, but every request I had made was one the application expected. So I added a CHECK constraint the code knew nothing about and broke it on purpose. PostgreSQL answered with "Failing row contains (...)" and put the whole row into the log. It was a ticket's row that time. The same failure on the users table would have put an email address and a password hash there.

My first fix was wrong. I raised Hibernate's error logger so it would stop printing, which worked and also silenced every other SQL error it reports: deadlocks, lost connections, timeouts. I only found that out by reading the class, because nothing failed. The real fix was one setting on the PostgreSQL driver, `logServerErrorDetail=false`, which keeps the error and the constraint name and drops the values. The hard part was never the fix. It was that nothing would have shown me the leak unless I went looking for the error nobody had planned for.

**One thing I would design differently**

The edit form. It sent every field on every save, so someone correcting a typo on a page they had opened ten minutes earlier would quietly put back a status a colleague had changed in the meantime. I spent a while assuming this was something I could fix on the server, and it is not. When the body says the status is OPEN, the server cannot tell whether I want it to be OPEN or whether it merely was OPEN when my page loaded. Those are the same bytes.

I added a version column, which catches two requests that genuinely overlap, and that is all it catches. The form now sends only the fields that changed, which fixes it for my frontend and for nothing else: any client that sends the whole form, Swagger's own Try it out included, opens the hole again. If I started again I would treat an edit as a description of what changed, and decide on the first day whether the client sends back the version it read.

**One thing I now understand that I did not eight weeks ago**

That a document describing an API is a claim, including one I wrote myself. When I compared the generated Swagger page against the contract I wrote in Week 4, the contract was right six times out of eight. Swagger typed the ticket list as a bare object, marked none of the response fields as required and showed none of the defaults. But twice the contract lost, and both times to lines I had written in Week 4 that the code had since outgrown. Making Swagger agree with the contract on those would have been the wrong direction.

Neither document decides anything. The running code does. What keeps either of them honest is the tests that compare them against it, which is why twenty-three of the contract's examples are now pinned by tests that fail when the code drifts.
