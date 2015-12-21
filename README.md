App Engine Conference App for Udacity Nanodegree: Full Stack Developer 

## Products
- [App Engine][1]

## Language
- [Python][2]

## APIs
- [Google Cloud Endpoints][3]
- [API Explorer][7]

## Setup Instructions
1. Update the value of `application` in `app.yaml` to the app ID you
   have registered in the App Engine admin console and would like to use to host
   your instance of this sample.
1. Update the values at the top of `settings.py` to
   reflect the respective client IDs you have registered in the
   [Developer Console][4].
1. Update the value of CLIENT_ID in `static/js/app.js` to the Web client ID
1. (Optional) Mark the configuration files as unchanged as follows:
   `$ git update-index --assume-unchanged app.yaml settings.py static/js/app.js`
1. Run the app with the devserver using `dev_appserver.py DIR`, and ensure it's running by visiting your local server's address (by default [localhost:8080][5].)
1. (Optional) Generate your client library(ies) with [the endpoints tool][6].
1. Deploy your application.

## Tasks

## Task 1: Add Sessions to a Conference

### Define the following Endpoints methods

- `getConferenceSessions(websafeConferenceKey)`
   Given a conference, return all sessions

- `getConferenceSessionsByType(websafeConferenceKey, typeOfSession)`
   Given a conference, return all sessions of a specified type (eg lecture, keynote, workshop)

- `getSessionsBySpeaker(speaker)`
   Given a speaker, return all sessions given by this particular speaker, across all conferences

- `createSession(SessionForm, websafeConferenceKey)`
   Open only to the organizer of the conference

### Define Session class and SessionForm

In the SessionForm pass in:
- name
- highlights
- speaker
- duration
- typeOfSession
- date
- start time (in 24 hour notation so it can be ordered)

### Explain your design choices

For the session implementation I modeled the conference class, but removed things like display name since I didn't need it for sessions.

I added speaker name as a string to get featured speaker announcements.
I added websafeConferenceKey to sessions that pass the key from the Conference class to show the parent/child relationship and to be able to show all sessions within a conference.

**name = ndb.StringProperty(required=True)**
This stores the session name and is required and is stored as a string property.


**highlights      = ndb.StringProperty()**
Highlist various key sessions 

**organizerUserId = ndb.StringProperty()**
stores the user id although this can also be gotten from the parent userid.

**speaker         = ndb.StringProperty()**
Stores speaker information and makes it easy to get featured speaker.

**duration        = ndb.IntegerProperty()**
How long a session runs and is stored as an Integer.

**typeOfSession   = ndb.StringProperty(repeated=True)**
Shows the types of sessions and can be repeated since there can be more than one type. 

**startDate       = ndb.DateProperty()**
Shows when the session begins and is held in a DateProperty.

**startTime       = ndb.TimeProperty()**
Shows what time a session starts. Held in TimeProperty type.
	
**websafeConferenceKey = ndb.StringProperty()**
Stores the conference key for the session in a websafe URL and works for the parent/child connection between conference and session.
	


## Task 2: Add Sessions to User Wishlist
For sessions to be added to wishlists I added websafeSessionKey to show that connection between the two.


### Define the following Endpoints methods

- `addSessionToWishlist(SessionKey)`
   adds the session to the user's wishlist of sessions to attend

- `getSessionsInWishlist()`
   query for all the sessions that the user is interested in

- `deleteSessionFromWishlist()`
	deletes the session from the user's wishlist of sessions to attend.  

## Task 3: Work on indexes and queries

### Part 1: Come up with 2 additional queries

Think about other types of queries that would be useful for this application.
Describe the purpose of 2 new queries and write the code that would perform them.

Note: The following two endpoints can be assessed from the [API Explorer][7]

#### query 1

Get mini sessions that are 30 minutes for people short on time.

```py
	@endpoints.method(message_types.VoidMessage, SessionForms,
		path='minisession',
		http_method='GET', name='miniSession')
	def miniSession(self, request):
		"""Find mini sessions at 30 minutes."""
		sessions = Session.query(Session.duration == 30)	
		return SessionForms(
			items=[self._copySessionToForm(sess) for sess in sessions]
			)	
```

#### query 2

Query 2 Find all Breakfast sessions for hungry people.

```py
	@endpoints.method(message_types.VoidMessage, SessionForms,
			path='breakfastSession',
			http_method='GET', name='breakfastSession')
	def breakfastSession(self, request):
		q = Session.query()
		# simple filter usage:
		q = q.filter(Session.typeofSession == "Breakfast")
		return SessionForms(
			items=[self._copySessionToForm(sess) for sess in q]
		)
```

### Part 2: Solve the following query related problem


What if User wanted to avoid workshop (type) sessions as well as sessions after 7pm? What are the issues here and how could this be implemented?

"A single query may not use inequality comparisons (<, <=, >, >=, !=) on more than one property across all of its filters."

But "Note that a query can combine equality (=) filters for different properties, along with one or more inequality filters on a single property."
https://cloud.google.com/appengine/docs/python/datastore/queries#Python_Property_filters

You could do a query where startDate < 7 AND sessionType = 'breakfast'
										 AND sessionType = 'opening'
										 etc
keep adding all sessionTypes and what they equal other than workshop. It isn't very elegant, but it would do the trick according to the notes on Google site.


## Task 4: Add a Task

### Define the following Endpoints method

- `getFeaturedSpeaker()`
   When a new session is added to a conference if there is more than one session
   by this speaker return featured speaker / sessions from memcache.

####Notes I Used While Working Through It:
When we use message_types.VoidMessage it's because  we there is no argument in the return body or no GET so there's nothing to pass.

But if we have a request that has a path or a string like /conference or /websafekey, etc then we use a resourcecontainer

this explains exactly what's going on with the resource container and the endpoints.method
https://cloud.google.com/appengine/docs/python/endpoints/create_api#using_resourcecontainer_for_path_or_querystring_arguments


[1]: https://developers.google.com/appengine
[2]: http://python.org
[3]: https://developers.google.com/appengine/docs/python/endpoints/
[4]: https://console.developers.google.com/
[5]: https://localhost:8080/
[6]: https://developers.google.com/appengine/docs/python/endpoints/endpoints_tool
[7]: https://apis-explorer.appspot.com/apis-explorer/?base=https://udacity--conference-central.appspot.com/_ah/api#p/

[8]: https://cloud.google.com/appengine/docs/python/endpoints/getstarted/backend/write_api

[9]: https://cloud.google.com/appengine/docs/python/
[10]: https://cloud.google.com/appengine/docs/python/modules/routing
[11]: https://cloud.google.com/appengine/docs/python/memcache/usingmemcache
[12]: https://cloud.google.com/python/
[13]: https://discussions.udacity.com/t/featured-speaker-not-showing-up-in-memcache/39444/4
