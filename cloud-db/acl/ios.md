---
title: Access Control (ACL)
---

[[toc]]


## Access Type

<!--- TODO: add something here -->


## Default ACL Settings

If you do not add access control on a record, the record will inherit
the default setting for access control. The default setting is that record
is public readable.

The default setting can be changed by calling `setDefaultAccessControl` method:

```obj-c
SKYAccessControl *acl = [[SKYContainer defaultContainer] defaultAccessControl];
[acl setReadOnlyForPublic];
[acl setReadWriteAccessForRole:webmaster];
[[[SKYContainer defaultContainer] publicCloudDatabase] setDefaultAccessControl:acl];
```

```swift
let acl = SKYContainer.default().defaultAccessControl
acl?.setReadOnlyForPublic()
SKYContainer.default().publicCloudDatabase.defaultAccessControl = acl
```

In the above example, each newly created record is readable by public and it
can be modified by a `webmaster`. If you add access control to a specific
record, the above settings will be ignored for that record.


## ACL by User

Example: Share docs to colleagues

```obj-c
// supervisor is a placeholder of user id
SKYRecord *document = [SKYRecord recordWithRecordType:@"doc"];
[document setReadWriteAccessForUser:supervisor];
```

```swift
// supervisor is a placeholder of user id
let document = SKYRecord(recordType: "doc")
document.accessControl.setReadWriteAccessFor(supervisor)
```


## ACL By User Role

Role-based access control is suitable for applications that limit access to
records based on the role of the user sending the request.

If your app is configured to use role-based access control, you need to define
what roles your app is needed. Instead of granting access to users on a per-user
basis, you grant accesses to users on a per-role basis. Each user of the same
role can access records that you grant access to that role.

In this guide, we are going to walk you through making a blogging application
in which many authors can contribute articles to the same blog while
others can post comments to the articles.

### Defining Roles

During the bootstrapping phrase of your app, you should define the roles that
your app is going to need. Once the app is frozen into production mode, you
cannot grant access to roles you have not defined.

In the case of a simple blogging application, an application might need
an `webmaster` role, an `author` role and a `reader` role. Users with the `author`
role can post articles, while users with the `reader` role can post comments.

The role of an `webmaster` is to assign roles to existing users.

To tell Skygear Server you have these roles, you should define them during the
bootstrapping phrase:

```obj-c
SKYRole *webmaster = [SKYRole roleWithName:@"webmaster"];
SKYRole *author = [SKYRole roleWithName:@"author"];
SKYRole *visitor = [SKYRole roleWithName:@"visitor"];
```

```swift
let webmaster = SKYRole(name: "webmaster")
let author = SKYRole(name: "author")
let visitor = SKYRole(name: "visitor")
```

At this point, the `webmaster` role is an ordinary role without special powers
from the view of Skygear. To actually give the role the power to assign
roles to other users, you have to designate those as admin roles.

```obj-c
// webmaster from previous declaration
SKYPublicDatabase *publicDB = publicCloudDatabase;
[publicDB defineAdminRoles:@[webmaster]
                completion:^(NSError *error) {
                  if (!error) {
                    NSLog(@"Success!");
                  } else {
                    NSLog(@"Error: %@", error.localizedDescription);
                  }
                }];
```
```swift
// webmaster from previous declaration
if let webmaster = webmaster as SKYRole! {
    SKYContainer.default().publicCloudDatabase.defineAdminRoles([webmaster]) { (error) in
        if error != nil {
            print ("Success!")
            return
        }

        print ("Error: \(error?.localizedDescription)")
    }
}
```

It is possible to designate multiple roles to have admin privilege.

Finally, you have to tell Skygear Server what the default role a new user should have.
To do this, you have to set `visitor` as the default role. New user will
automatically gain the `visitor` role.

```obj-c
// visitor from previous declaration
SKYPublicDatabase *publicDB = publicCloudDatabase;
[publicDB setUserDefaultRole:@[visitor]
                  completion:^(NSError *error) {
                    if (!error) {
                      NSLog(@"Success!");
                    } else {
                      NSLog(@"Error: %@", error.localizedDescription)
                    }
                  }];
```
```swift
// visitor from previous declaration
if let visitor = visitor as SKYRole! {
    SKYContainer.default().publicCloudDatabase.defineAdminRoles([visitor]) { (error) in
        if error != nil {
            print ("Success!")
            return
        }

        print ("Error: \(error?.localizedDescription)")
    }
}
```

