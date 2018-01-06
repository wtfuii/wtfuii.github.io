---
layout: post
title:  "The art of uploading to not yet existing entities in RESTful applications"
date: 2018-01-06 12:50:02 +0100
comments: true
tags: REST 
---
Single page appliations are the preferred way to build appealing web apps these days. If those apps are not already playing with GraphQL, most of them still rely on a more or less RESTful API to interact with the backend. In most cases, it is not a issue to create a new REST entity with a single call to the backend, as a JSON can include all necessary Data. But when it comes to files which are attached to an entity, things will become a little cumbersome.

Imagine the follwing situation: You are creating a your brand new JIRA clone. It should be a SPA wich uses your favourite Backend language and framework, as well as a relational database. You are currently creating a "Create Issue" form. 

Similar to JIRA, you want to attach pictures and other files to the issue you are currently creating. As you are working with a relational database, you want to have a relation between your issues and the uploaded files to quickly query all files attached to an issue. This is where you start running into problems. You cannot upload the file instantly after the user selects it in the browser, because there is yet no "Issue" entity in the backend, to which you could attach the uploaded file. The related "Issue" entity will be created if the user hits the "Create" button. So, how to solve this problem? I tried to collect the possible solutions and ordered them from "Bad" to "Still not beautiful, but a good trade-off for me".

![Your files during upload]({{ "/images/upload_burger.gif" | absolute_url }}){: style="margin: 0 auto; display: block; max-height: 200px;"}
p=. Your files during upload

## 1. Adding base64 encoded file to JSON
 On submission of your form, you would encode your file content in base64 and attach it to a field of your submitted JSON. You'd have to provide an according deserialization method in the backend to convert the base64 string back into a file object. I wouldn't recommend this solution as base64 encoding may be errorprone due to different implementations on the frontend and backend. Additionally, base64 encoded files are approximately 37 % bigger than the original size was. That said, the JSON will be sent and the actual upload will be performed once the user hits the "Create" button. The upside of all if this is, that you can work with JSON instead of plain old FormData (yay!). In the backend you will have all necessary data in place with a single call to create the appropriate database relations.

 Your JSON would look something like this:

 ```JavaScript
{
    "summary": "This will include a file!",
    "description": "Indeed, it does!",
    "files": [
        "data:image/jpeg;base64,/9j/4AAQSkZJRgABAQAAAQABAAD//gA7Q1JFQVRPUjogZ2QtanB..."
    ]
}
 ```
 Additionally, this kind of JSON might be a little difficult to read for humans.

## 2. Upload file after successful creation of the related entitiy
This is actually the solution JIRA provides for this problem. Their REST API uses two separate calls for "POST Issue" and "POST Attachment to Issue". This approach might lead to missing attachemnts after the user submitted his data. The first call might go through the wire, which will return the ID for the newly created Issue, that will be used by the "Upload" call to identfy the upload target. For whatever reason (network instability?), this second call could fail and lead to the situation that the Issue got created successfully, but the Attachment is missing. Sure, there are ways to work around this limitation by providing appropriate errormessages, but it would be better to avoid the problem entirely.

## 3. Send plain old FormData
Okay, lets take two steps back. Why don't we just use the web the way it was originally designed to be used? Do you really need to send JSON to your HTTP endpoint? Do you really need to send JSON lists and nested objects? If the answer is "No", then you could go ahead and send plain old FormData. FormData endpoints can be described by Swagger (Open API) and other documentation languages, so this is area would still be covered. If you insist on using JSON, you could serialize it to a form field and send it as string in a form property to the server together with your attachments. This will cause some deserialization effort on the server side. The biggest downside to this would be, that API documentation will become messy, especially if the documentation is generated automatically from code.

### The downside of all previously mentioned proposals
All of the mentioned solutions share one flaw: Attachments will be uploaded once the user creates the depending Issue in the databse. If the filesize is considerable and the user is working with a poor network connection, usability might suffer due to delays after issue creation and during the upload. We shouldnt bother users with more waiting time than absolutely necessassary.

## 4. Create an entity stub on page load
This approach would be applicable, if you are creating a "Draft" feature, that creates a new issue in the database as soon as the user opens the "Create" dialog. Such a feature might come handy to retain form data if the user refreshes the page or logs in from another client. There would be an "Issue" record in the databse from the start, which would provide the foreign key for new upload records already when the creation process is not yet finished. Though this might sound like a clean and good solution, it may cause significant effort to implement this in a clean manner.

## 5. Upload files without relation to entity, add relation during creation of the entitiy
This solution requires some database cleanup from time to time, but apart from that, it needs only little effort to be implemented. This is the deal: If the user adds a file to be uploaded during the issue creation process, it will be instantly uploaded to the server, without any realtion to an issue. During this process, the user keeps on working on his new issue. After the upload finished, the ID of the uploaded file will be returned to the client and the client will include this Upload ID during submission of the issue to create the actual references. 

That solution requires a specific database design, as a database enforced 1:n relation cannot be used, because this would not allow to upload attachments without a foreign key. You either handle the relation on the application level (not database enforced) or you create a n:n relation between issues and files, though you need only a 1:n relation.

You still need to find a solution for the cases, when the user uploads files during the issue creation, but decides then not to finish the creation or reloads the page. You need to find a way to clean the orphaned database entries, which will not be used anymore by a soon-to-be-created issue. 

One "simple" way to achieve this would be to add a "dateCreated" column to your upload records and trigger a CRON job, which deletes all orphaned uploads older than x days. 

![Your CRON Job, cleaning up the database]({{ "/images/cleanup.gif" | absolute_url }}){: style="margin: 0 auto; display: block; max-height: 200px;"}
p=. Your CRON Job, cleaning up the database


Another, more sophisticated, solution would be to create a websocket connection once the user opens the "Create" dialog, which will check if the user is still creating the issue. If the connection gets closed before the user finished the creation of his issue, all previously added attachments will be removed from the database.


Feel free to leave a comment or add an approach, which I did not come up with. 