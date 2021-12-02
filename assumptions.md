The document describes the file store portion, but seems to make references to the database as a separate entity.
Based on the information provided, it seems safe to assume that:

- The database has some form of layout to facilitate at least: clinicians, trials, and patients

- The existence of authorization, though not mentioned, is implied. 
The means of auth need not be discussed, but whatever mechanism the user is using for authentication (in this case in the API layer) will handle permissions, and limiting of content to that which they 'own'.
The term "user" will be used interchangibly with clinician, "system user" (someone that may be logging in to a GUI application), and API user. Those things may be quite different in reality, but the context should hold up well enough. For instance, in API documentation, "user" implying "api user" doesn't seem as though it would be ambiguous with the given information.


Furthermore, as the described metadata does not include things like a trial to which it might belong, so it is assumed that kind of association exists in the database. 

---

As for the requirements, the following assumptions were made:

- The subject of the report is left vague, but based on the phrase 'subscribed users', the data won't be from a single user, but something that multiple users would be collaborating on -- most likely a trial.
Therefore, the asusmption will be made that user subscriptions will be to a report to be compiled based on a given trial, and that particular trial id will be used as the aggregating factor for report subjects / subscriptions.

- The technical specifics, along with a discussion with the product owner, support the idea that realtime or near-realtime reports wouldn't be out of the question, and so designing the system between this and scalability is a sound approach.

- The requirements mention subscribing users, rather than e-mails. This could be clarified with product manager / user discussion, but for the purposes of the exercise, let's assume that the intent is to subscribe a user. This means that the system would need to have some association between users and e-mails, which seems safe to assume already exists (as user accounts in any system without associated e-mail addresses would be EXTREMELY uncommon). Therefore, the subscription will be made between the user and the trial they wish to subscribe to, with the e-mailer fetching the address at delivery time.


---

Assumptions made while designing:

- Users would want to subscribe themselves, one-by-one, and not an entire grouping at a time. If this assumption proves false, a 'mass subscribe' endpoint could be added later. (single subscribe would work better for testing the new functionality on a small group of initial users anyway)

- Assuming there's neither a system for reports nor for subscriptions, it's tough to know whether outright the new API is for "reports", "subscriptions", or "report subscriptions". In reality, it would be best to decide how this will live in regards to those concepts. But for a first draft, simply saying "subscriptions" should keep us all on the same page until that can be finalized.