# Subscriptions
Proposal for extending existing documents API to support subscriptions to reports.

---
## Supporting Files

- Assumptions made are outlined in [assumptions.md](assumptions.md).
- An explanation of tools used explained in [tools.md](tools.md).
- [Flow chart](reeport-flow.png) given in PNG format, with [xml](florence.xml) provided for easy import into draw.io.
- API documentation done using swagger. The [swagger.yaml](swagger.yaml) file can be imported into [editor.swagger.io](https://editor.swagger.io/) for easy viewing.

---
## Summary

The subscription process, outlined in the swagger doc, is fairly straightforward -- attaching or detaching a user from a given report subject.

The meat of the design is in the report generator itself. The assumptions doc explains that it is assumed reports are intended to be generated based on individual clinical trials. 

While it is currently defined to be kicked off via a scheduler, it has been built in a way that should allow for arbitrary and even "on demand" report generation. 

The scheduler initiates a kind of orchestrator that will tap into the (new) Subscriptions data store, retrieving a list of all trials that need reports.

It then kicks off one report generator process for each trial, passing in the trial_id as the parameter. This is for scalability as well as speculation for expansion. Being able to see how long a given report takes to generate will help understand compute/billing impact, which would have sprawling implications for things like different cost/feature levels for clients in the future. It's also inevitable that users will ask for feasibility of "on demand" reports, which this will also help us understand.

--

The individual report generator will reach into the document store and perform the ambiguous "data aggregation" and "image processing". If possible, we should look into whether the latter can be further offloaded to background processes for scalability purposes.

Depending on the chosen PDF compilation solution, it may be prudent / necessary to separate that step, and here place the report generator's results in some temp location for the PDF'er to composite (in case business solutions push us to change PDF tools in the future). 

The resulting compiled PDF will be dropped into an S3 bucket. This allows the process to be somewhat asynchronous, but more importantly allows for resending reports without needing to spend compute regenerating them. It seems prudent for the path to follow the format "/{trial_id}/{scope}/{timestamp}.pdf", with the scope likely being a time scope (and could just be a datestamp, implying "all data up to the provided date / time") with the timestamp ideally being selected AFTER the files to aggregate have been selected. This will provide a slight assist to any debugging / troubleshooting, as it will be easy to tell (for instance) that a file has been missed if the file's created stamp in S3 is before the report's timestamp.

The apparition of the file in the S3 bucket can then be configured to trigger a final process: the report sender. This will use the trial_id from the evented file path, retrieve the list of subscribed users from the subscription store, and bounce that through the user store to resolve a list of destination e-mail addresses. The process then simply loads a template, attaches the report from the file generation event, and sends it out to the resolved distribution list.

---

## Final notes / considerations

Though a "sliced up" application as shown in the diagram does support asynchronous computing, flexibility for requirements changing in the future, and some measure of "single responsibility" programming, care was first and primarily taken to reduce the number of systems and processes that have access to specific data. 

This allows us to insulate the 'responsibilities' (processes) that have access to respective data stores.

Further insulation could be accomplished by having the report sender make a call to an API process 'guarding' the user data store.

--

Amazon's SES service has sufficient eenough auditing data to make the usage of a florence auditing BCC address an unnecessary risk in production environments.

- Due to the nature of medical data, it's easier NOT to distribute if avoidable.
- SES will provide information about mails sent.
- Auditing the payload can be done with permissioning around the S3 report bucket. While it should be only available to 'system processes' by default, some auditing path will need to be thought up that complies appropriately with medical guidelines for the file contents.