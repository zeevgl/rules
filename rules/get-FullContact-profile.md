---
gallery: true
categories:
- enrich profile
---
## Enrich profile with FullContact

This rule gets the user profile from FullContact using the e-mail (if available). If the information is immediately available (signaled by a `statusCode=200`), it adds a new property `fullContactInfo` to the user_metadata and returns. Any other conditions are ignored. See [FullContact docs](http://www.fullcontact.com/developer/docs/) for full details.

```
function (user, context, callback) {
  var FULLCONTACT_KEY = 'YOUR FULLCONTACT API KEY';
  var SLACK_HOOK = 'YOUR SLACK HOOK URL';

  var slack = require('slack-notify')(SLACK_HOOK);
  
  // skip if no email
  if(!user.email) return callback(null, user, context);
  // skip if fullcontact metadata is already there
  if(user.user_metadata && user.user_metadata.fullcontact) return callback(null, user, context);
  request({
    url: 'https://api.fullcontact.com/v2/person.json',
    qs: {
      email:  user.email,
      apiKey: FULLCONTACT_KEY
    }
  }, function (error, response, body) {
    if (error || (response && response.statusCode !== 200)) {
      
      slack.alert({
        channel: '#slack_channel',
        text: 'Fullcontact API Error',
        fields: {
          error: error ? error.toString() : (response ? response.statusCode + ' ' + body : '')
        }
      });

      // swallow fullcontact api errors and just continue login
      return callback(null, user, context);
    }


    // if we reach here, it means fullcontact returned info and we'll add it to the metadata
    user.user_metadata = user.user_metadata || {};
    user.user_metadata.fullcontact = JSON.parse(body);

    auth0.users.updateUserMetadata(user.user_id, user.user_metadata);
    return callback(null, user, context);
  });
}
```