Remember that the above definitions are frozen at the time the application
is switched to production mode.


### Changing Roles

A user with an admin role is able to change roles of other users. In
previous section, we defined a `webmaster` will have admin privilege, and this
user can promote a `visitor` to an `author`.

This is done by setting the `roles` property of an `SKYUser` object.

```obj-c
SKYContainer *container = [SKYContainer defaultContainer];
[[container auth] queryUsersByEmails:@[@"johndoe@example.com"] completionHandler:^(NSArray<SKYRecord *> *users, NSError *error) {
	if (users.count == 1) {
	    SKYUser *john = users[0];
	    john.roles = @[author, visitor];
	    [[container auth] saveUser:john completion:nil];
	}
}];
```

### Role-based access control per record

In role-based access control, you control access to a record by specifying which
roles have access to it, and which type of access is allowed.

There are three types of access types: no access, read only and read write.
A role having read only access can fetch and query the record from the database.
A role having read write access can save and delete the record in addition
to fetch and query..

You can set access by calling these methods on a record object:

* `setNoAccessForRole:` - remove all access of a record to the specified role
* `setReadOnlyForRole:` - set read only access of a record to the specified role
* `setReadWriteAccessForRole:` - Add read write access of a record to the specified role


In our example,
when you create an article, you want `visitor` to see it, but only
only `webmaster` and `author` will be able to change it. You can set the
access by calling these methods on a record object:

```obj-c
SKYRecord *article = [SKYRecord recordWithRecordType:@"article"];
[article.accessControl setReadWriteAccessForRole:webmaster];
[article.accessControl setReadWriteAccessForRole:author];
[article.accessControl setReadOnlyForRole:visitor];
[[[SKYContainer defaultContainer] publicCloudDatabase] saveRecord:article completion:nil];
```

```swift
let article = SKYRecord(recordType: "article")
article.accessControl.setReadWriteAccessFor(webmaster)
article.accessControl.setReadWriteAccessFor(author)
article.accessControl.setReadOnlyFor(visitor)
SKYContainer.default().publicCloudDatabase.save(article, completion: nil)
```


Role-based access control are applied to each record individually. In other
words, access control applied to a record does not affect access control
applied to other records.

Note that the owner of a record always have access to a record.

Alternatively, you can modify access control at the time when `SKYContainer`
post the `SKYContainerWillSaveRecordNotification`. By doing this, you put the
access control logic in one place, and that access control settings are applied
consistently.

```obj-c
NSNotificationCenter *center = [NSNotificationCenter defaultCenter]
[center addObserverForName:SKYContainerWillSaveRecordNotification
                    object:nil
                     queue:[NSOperationQueue mainQueue]
                usingBlock:^(NSNotification *note) {
                    NSArray *records = [note.userInfo objectForKey:SKYContainerNotificationRecordsKey]
                    for (SKYRecord *r in records) {
                        if ([r.RecordType isEqualToString:@"article"]) {
                            if (r["isLocked"]) {
                                [r setReadOnlyForRole:author]
                            } else {
                                [r setReadWriteAccessForRole:author]
                            }
                        }
                    }
                }];
```


## ACL By Record Type

<!--- TODO: add something here -->

### Setting creation access

In the above examples, you add or remove access for each individual record,
but such access control does not apply when creating a new instance of
the record. At this point a `visitor` cannot modify an existing article, but
a `visitor` can create a new instance of article.

To restrict record creation to a certain role, you call `createAccess` on
a record class in the bootstrapping phrase.

```obj-c
SKYContainer *container = [SKYContainer defaultContainer];
[[container publicCloudDatabase] defineCreationAccessWithRecordType:@"article"
                                                              roles:@[webmaster, author]
                                                         completion:^(NSError *error) {
                                                           if (error) {
                                                             NSLog(error.localizedDescription);
                                                           }
                                                         }];
[[container publicCloudDatabase] defineCreationAccessWithRecordType:@"comment"
                                                              roles:@[webmaster, author, visitor]
                                                         completion:^(NSError *error) {
                                                           if (error) {
                                                             NSLog(error.localizedDescription);
                                                           }
                                                         }];
```

```swift
let container = SKYContainer.default()
container.publicCloudDatabase.defineCreationAccess(withRecordType: "article", roles: [webmaster!, author!], completion: { (error) in
    if let error = error {
        print("\(error.localizedDescription)")
    }
})
```

Note that creation access is frozen in production mode.
