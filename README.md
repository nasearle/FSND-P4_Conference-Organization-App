## Conference Central
This is a web application created with Google ProtoRPC, Google App Engine, and Google Datastore.
Users sign in to create, customize, and organize conferences.

### Included files:
* LICENSE
* README
* static
* templates
* app.yaml -  configuration file for the App Engine app. Contains
	handler urls and python libraries
* cron.yaml - configuration file for App Engine cron jobs
* index.yaml - contains indexes to improve ndb query times
* conference.py - application server containing endpoints for
	creating, editing, and deleting conferences, sessions and profiles
* main.py - contains handlers for task queues called in conference.py
* models.py - contains the ndb and protorpc models
* settings.py - contains App Engine WEB_CLIENT_ID
* utils.py - contains getUserId function
* LICENSE

### Using the Application:
To use the application go to [delta-entity-114022.appspot.com](https://delta-entity-114022.appspot.com).
From the homepage you can log in, edit your profile, create conferences, and view and edit conferences.

#### Task 1: Design Choices Response
New Session and SessionForm classes were created in models.py. The Session class is an ndb model 
that maps its properties to corresponding properties of Session entities in Datastore. All properties
in the Session class are string data types except date and startTime. 
If necessary, data can then be converted to the correct type for operations in filters after 
retrieving it from Datastore. SessionForms is a protorpc Messages class that defines the 
response-parameters for an external call to the application. All fields of SessionForm are string 
types, which are converted to the correct data types upon the creation of a new Session entity 
(date and startTime are converted to date and time data types).

The createSession endpoint takes the websafeConferenceKey as a parameter. It passes the 
key to the createSessionObject function, which copies the data in the 
request to a dictionary object, converts the date and time fields to the correct types, and 
'puts' the data to Datastore in a new Session entity. The websafeConferenceKey is used to make the 
conference object the parent of the new session object. Finally, if the session-creator entered a 
speaker, a push task is created to check if the speaker will become the new featured speaker.

The getConferenceSessions, getConferenceSessionsByType, and getConferenceSessionsBySpeaker endpoints
each take the websafeConferenceKey as a parameter, which is used to get the conference object from
Datastore. getConferenceSessions gats all sessions with this conference as the ancestor, and apply filters
in the cases of getConferenceSessionsByType and getConferenceSessionsBySpeaker.

#### Task 2: Additional Queries
Adds two additional query types: getConferenceSessionsByDuration and getConferenceSessionsByTime.
getConferenceSessionsByDuration takes a time in minutes as input and returns
all sessions of that duration. getConferenceSessionsByTime takes a time of day as input (24-hour time
e.g. '13:00') and returns all sessions at that time.

#### Task 3: Query Problem
The not-equal (!=) filter is implemented by combining two inequality (>, <) filters joined by an OR 
operator. In Datastore, an inequality filter can be applied to at most one property per query,
so applying "Session.type != workshop" and "Session.startTime < 7pm" wouldn't work. One solution 
would be to put the results from a query using one filter into a temporary table and apply the other
filter in a query of this new table.
