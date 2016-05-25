Login Security
==============

This module was developed to improve the security options in the login operation
of a Backdrop site. By default, Backdrop has only basic access control denying IP
access to the full content of the site.

With Login Security module, a site administrator may add any of the following
access control features to the login forms (default login form in /user and the
block login form).

These are the features included:

User Protections
----------------
When there is evidence of hard account guessing operations, or when you need to
block users because of leak password guessing attempts, Hard protections may
help defeating the system.

 - Block account: it's common to block an account after a number of failed login
   attempts. Once this count is completed, the account can be blocked and user
   and admin are advised about this.


Track time: Protection time window
----------------------------------
The time the protections operate is defined as the track time (or time window).
It's, the time that login events are being considered for the protections. For
example, we can say that an account will be blocked on the third login attempt,
but these three attempts should have happened in the protection time window.
If the time protection window is 1 hour, the three attempts should be in the
last 60 minutes. If one of this attempts was done earlier it's not considered.

The time protection window is not the time each protection is active. Blocked
accounts will remain blocked until an administrator unblocks them, and banned
hosts need also administration interaction to be unbanned.


Duration of protections
-----------------------
The duration of the enabled protections depends of its type. Soft protections
are temporary, and will expire in the time defined by the 'track time' or
protection time window.

Hard protections are permanent, and an administrator should unblock or un-ban an
account.

A blocked account could be unblocked through the administration section:
  /admin/user/user


Installation
------------
To install copy the login_security folder in your modules directory. Go to
Administer -> Site Building -> Modules and under the "Other" frame the
"Login Security" module Item will appear in the list. Enable the Checkbox
and save the configuration.


Configuration options
---------------------
You should have the 'administer site configuration' permission to configure the
Login Security module.

Go to Administer -> Site configuration -> Login Security (the url path is
admin/settings/login_security) and a form with the module options will be shown.
(Note: Any value set to 0 will disable that option.).

Basic options

 - Track time: The time window where to check for security violations. It's,
   the time in hours the login information is kept to compute the login attempts
   count. A common example could be 24 hours. After that time, the attempt is
   deleted from the list, so it will not count.

 - Maximum number of login failures before blocking a user: It's that easy,
   after this number of attempts to login as an user no matter the IP address
   attempting to, the user will be blocked. To remove the blocking of the user,
   you will have to go to: Administer -> User Management -> Users

Notifications

 The module also provides some notifications for the users to understand what is
 happening.

 - Notify the user about the number of remaining login attempts: It's also
   possible to advice the user about the attempts available prior to block the
   account.

 - Send a notification email message to a site user of your choice, each time an
   account is blocked.

 - Send a notification email message to a site user of your choice about login
   suspicious activity (a determined value threshold of invalid login attempts
   is reached).

Notifications are configurable in the Login Security settings section, where
the strings could be personalized using the following placeholders:

    %date                  :  The (formatted) date and time of the operation
    %ip                    :  The IP Address performing the operation
    %username              :  The username entered in the login form (sanitized)
    %email                 :  If the user exists, this will be it's name
    %uid                   :  ..and if exists, this will be it's uid
    %site                  :  The configured site's name
    %uri                   :  The base url of the Backdrop site
    %edit_uri              :  Direct link to the user (name entered) edit page
    %user_block_attempts   :  Configured login attempts before blocking the user
    %user_current_count    :  The total login attempts for this name
    %tracking_current_count:  Total tracked events
    %activity_threshold    :  Value of attempts to detect ongoing attack.


Cleaning the event tracking information
---------------------------------------
If for any reason, you are encountering problems with specific users (because
they are restricted), or if you change settings and want to do a module
'restart', you may clean the event track in this settings page.
At the bottom you will find a "Clear event tracking information" button.
If you click this button, all tracked events will be deleted.


Understanding protections
-------------------------
Internally, protections could consider user name, IP address or both. This is a
list of what's now implemented and how login submissions affect the protections:

 1.- On any login, the pair host<->username is saved for security, and only on a
   successful login or by track time expiration, the pair host-username is
   deleted from the security log.

 2.- For the soft blocking operation, any failed attempt from that host is being
   count, and when the number of attempts exceeds, the host is not allowed to
   submit the form.

   Note: (2nd and 3rd impose restrictions to the login form, and the time these
   restrictions are in rule is the time the information is being tracked: "Track
   Time").

 3.- For the user blocking operation, any failed attempt is count, so no matter
   what the source IP address is, when too many attempts appear the account is
   blocked. A successful login, even if the user is blocked will remove any
   tracking entry from the database.

 4.- For the host blocking operation, only the host is taken in count. When too
   many attempts appear, no matter the username being tested, the host IP
   address is banned.

   Note: (4th and 5th operations are not being canceled automatically).
   Note: The tracking entries in the database for any host <-> username pair are
        being deleted on: 'login', 'update' and 'delete' user operations.

 5.- For the ongoing attack detection, all the tracked events are taken in count.
   The system detects an ongoing attack and notices the admin about that. It will
   remain in attack mode (no more notices will be sent) until the attack is no
   longer detected. This will happen when the total number of tracked events is
   below 'maximum allowed to detect ongoing attack' / 3. Since then, once the
   threshold value is reached again, a new notification will be set in the log
   or sent by email.

   E.g Say you put 1 hour of track time and a maximum number of login failures
   to detect ongoing attack of 20. This means that if during the last hour there
   are more than 20 invalid login attempts an attack is detected. A log entry is
   created to notice the detected attack, and system switches to 'attack' status,
   where no more notices about the attack will be logged or sent. After sometime
   the attack stops. And once the number of invalid login attempts for this last
   hour is below 1/3 of this maximum: invalid attempts are below 6 (20 / 3 for
   the example) a normal status is recovered. If a new attack is detected, the
   module will alert again about it.

Most used configuration
-----------------------
The most common configuration options will look like this:

 Track time = 1 Hour
 Max number of logon failures before blocking a user  = 5
 Max number of logon failures before soft blocking a host  = 10
 Max number of logon failures before blocking a host  = 15

 - The user will be blocked after five attempts of account guessing within the
   last 60 minutes.
 - Any host trying 10 login attempts will be punished not being able to submit
   the form again within the 60 minutes track time.
 - If the number of attempts reaches 15, the host will be banned.


Other modules interaction
-------------------------
If you want your users to be informed when their accounts have been blocked,
you can use the module "Extended user status notifications":
 http://www.drupal.org/project/user_status


Important note
--------------

Currently, user UID 1 is never blocked even if the "user blocking operation" is
enabled. User UID 1 is widely exposed in too many sites (and probably the name
for that account is 'admin' in most of the cases) that deliberately was removed
from this protection because of the risk to be easily blocked. If you want to
protect your users, you may use a combination of features, and not only rely on
the user blocking operation.

More information: http://drupal.org/node/601846


Other notes
-----------
The session ID (PHP session neither Backdrop's session) is not taking in count for
the security operations, as automated bruteforce tool may request new sessions
on any attempt, ignoring the session fixation from the server.


Current Maintainer
------------------

- David Norman (https://github.com/deekayen)


Credits
-----------

- Originally written for Drupal by ilo
  (https://www.drupal.org/project/login_security)
- Ported to Backdrop by David Norman (https://github.com/deekayen)


License
-------

This project is GPL v2 software. See the LICENSE.txt file in this directory for
complete text.
